# Worktree And Dev Environment DX Report

Date: 2026-06-30

> **Status: archived reference (decision record).** This investigation studied four **external**
> repositories (`on-class-web`, `pathway`, the retiring `workflow-kit` prototype, and `ghx`) — none
> of which are part of the `agentic-workflow-kit` org — to derive a developer-setup and worktree
> standard. The **decisions adopted by this org** live in
> [`REPO-STRUCTURE.md` → Developer setup and worktrees](../REPO-STRUCTURE.md#developer-setup-and-worktrees):
> `pnpm dev:setup`, `pnpm worktree:new`, the no-nested-worktree invariant, the bare-hub/sibling
> topology, and the store-policy caveat. This file is kept for the **reasoning** behind those
> decisions (the OSS/pnpm survey, the `git-wt` vs `git-worktree-runner` comparison, and the
> nested-worktree failure analysis). The external repos are referenced as evidence only; this org
> does not own or migrate them.
>
> **One recommendation in this report was explicitly _not_ adopted:** the body proposes carrying
> worktree/base/verify policy in a `.workflow/config.yaml` ("Standard policy sources", "Portable
> workflow config shape", "Final Recommendation" item 5). That is **legacy-kit config and is out of
> scope here.** The org's worktree policy is the invariant plus the `dev:setup` / `worktree:new`
> commands in `REPO-STRUCTURE.md`, which resolve topology from `CODE_WORKTREE_ROOT` (or the checkout
> layout) — there is no `.workflow/config.yaml` dependency. Read those sections as historical
> reasoning, not as org policy.

## Task

Review whether `coderabbitai/git-worktree-runner` can improve our development experience, with specific attention to how worktree setup currently works across:

- `/Users/aryekogan/repos/on-class-web`
- `/Users/aryekogan/repos/pathway`
- `/Users/aryekogan/repos/workflow-kit`
- `/Users/aryekogan/repos/ghx`

Then broaden the question: research current OSS and pnpm best practice for developer setup, environment bootstrap, and worktree workflows. Decide whether the right standard is to adopt a worktree runner, create a local standard setup script, or combine both.

Follow-up reviewed on the same date: `zkochan/git-wt`, which is referenced by pnpm's current worktree guidance and appears to be the extracted/simplified helper for the pnpm-style bare-repo worktree workflow.

## Background

The starting hypothesis was that worktree setup may be too inconsistent across repos, and `git-worktree-runner` might offer a ready-made standard for creating, preparing, navigating, launching agents/editors in, and cleaning worktrees.

During the review, it became clear that "worktree management" and "developer environment setup" are separate layers:

- Developer setup prepares a checkout so it can build, run, test, and use local secrets/caches/tooling safely.
- Worktree setup creates an additional checkout and then must delegate to the same developer setup logic.
- Agent launch/editor launch is a convenience layer above both.

This distinction matters because our repos already have project-specific setup needs: ignored env files, Turbo caches, Vercel/Supabase overlays, Playwright installs, pnpm store policy, and branch/base rules.

## What I Did

### Local repo inspection

I inspected live local state for the four target repos:

- `git status --short --branch`
- `git worktree list --porcelain`
- closest repo instructions such as `AGENTS.md`, `CONTRIBUTING.md`, `README.md`
- setup scripts such as `scripts/setup-worktree.sh` and `scripts/setup-dev-env.sh`
- workflow config such as `.workflow/config.yaml`
- Codex overlays such as `.codex/config.toml`

I did not modify those repos.

### Tool review

I cloned and reviewed `coderabbitai/git-worktree-runner` at:

- Repository: https://github.com/coderabbitai/git-worktree-runner
- Commit reviewed locally: `ad7a3c534fc36e6adfee44c480b03c2f7f959502`

I inspected:

- `README.md`
- `docs/configuration.md`
- `lib/commands/create.sh`
- `lib/commands/clean.sh`
- `lib/commands/run.sh`
- `lib/hooks.sh`
- `lib/adapters.sh`
- AI adapters for Claude/Cursor and generic Codex support

I also cloned and reviewed `zkochan/git-wt` at:

- Repository: https://github.com/zkochan/git-wt
- Commit reviewed locally: `c949edb2c593420e5a2335fdf00b085ee6ceafa6`
- npm package: `@zkochan/git-wt`
- npm version reviewed: `0.0.4`
- npm created: `2026-04-24`
- npm latest modified: `2026-05-17`

I inspected:

- `README.md`
- `package.json`
- `src/index.ts`
- `src/create.ts`
- `src/cleanup.ts`
- `src/init.ts`

I also checked pnpm's current repository and docs around this tool:

- pnpm worktree docs: https://pnpm.io/git-worktrees
- pnpm repo: https://github.com/pnpm/pnpm
- pnpm checked-in `.git-wt/pr-hook`: https://github.com/pnpm/pnpm/blob/main/.git-wt/pr-hook
- pnpm workspace config with `enableGlobalVirtualStore: true`: https://github.com/pnpm/pnpm/blob/main/pnpm-workspace.yaml

### OSS and pnpm research

I sampled well-known pnpm and large OSS repos for their actual setup patterns:

- Next.js: https://github.com/vercel/next.js/blob/canary/contributing/core/developing.md
- Vite: https://github.com/vitejs/vite/blob/main/CONTRIBUTING.md
- Astro: https://github.com/withastro/astro/blob/main/CONTRIBUTING.md
- Supabase: https://github.com/supabase/supabase/blob/master/DEVELOPERS.md
- Biome: https://github.com/biomejs/biome/blob/main/CONTRIBUTING.md
- pnpm itself: https://github.com/pnpm/pnpm

I also checked pnpm's official documentation for:

- `pnpm setup`: https://pnpm.io/cli/setup
- package scripts: https://pnpm.io/scripts
- Git worktrees and multi-agent development: https://pnpm.io/git-worktrees

### Independent review

After adding the `git-wt` and bare/sibling topology recommendation, I spawned an independent read-only reviewer against this report.

The reviewer agreed that the direction is technically sound, but noted that this report should not pretend to be an implementation plan without a separate migration design.

The reviewer specifically called out two boundaries:

- A future implementation should define a portable external-worktree policy shape instead of replacing `.worktrees/` with a user-specific absolute path.
- Existing local nested worktrees should be treated as reference evidence and current-state risk, not as something this report attempts to migrate.

The reviewer also recommended:

- adding a per-repo pnpm store/global virtual store decision table
- softening overbroad tooling-risk language where repo ignore files already mitigate some commands
- adding the pnpm shared-store trust boundary caveat
- adding or softening the pnpm nested-worktree issue citation

Those points are incorporated below as findings and recommendation criteria, not as a migration runbook.

## Current Repo Findings

### OnClass

Evidence:

- `AGENTS.md` says non-trivial work happens in a git worktree branched from `main`.
- `.workflow/config.yaml` sets:
  - `git.strategy: worktree`
  - `git.baseBranch: main`
  - `git.worktreeDir: .worktrees`
  - `verify.full: pnpm check`
- `scripts/setup-worktree.sh`:
  - requires Node 24+
  - enables Corepack
  - finds the primary `main` checkout
  - copies missing ignored env files from the primary checkout
  - copies `scripts/.env.seed` when present
  - links `.turbo`
  - links `.vercel`
  - runs `pnpm install --frozen-lockfile --prefer-offline`
  - installs Playwright Chromium

Assessment:

OnClass already has the right shape: repo-specific setup script plus workflow config. The setup behavior is more specific than `git-worktree-runner`'s generic copy hooks.

### Pathway

Evidence:

- `AGENTS.md` says fresh worktrees should run `scripts/setup-worktree.sh`.
- `.workflow/config.yaml` sets:
  - `git.strategy: worktree`
  - `git.baseBranch: main`
  - `git.worktreeDir: .worktrees`
  - `verify.full: pnpm run check`
- `.codex/config.toml` enables repo-specific Vercel and Supabase app/plugin overlays.
- `scripts/setup-worktree.sh`:
  - requires Node 24+
  - enables Corepack
  - finds the primary `main` checkout
  - copies ignored env files
  - links `.turbo`
  - links `.vercel`
  - installs dependencies
  - installs Playwright Chromium

Assessment:

Pathway is similar to OnClass, but with stronger production/Supabase constraints. Any standard must preserve repo-specific Codex capability overlays and production-secret boundaries.

### workflow-kit

Evidence:

- `AGENTS.md` says:
  - `v-next` is the integration base
  - `main` is frozen legacy
  - non-trivial work belongs under `.worktrees/<name>` cut from `v-next`
  - PRs target `v-next`
  - `pnpm check` is the local gate
- `scripts/setup-worktree.sh`:
  - requires Node 24+
  - enables Corepack
  - finds the primary `v-next` checkout
  - copies `.turbo` from primary checkout rather than linking it
  - installs dependencies using the primary checkout's `.pnpm-store`
  - supports `--warn-as-error`

Assessment:

workflow-kit intentionally differs from OnClass/Pathway. Its setup is designed around `v-next`, isolated Turbo logs/cache behavior, and explicit pnpm store reuse. A generic worktree runner cannot replace this without duplicating important policy.

### ghx

Evidence:

- No repo-level `AGENTS.md` found.
- No `scripts/setup-worktree.sh` found.
- `CONTRIBUTING.md` says:
  - Node 22+
  - run `./scripts/setup-dev-env.sh`
  - run `pnpm install`
  - run `pnpm run build`
  - create branches from `main`
  - ensure `pnpm run ci` passes
- `scripts/setup-dev-env.sh` only enables Corepack and activates `pnpm@10.29.3`.

Assessment:

ghx is the repo most in need of a small standardized setup contract. It does not need `git-worktree-runner` first; it needs repo instructions, a worktree-aware setup script, and workflow config parity.

## git-worktree-runner Findings

`git-worktree-runner` is useful as a convenience CLI. It provides:

- `git gtr new <branch>`
- `git gtr run <branch> <command>`
- `git gtr go <branch>`
- `git gtr editor <branch>`
- `git gtr ai <branch>`
- `git gtr list`
- `git gtr rm <branch>`
- `git gtr clean --merged --closed`
- `.gtrconfig` team defaults
- copy include/exclude patterns
- post-create and remove hooks
- trust-gating for executable commands in `.gtrconfig`

Useful traits:

- Makes manual worktree creation/navigation easier.
- Supports repo-relative `gtr.worktrees.dir = .worktrees`.
- Can run a repo bootstrap through `postCreate`.
- Has a trust model for executable `.gtrconfig` hooks.
- Skips dirty worktrees during cleanup unless forced.
- Can launch AI tools, including generic Codex CLI by PATH.

Limitations for our use case:

- It does not read our `.workflow/config.yaml`.
- It does not understand `AGENTS.md` as policy.
- It does not know workflow-kit's `v-next` vs `main` distinction unless duplicated into `.gtrconfig`.
- It does not run `ghx` review-thread or merge-readiness checks.
- It does not enforce our "verify live state first" habit.
- It would add another configuration surface, `.gtrconfig`, that can drift from `AGENTS.md`, `.workflow/config.yaml`, and setup scripts.
- Generic copy/include hooks are a weaker fit than our repo-specific setup scripts.

Conclusion:

`git-worktree-runner` is a useful optional ergonomic layer, not the standard itself.

## zkochan/git-wt Findings

`zkochan/git-wt` is much closer to the pnpm worktree pattern than `git-worktree-runner`.

It provides:

- `git-wt <branch-name>`
- `git-wt <pr-number>`
- native `git wt <branch-or-pr>` usage through Git's `git-<cmd>` convention
- shell `wt` helper for bash/zsh/fish that creates the worktree and `cd`s into it
- GitHub PR checkout through `gh pr view`
- automatic fork remote creation for PR worktrees
- local branch naming of `pr-<number>` for PR checkouts
- upstream config so pushes target the source fork/branch
- optional PR hook at `<worktree>/.git-wt/pr-hook`
- personal fallback PR hook at `~/.config/git-wt/pr-hook`
- cleanup for worktrees whose branches map to merged PRs
- symlinking of shared `.claude` and `.local-settings` from the Git common directory

Important implementation details:

- New branch worktrees are created as siblings of the current worktree.
- Branch slashes are converted to dashes for directory names.
- New branches are based on the repository default branch detected from `origin/HEAD`, with fallback to `main`.
- Dependency install only runs when `package.json` has `devEngines.packageManager.name`.
- The install command is plain `<packageManager> install`.
- Cleanup protects `main`, `master`, and `v<NN>` branches.
- Cleanup removes worktrees with `git worktree remove --force` and deletes branches with `git branch -D`.

Why it matters:

`git-wt` validates the direction of a small `pnpm worktree:new <branch|pr>`-style command better than `git-worktree-runner` does. It is intentionally narrow and matches the pnpm bare-repo/sibling-worktree pattern.

It is also current pnpm-maintainer practice, not merely a generic third-party tool. pnpm's public docs describe the same pattern: bare repository hub, sibling worktrees, `enableGlobalVirtualStore: true`, fast installs, and helper commands. The pnpm repo currently contains a checked-in `.git-wt/pr-hook` that references `@zkochan/git-wt`.

Limitations for our use case:

- It does not read `.workflow/config.yaml`.
- It does not understand `AGENTS.md` as policy.
- It does not know workflow-kit's `v-next` integration base.
- It assumes sibling worktrees relative to the current checkout, not our existing nested `.worktrees/` layout.
- It would not run setup automatically in our repos unless we add `devEngines.packageManager.name` or wrap it, because our repos primarily use `packageManager`.
- Plain package-manager `install` is not enough for OnClass, Pathway, or workflow-kit.
- It does not copy/link env files, `.turbo`, `.vercel`, Playwright browsers, or workflow-kit's primary `.pnpm-store`.
- Its cleanup is less conservative than our desired agent closeout workflow because it force-removes worktrees.
- It is not `ghx`-aware and does not inspect unresolved review threads or repo-specific merge readiness.
- The package is new and small: useful, but not mature enough to make mandatory across our repos without a wrapper.

Conclusion:

`git-wt` is the better reference model for our desired UX. It should influence the standard, especially the `pnpm worktree:new` shape and optional `wt` shell helper. It should not replace our repo-owned policy and setup scripts.

## OSS Pattern Findings

Across sampled OSS repos, I did not find a universal `pnpm setup` convention for project setup.

The dominant pattern is:

1. Pin the package manager in `packageManager`.
2. Document Node version and Corepack.
3. Run `pnpm install`.
4. Run repo-specific scripts such as `pnpm dev`, `pnpm build`, `pnpm test`, `pnpm check`, or `pnpm run ci`.
5. Add repo-specific setup steps only when needed, such as env-file copies, Docker, merge drivers, generated native assets, or local tool installs.

Examples:

- Next.js: Corepack/pnpm guidance, `pnpm install`, `pnpm dev`, branch from `canary`, plus repo-specific lifecycle hooks such as `pnpm:devPreinstall` and `postinstall`.
- Vite: use pnpm from `packageManager`, run `pnpm i`, run `pnpm run build`.
- Astro: prerequisites, Corepack recommendation, `pnpm install`, `pnpm run build`, plus optional local git config and pnpm merge-driver setup.
- Supabase: install dependencies, copy env example, run `pnpm install`, run `pnpm dev`, and use Docker for Studio.
- Biome: because it is Rust-first, it uses `just install-tools` for Rust tooling and still requires `pnpm install` for Node/changeset tooling.

The common lesson is that the durable standard is not the script name. The durable standard is:

- one documented setup entrypoint
- pinned toolchain
- minimal bootstrapping
- repo-specific setup encoded in versioned scripts
- clear local verification command

## pnpm-Specific Findings

### Avoid `pnpm setup` as a project script

`pnpm setup` is already a pnpm CLI command for configuring `PNPM_HOME` and shell integration. pnpm can run project scripts with names that conflict with built-in commands, but that ambiguity is not desirable for our standard.

Recommendation: do not standardize on `pnpm setup` for project setup.

### Prefer `pnpm dev:setup`

`pnpm dev:setup` is explicit, unlikely to collide with pnpm built-ins, and clearly means "prepare this checkout for local development."

Suggested meaning:

- validate Node version
- enable/prepare Corepack
- install dependencies using pinned pnpm
- copy/link local env and cache files according to repo policy
- install browser/test tool prerequisites when needed
- print the resulting pnpm store path and next commands

### Add `pnpm worktree:new`

pnpm's own docs now document Git worktrees for multi-agent development and describe a helper pattern around `pnpm worktree:new <branch|pr>`. The current `zkochan/git-wt` package implements the same basic shape as a standalone `git wt` / shell `wt` helper.

Recommendation: use `pnpm worktree:new` as the worktree layer above `pnpm dev:setup`.

Suggested meaning:

- read repo policy for base branch and worktree dir
- create the worktree from the right base
- run `pnpm dev:setup` inside the new worktree
- print path, branch, base, and verify command

`git-wt` can be used as inspiration for this command's ergonomics:

- branch or PR number as the only required argument
- PR checkout support through `gh`
- branch-name-to-directory-name normalization
- optional shell helper that `cd`s into the new worktree
- optional PR hook for review agents

However, our command should be repo-aware in ways `git-wt` is not.

### Evaluate worktree layout

Our repos currently use nested `.worktrees/`. That is convenient and already gitignored.

However, nested worktrees are the wrong default for our agent-heavy workflow. Git can ignore `.worktrees/`, but the directories are still physically below the repo root. Tools that honor Git ignore or explicit ignore files may avoid them, but every scanner, formatter, test runner, file watcher, affected-package calculator, editor indexer, and agent context loader has to get that behavior right consistently.

Concrete local risk examples:

- workflow-kit has scripts such as `biome format --write .`, `biome format .`, and `biome lint .`.
- Pathway has `prettier --check .`.
- ghx uses Nx `run-many` and `affected` commands.
- OnClass has changed/test/check commands layered through Turbo.

Some local configs already mitigate parts of this: Pathway's `.prettierignore` excludes `.worktrees/`, workflow-kit's `.gitignore` excludes `.worktrees/`, and OnClass has several commands scoped to `src` or configured Turbo inputs. That reduces the blast radius, but it does not remove the architectural footgun. Agents and tools that do not honor those ignore files can still discover duplicate nested `AGENTS.md` files, re-read or over-apply instructions from sibling work, or run broad scans over inactive branches.

pnpm's own worktree docs and `git-wt` favor a bare repo hub plus sibling worktrees for multi-agent development. There is also a pnpm issue indicating nested worktrees can cause problems for `pnpm --filter '[<ref>]'` changed-package workflows: https://github.com/pnpm/pnpm/issues/12626.

Recommendation:

- Stop making nested `.worktrees/` the standard.
- Do not create new nested worktrees for agent work unless a repo has not yet been migrated and the task is low-risk.
- Make the durable invariant: no active worktree should live under another active worktree.
- Use an external sibling worktree root as the immediate target.
- For high-parallelism repos, use a bare Git hub plus sibling worktrees.
- Keep `.worktrees/` only as a legacy compatibility path during migration.
- Audit any `pnpm --filter '[<ref>]'`, `check:changed`, Nx affected, Turbo affected, Biome, Prettier, and agent context-loading behavior before declaring a repo migrated.
- Treat `git-wt` as evidence that sibling worktrees are a serious OSS/pnpm pattern, but keep our repo-owned setup and policy layer above it.

### Bare repo topology explained

A bare repository is a Git repository with no checked-out working files. It contains Git metadata only: `HEAD`, refs, config, objects, and worktree metadata.

Normal clone:

```text
repo/
  .git/
  package.json
  src/
  AGENTS.md
```

Bare hub plus sibling worktrees:

```text
repo-family/
  .bare/        # shared Git database; do not edit code here
  main/         # real checkout for main
  feature-a/    # real checkout for feature-a
  pr-123/       # real checkout for PR review
```

Each checkout has its own working files:

```text
repo-family/main/package.json
repo-family/feature-a/package.json
repo-family/pr-123/package.json
```

But all of them share one Git object database through the bare hub. This gives us most of the disk/time benefits of worktrees without placing one checkout inside another checkout.

Example setup:

```bash
mkdir -p ~/code/on-class-web
git clone --bare git@github.com:ORG/on-class-web.git ~/code/on-class-web/.bare
git --git-dir ~/code/on-class-web/.bare worktree add ~/code/on-class-web/main main
git --git-dir ~/code/on-class-web/.bare worktree add -b feature-x ~/code/on-class-web/feature-x main
```

After setup, humans and agents work inside `main/`, `feature-x/`, or `pr-123/`, not inside `.bare/`.

This avoids the nested failure mode:

```text
repo/
  AGENTS.md
  package.json
  .worktrees/
    feature-a/
      AGENTS.md
      package.json
```

The bare/sibling topology keeps each agent's root clean:

```text
repo-family/
  main/
    AGENTS.md
  feature-a/
    AGENTS.md
```

Running `biome .`, `prettier --check .`, `pnpm check`, `nx affected`, `turbo`, or an agent context scan from `feature-a/` cannot accidentally walk into `main/`, because `main/` is a sibling rather than a descendant.

### Target local layout

Use this as the practical target shape:

```text
/Users/aryekogan/code/<family-root>/
  <repo-name>/   # primary checkout, or v-next for workflow-kit primary work
  worktrees/
    <repo-name>/
      <branch>/
      pr-<number>/
```

The exact parent directory can change, but the invariant should not:

- the bare Git hub is not an active checkout
- primary and branch worktrees are siblings
- branch worktrees are not descendants of primary
- `pnpm worktree:new` owns the path convention
- repo setup still runs through `pnpm dev:setup`

### Portable workflow config shape

Do not commit user-specific absolute paths such as `/Users/aryekogan/code/...` into repo config.

If we later implement this standard, the committed config should describe topology and names, not the local machine's root directory. A portable shape could be:

```yaml
git:
  strategy: worktree
  baseBranch: main
  commitOnBase: forbid
  worktree:
    layout: bare-sibling
    bareDirName: .bare
    primaryName: main
    branchesDirName: wt
    pathEnv: CODE_WORKTREE_ROOT
```

For workflow-kit:

```yaml
git:
  strategy: worktree
  baseBranch: v-next
  commitOnBase: forbid
  worktree:
    layout: bare-sibling
    bareDirName: .bare
    primaryName: v-next
    branchesDirName: wt
    pathEnv: CODE_WORKTREE_ROOT
```

Resolution rules for `pnpm worktree:new`:

1. If `CODE_WORKTREE_ROOT` is set, use `$CODE_WORKTREE_ROOT/<repo-name>/`.
2. Else, if the Git common directory is a bare hub named `.bare`, use its parent as the repo-family root.
3. Else, if the current checkout path ends with the configured `primaryName`, use its parent as the repo-family root.
4. Else, refuse to create an external worktree and print the expected layout.

This avoids hardcoding a personal path while still giving the script a deterministic target.

Keep legacy compatibility separate:

```yaml
git:
  legacyWorktreeDir: .worktrees
```

`legacyWorktreeDir` should allow cleanup and inventory of old nested worktrees. It should not be used for new agent work unless a migration has not happened yet and the caller explicitly opts into legacy mode.

### Local repo reference state

The external repos are useful as reference evidence for why nested worktrees are risky, not as a
migration target. At the time of the report each carried several live nested `.worktrees/`
checkouts (OnClass, Pathway, and the retiring workflow-kit prototype). The specific inventory was
machine state with no archival value and is omitted here; the implications below are what matters.

Reference implications:

- OnClass and Pathway already encode `.worktrees` in `.workflow/config.yaml`, so any future standard must explicitly change the policy surface rather than relying on shell habits.
- workflow-kit has no `.workflow/config.yaml` today, but its `AGENTS.md` encodes `.worktrees/<name>` and `v-next`, so local instructions are the policy source to reconcile later.
- ghx has no current nested worktree state and no `.workflow/config.yaml`, making it the cleanest candidate for a future standard, but that implementation is out of scope for this report.
- Active local worktrees mean no future change should assume `.worktrees/` can simply be deleted or renamed.
- This report does not prescribe how to migrate these worktrees.

### pnpm store decisions

The bare/sibling topology solves the nested-checkout problem. It does not automatically standardize pnpm store policy across repos.

| Repo         | Current evidence                                                                                                         | Recommendation                                                                                                      |
| ------------ | ------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------- |
| OnClass      | No `enableGlobalVirtualStore` in `pnpm-workspace.yaml`; setup links `.turbo`/`.vercel` and installs Playwright Chromium. | Do not assume pnpm's fast-worktree benefit applies until store behavior is evaluated separately.                    |
| Pathway      | `enableGlobalVirtualStore: true`; setup links `.turbo`/`.vercel`, copies env, installs Playwright Chromium.              | Already aligns with pnpm's global virtual store direction; still needs repo-specific setup preserved.               |
| workflow-kit | `enableGlobalVirtualStore: true`; setup intentionally uses primary `v-next` checkout `.pnpm-store` and copies `.turbo`.  | Store behavior is intentionally repo-specific; any future topology change must preserve or deliberately replace it. |
| ghx          | No `enableGlobalVirtualStore`; no current worktree setup standard.                                                       | Use as a reference for the missing setup/worktree contract; do not assume global virtual store yet.                 |

pnpm's docs also include a trust-boundary warning: do not share one writable pnpm store across mutually untrusted agents or users. Our local agent work generally shares one developer trust boundary, but the standard should still make this explicit.

## Recommended Standard

### Standard command surface

Use this as the repo-level interface:

```json
{
  "scripts": {
    "dev:setup": "bash scripts/dev-setup.sh",
    "worktree:new": "node scripts/worktree-new.mjs",
    "check": "...",
    "dev": "..."
  }
}
```

Compatibility wrappers are fine:

```bash
scripts/setup-worktree.sh
```

can become:

```bash
#!/usr/bin/env bash
set -euo pipefail
bash scripts/dev-setup.sh --worktree "$@"
```

### Standard policy sources

Use this precedence:

1. `AGENTS.md` for human/agent contract and repo-specific rules.
2. `.workflow/config.yaml` for machine-readable branch/worktree/verify policy.
3. `scripts/dev-setup.sh` for checkout setup.
4. `scripts/worktree-new.mjs` for worktree creation.
5. Optional local tool config such as `git-worktree-runner` or `git-wt` only as convenience adapters.

Avoid making `.gtrconfig`, `.git-wt` hooks, or shell-local helper config another required source of truth unless generated from `.workflow/config.yaml` or kept deliberately minimal.

### Standard `dev:setup` responsibilities

Every repo's setup script should:

- validate it is at the repo root
- validate Node version
- enable Corepack
- install with frozen lockfile
- preserve ignored local secrets
- never overwrite existing env files
- print the effective package manager and store path
- be idempotent
- avoid destructive cleanup

Repo-specific additions:

- OnClass: copy env and seed env, link `.turbo` and `.vercel`, install Playwright Chromium.
- Pathway: copy env, link `.turbo` and `.vercel`, install Playwright Chromium, preserve Supabase/Vercel production-secret boundaries.
- workflow-kit: copy `.turbo`, use primary `v-next` checkout `.pnpm-store`, preserve isolated Turbo logs, support `--warn-as-error`.
- ghx: add setup script parity; likely `setup-dev-env`, frozen install, build/ci guidance, and optional worktree bootstrap.

### Standard `worktree:new` responsibilities

The worktree script should:

- read base branch, topology, primary name, bare dir name, and branch worktree directory name from `.workflow/config.yaml` when present
- fall back to repo defaults:
  - OnClass/Pathway/ghx: `main`
  - workflow-kit: `v-next`
- refuse to create a new worktree inside an active checkout unless explicitly allowed for legacy compatibility
- prefer an external sibling layout, ideally under the repo-family root managed by the bare hub
- refuse to run when the main checkout has unsafe local state unless explicitly allowed
- create the worktree under the resolved sibling branch directory, such as
  `<repo-family>/worktrees/<repo>/<branch>`
- run `pnpm dev:setup` in the new worktree
- print:
  - repo
  - branch
  - base ref
  - worktree path
  - setup result
  - verification command
- optionally accept a PR number and use GitHub state to fetch the right head branch, preferably through `ghx` where review/thread state matters and raw `gh` where fetch/checkout requires it
- optionally expose a shell `wt` helper that changes into the created worktree

### Role for git-worktree-runner

Use `git-worktree-runner` optionally for:

- manual navigation
- `git gtr run <branch> <command>`
- editor launch
- AI launch
- non-authoritative list/remove helpers

Do not use it as the source of truth for:

- branch base policy
- repo setup policy
- PR closeout
- review-thread handling
- production/deployment operations

If piloted, configure it locally first:

```bash
git gtr config set gtr.worktrees.dir ../wt
git gtr config set gtr.defaultBranch main
git gtr config add gtr.hook.postCreate "pnpm dev:setup"
git gtr config set gtr.ai.default codex
```

For workflow-kit, use `v-next` instead of `main`. Treat `../wt` here as an example from a primary checkout such as `<repo-family>/main` or `<repo-family>/v-next`, not as committed repo policy.

### Role for git-wt

Use `git-wt` as:

- a reference implementation for simple branch/PR worktree creation
- a possible personal shell helper if a repo already uses the bare hub plus sibling-worktree layout
- inspiration for our `pnpm worktree:new <branch|pr>` UX
- inspiration for a checked-in `.git-wt/pr-hook`-style review-agent hook, if we want one

Do not use `git-wt` directly as the repo standard unless the repo has explicitly adopted its assumptions:

- bare repository hub
- sibling worktrees
- `devEngines.packageManager.name` in `package.json`
- plain install is sufficient setup
- default branch from `origin/HEAD` is the correct base
- force cleanup is acceptable

Those assumptions are currently false or incomplete for the target repos.

## Future Work Boundaries

This report is advisory. It uses local repos as reference evidence and recommends a target standard. It does not plan or execute migrations for the current local worktrees.

### Standard definition

A future standard doc, likely under shared agent/dev guidance, should define:

- `pnpm dev:setup`
- `pnpm worktree:new`
- no nested active worktrees
- bare hub plus sibling worktrees as the target topology
- required outputs
- idempotency and secret rules
- `.workflow/config.yaml` as machine-readable branch/worktree policy
- `AGENTS.md` as human/agent policy

### Topology decision criteria

Before changing any repo, decide the portable path convention. A candidate shape is:

```text
/Users/aryekogan/code/<family-root>/
  <repo-name>/           # primary checkout, or v-next for workflow-kit
  worktrees/<repo>/<branch-or-pr>/
```

Use the local repos only as reference checks:

- no active worktree is inside another active worktree
- broad format/lint/test commands do not traverse sibling worktrees
- agent context discovery sees only the current checkout's instruction files
- pnpm changed filters, Nx affected, and Turbo affected behavior are verified from the new layout
- legacy `.worktrees/` state is recognized as current-state evidence, not silently ignored

### ghx reference gap

ghx shows the missing pieces a standard should eventually cover:

- `AGENTS.md`
- `.workflow/config.yaml`
- `scripts/dev-setup.sh`
- `scripts/worktree-new.mjs` or a shared wrapper
- optional `scripts/setup-worktree.sh` compatibility wrapper

This report does not implement those changes.

### OnClass and Pathway reference

OnClass and Pathway show what must be preserved by any future standard:

- existing env handling
- `.turbo` and `.vercel` handling
- Playwright Chromium setup
- `.workflow/config.yaml` policy
- `main` as base branch

This report does not plan the mechanical conversion of their current `.worktrees` config.

### workflow-kit reference

workflow-kit shows why a single generic worktree runner is not enough. Any future standard must preserve:

- `v-next`
- primary `.pnpm-store`
- copied `.turbo`
- isolated logs
- `pnpm check`
- strict branch and PR rules

This report does not plan workflow-kit's topology migration.

### Optional tool pilot

Use `git-worktree-runner` and/or `git-wt` locally, not as committed standards, after the repo-owned scripts exist.

Evaluate:

- Does it reduce manual commands?
- Does it drift from `.workflow/config.yaml`?
- Does hook trust create friction?
- Does cleanup stay safe enough compared with current `ghx`-aware closeout?
- Does `git-wt`'s sibling-worktree topology improve pnpm changed-filter behavior enough to justify migration?
- Does `git-wt`'s PR hook model improve review-agent launch without bypassing repo instructions?

Only commit `.gtrconfig` if it proves useful and stays minimal.

Only commit `.git-wt/pr-hook` if it stays a thin launcher and the prompt delegates to repo instructions, `ghx` review-thread inspection, and local verification.

## Final Recommendation

Adopt a repo-owned setup standard, not a third-party worktree runner standard.

The recommended durable shape is:

1. `pnpm dev:setup` is the canonical checkout setup command.
2. `pnpm worktree:new <branch-or-pr>` is the canonical worktree creation command.
3. `pnpm worktree:clean <branch>` is the canonical post-merge worktree cleanup command.
4. New worktrees use grouped external sibling paths: `worktrees/<repo>/<branch>`.
5. Nested `.worktrees/` is legacy only; new worktrees should be external siblings, preferably through a bare Git hub.
6. Existing `scripts/setup-worktree.sh` scripts become compatibility wrappers.
7. `.workflow/config.yaml` owns machine-readable worktree/base/verify policy.
8. `AGENTS.md` owns human and agent behavior.
9. `git-worktree-runner` remains optional local ergonomics.
10. `git-wt` is the strongest OSS/pnpm reference for the desired command UX and topology, but remains optional until wrapped by repo policy.

This matches OSS practice: pin the toolchain, document setup, keep project-specific setup in repo scripts, and add worktree helpers only above that foundation.

The updated recommendation after reviewing `git-wt` and the nested-worktree failure modes: borrow the pnpm-style worktree model more confidently, stop treating nested `.worktrees/` as the steady-state layout, and keep the standard repo-owned. `git-wt` should influence the shape of `pnpm worktree:new`; it should not become the authoritative setup layer for OnClass, Pathway, workflow-kit, or ghx.
