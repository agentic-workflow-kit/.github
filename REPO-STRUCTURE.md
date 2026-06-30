# Repository structure standard

This is the default layout for repositories in the `agentic-workflow-kit` organization. It exists so
every repo scans the same way — a contributor or agent landing in any product knows where the _what &
why_, the _how_, and the _source_ live without exploring.

A repository may override this when it has a documented reason, but the burden is on the override to
justify itself. The reference implementation is [`jig`](https://github.com/agentic-workflow-kit/jig).

## The spine

Every repo, regardless of archetype, follows the same top-level spine:

```text
<repo>/
  README.md          # entry point: what it is, status, how to start
  LICENSE            # MIT unless the repo states otherwise
  docs/
    README.md        # altitude index: product above design
    product/         # what & why — audience-facing
    design/          # how — mechanics, decisions, contracts
  <source>           # the thing the repo produces (see archetypes)
```

- **`README.md`** is the front door: one-paragraph description, status, where the product sits in the
  suite spine (`define / PRD → technical-design → jig → learning loop`), and how to start. Keep links
  to `docs/product/` and `docs/design/` near the top.
- **`docs/product/`** owns the _what and why_: concepts, guarantees, use cases, fit ("when to use /
  when not to"). Audience-facing; readable without the code.
- **`docs/design/`** owns the _how_: architecture, decisions, internal contracts, formats, and any
  lessons ledger. Mechanics, not marketing.
- **`docs/README.md`** is an altitude index: a short table placing `product/` (intent) above
  `design/` (mechanics). Both `docs/product/` and `docs/design/` also carry a `README.md` that
  indexes the folder.

Do not leave loose `*.md` design or product docs at the repo root — `README.md` is the only Markdown
file that belongs at the top level. Everything else lives under `docs/`.

## Source tier by archetype

"Source" means the artifact the repo produces. It differs by archetype, but always sits beside `docs/`,
never inside it.

| Archetype            | Example            | Source tier                                                                   |
| -------------------- | ------------------ | ----------------------------------------------------------------------------- |
| **App / engine**     | `jig`              | `packages/`, `src/`, plus build/test config and `scripts/`                    |
| **Skills pack**      | `technical-design` | `skills/` + `methodologies/` (or equivalent profiles) + `evals/` + `scripts/` |
| **Community health** | `.github`          | profile and org default files at root; no `docs/` split required              |

The split is the same; only the source tier changes. A skills pack's "source" is its skills and the
fixtures that prove them, exactly as an engine's source is its packages and tests.

## Package and check standard

Active repositories carry a root `package.json` even when they do not publish a runtime package yet.
The package is the local tooling contract:

- `private: true` until the repository intentionally publishes an artifact.
- `packageManager: "pnpm@11.5.1"` and `engines.node: ">=22"`.
- A root `.nvmrc` pins the **local** dev version (currently `26`, the current release line).
  `engines.node` is the **compatibility floor**; CI runs that floor (Node 22, see below) so the
  supported range is actually verified, while contributors get the current line locally.
- `pnpm check` is the single required local and CI gate.
- Docs-only repositories may make `pnpm check` a lightweight formatting/lint validation.
- Skills packs and engines wrap their repo-specific validation under `pnpm check` instead of exposing
  one-off commands as the primary contributor path.

The default CI workflow is `.github/workflows/check.yml` with a job named exactly `check`, running on
pull requests and pushes to `main`. It uses `actions/checkout@v7`, `pnpm/action-setup@v6`,
`actions/setup-node@v6` pinned to the `engines.node` floor (`node-version: 22`) so the `>=22`
support claim is verified — pin it explicitly, since a range like `>=22` resolves to the latest
satisfying version, not the floor — `actions/cache@v6`, pnpm 11.5.1, a repo-local `.pnpm-store`,
and `pnpm --config.store-dir="$PNPM_STORE_DIR" check`.

### Engine-archetype tooling

The check gate varies by archetype, the same way the source tier does. Docs-only and skills-pack
repos keep `pnpm check` as a prettier formatting gate over Markdown/YAML/JSON. An **engine** repo,
the moment it grows real TypeScript source, adopts the engine tooling standard — this is
**archetype-scoped, not optional**: every engine conforms, and docs/skills repos must not adopt it.

- **Lint/format stance: biome for code, prettier for docs.** biome formats, lints, and sorts
  imports for TS/JS/JSON; prettier keeps Markdown/YAML. Each repo runs one formatter per file type,
  so there is no in-repo conflict.
- **Type and test gate:** `tsc -b` (strict, `NodeNext`, project-referenced) plus vitest with a 90%
  coverage floor (aim 95%) — consistent with the test-driven rule once code exists.
- **Composed gate:** `check` runs `lint` (`biome check .`) + `format:check` (prettier on docs) +
  `typecheck` + `test`.

The canonical, validated configs live in
[`repo-template/archetypes/engine/`](https://github.com/agentic-workflow-kit/repo-template/tree/main/archetypes/engine)
(`biome.json`, `tsconfig.base.json`, `tsconfig.json`, `vitest.config.ts` + an adoption guide),
validated end-to-end on Node 26 with biome 2.5, TypeScript 6, and vitest 4. turbo,
dependency-cruiser, and monorepo project references are deliberately deferred until a repo's weight
earns them.

## Developer setup and worktrees

Setup and worktree management are two layers: a setup step prepares one checkout to install, build,
and test; a worktree step creates an _additional_ checkout and then runs that same setup inside it.
Keep both as named scripts so every repo drives the same way. The reasoning behind this section —
the OSS/pnpm survey, the tool comparison, and the nested-worktree failure analysis — is archived in
[`references/worktree-devx-report-2026-06-30.md`](references/worktree-devx-report-2026-06-30.md).

### `pnpm dev:setup`

The canonical command to prepare a checkout for local development. A repo exposes it once setup is
more than `pnpm install`; the baseline does:

- validate the Node version against `engines.node` and enable Corepack;
- install with the pinned pnpm and a frozen lockfile;
- preserve ignored local secrets — never overwrite an existing env file;
- print the effective package manager, the pnpm store path, and the next command (`pnpm check`);
- stay idempotent and non-destructive.

Repos layer their own steps on top (env/seed copies, cache links, browser installs) but keep the
contract: one entrypoint, pinned toolchain, minimal bootstrapping, no destructive cleanup. A
docs-only repo where `pnpm install` is the whole story does not need `dev:setup` — reach for it the
moment setup grows a second step. The reference implementation ships in
[`repo-template`](https://github.com/agentic-workflow-kit/repo-template).

### `pnpm worktree:new <branch>`

The canonical command to create a worktree and run `pnpm dev:setup` inside it. The durable
invariant it enforces:

**No active worktree lives inside another checkout — worktrees are external siblings, never nested
under the repo root.** A nested `.worktrees/` is a footgun: broad globs (`prettier "**/*..."`,
`biome .`), file watchers, affected-package calculators, and agent context loaders all walk into it
and double-process or cross-contaminate sibling work, and duplicate nested `AGENTS.md` files
mislead agents. Git can ignore the directory; the tools and agents that do not honor that ignore
are the problem. So even a light, prettier-only repo benefits from the invariant.

The recommended local layout is a **bare hub plus sibling worktrees**: a bare clone holds the shared
Git object database, and each checkout (`main/`, `<branch>/`, `pr-<n>/`) is a sibling, so no
checkout is a descendant of another. `worktree:new` resolves the sibling root from
`$CODE_WORKTREE_ROOT` (or the bare hub's parent) and never commits a machine-specific path; see the
archived report for the `git clone --bare` walkthrough.

### pnpm store policy

Our repos are single-package with light dependencies, so pnpm's fast-worktree store optimizations
are not needed yet. Use the default global store; it is shared across a developer's worktrees within
one trust boundary. Two rules carry forward: a shared writable store assumes mutual trust — do not
share one store across untrusted agents or users; and revisit `enableGlobalVirtualStore` only if a
repo grows heavy dependencies and runs many parallel worktrees. Until then, a frozen `pnpm install`
per worktree (driven by `dev:setup`) is enough.

## Contributor contract (AGENTS.md)

Every active repo carries its own `AGENTS.md` at the root: the lean, always-loaded contract for
humans and agents working in that repo. It is **self-contained** — an agent with only this one
repo checked out (including Claude or Codex cloud runs) must be able to act on it without reading
any other repo or assuming the local multi-repo layout. Keep it to what matters every session:
where ground truth lives in this repo's `docs/`, the few real invariants, the `pnpm check` gate,
and the conventions that prevent drift; let `docs/` hold the detail. Restate the small shared
baseline inline rather than pointing at this `.github` repo for rules a contributor needs. The
org-level / container `AGENTS.md` is **local orientation only** (the multi-repo picture), never a
runtime dependency of a repo.

`AGENTS.md` is the single source of truth and Codex reads it natively. Claude Code auto-loads
`CLAUDE.md`, not `AGENTS.md`, so every repo also carries a one-line `CLAUDE.md` that imports it:

```text
@AGENTS.md
```

No content is duplicated — the `CLAUDE.md` is a loader shim so Claude (local and cloud) reads the
same contract. Add Claude-specific notes below the import only if a repo genuinely needs them.

## Repository settings standard

Default repo policy is `main`-based:

- `main` is the default protected branch.
- Pull requests are required before merge.
- The required status check is `check`.
- Review conversations must be resolved before merge.
- Force pushes and branch deletion are disabled for `main`.
- Squash merge is the only enabled merge method; merged branches are deleted automatically.
- Actions run with read-only default workflow permissions.

Branch rules are enforced by a repository ruleset on `main`; merge methods and workflow permissions
are repo settings. Because a GitHub template repository copies files but **not** settings, all of
this is applied reproducibly by `scripts/apply-repo-standard.sh <owner>/<repo>` in the
[`repo-template`](https://github.com/agentic-workflow-kit/repo-template) repo (idempotent — updates
the existing `main` ruleset if present, else creates it).

This is applied **per repo by design.** A single org-level ruleset covering every repo at once
would automate branch protection for new repos, but org-level rulesets are a GitHub **Team**
feature and this org runs on the **Free** plan, so the per-repo script is the mechanism. The one
exception is the default workflow permission: it is already set org-wide to read-only, so new repos
inherit that without the script.

Stricter supply-chain policy, such as SHA-pinned Actions or selected-actions allowlists, is a future
hardening step and is not part of the default moderate baseline.

## Shared conventions

- Community health files (`CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, `SECURITY.md`, PR/issue templates)
  default to the org-level [`.github`](https://github.com/agentic-workflow-kit/.github) repo. A repo
  only carries its own copy when it needs a policy that differs from the default.
- Keep product boundaries explicit: design artifacts do not execute delivery; execution artifacts do
  not invent product or design scope.
- Reference material copied from another repo is read-only unless the repo explicitly owns it. Mark
  such snapshots as references, not runtime dependencies.

## Starting a new repo

Use the [`repo-template`](https://github.com/agentic-workflow-kit/repo-template) repository (the
"Use this template" button on GitHub). It carries the standard spine — `README`, `docs/` altitude
index, self-contained `AGENTS.md` + `CLAUDE.md`, `package.json`, `.nvmrc`, `check.yml`, `LICENSE`,
and ignore files. After creating the repo:

1. Replace the `<repo>` placeholders and delete the "using this template" block in `README.md`.
2. Add your archetype's source tier beside `docs/` (engine: `packages/` or `src/` plus `tests/`;
   skills pack: `skills/`, `methodologies/`, `evals/`; docs-only: nothing extra).
3. Apply the settings standard (which the template does not copy):
   `bash scripts/apply-repo-standard.sh <owner>/<repo>`.
4. Run `pnpm check` to confirm the gate is green.

## Adopting the standard in an existing repo

1. Create `docs/product/` and `docs/design/`; add a `README.md` index to each and a `docs/README.md`
   altitude index.
2. Move loose root-level design/product docs into the right folder; leave only `README.md` at root.
3. Update internal cross-references and any tooling (CI, validators) that hardcodes a moved path.
4. Re-run the repo's checks to confirm nothing broke, then link the two doc trees from `README.md`.
