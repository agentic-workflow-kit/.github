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
- `packageManager: "pnpm@11.5.1"` and `engines.node: ">=24"`.
- `pnpm check` is the single required local and CI gate.
- Docs-only repositories may make `pnpm check` a lightweight formatting/lint validation.
- Skills packs and engines wrap their repo-specific validation under `pnpm check` instead of exposing
  one-off commands as the primary contributor path.

The default CI workflow is `.github/workflows/check.yml` with a job named exactly `check`, running on
pull requests and pushes to `main`. It uses `actions/checkout@v7`, `pnpm/action-setup@v6`,
`actions/setup-node@v6`, `actions/cache@v6`, Node 24, pnpm 11.5.1, a repo-local `.pnpm-store`, and
`pnpm --config.store-dir="$PNPM_STORE_DIR" check`.

## Repository settings standard

Default repo policy is `main`-based:

- `main` is the default protected branch.
- Pull requests are required before merge.
- The required status check is `check`.
- Review conversations must be resolved before merge.
- Force pushes and branch deletion are disabled for `main`.
- Squash merge is the only enabled merge method; merged branches are deleted automatically.
- Actions run with read-only default workflow permissions.

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

## Adopting the standard in an existing repo

1. Create `docs/product/` and `docs/design/`; add a `README.md` index to each and a `docs/README.md`
   altitude index.
2. Move loose root-level design/product docs into the right folder; leave only `README.md` at root.
3. Update internal cross-references and any tooling (CI, validators) that hardcodes a moved path.
4. Re-run the repo's checks to confirm nothing broke, then link the two doc trees from `README.md`.
