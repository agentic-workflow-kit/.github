# pnpm Infra Hardening Review — 2026-06-30

A best-in-class pass over the org's pnpm/dependency infrastructure, done **before** real engine code
lands so the hardening is cheap. Upgraded to **pnpm 11.9.0** and reviewed eleven pnpm docs pages;
every load-bearing setting below was **verified against the live docs** and validated end-to-end on
pnpm 11.9.0 / Node 26.4.0 (engine sandbox: `pnpm check` exit 0 with the hardening block active).

This document is the durable record; the operative rules live in [`REPO-STRUCTURE.md`](../REPO-STRUCTURE.md).

## Toolchain

- **pnpm 11.9.0** (was 11.5.1), pinned via `packageManager` for Corepack.
- **Node floor `>=22.13.0`** (was `>=22`). Not arbitrary: pnpm 11.9 itself requires `>=22.13`, and the
  engine toolchain's `vite@8` (via vitest 4) requires `>=22.12`. CI pins `node-version: 22.13.0`;
  `.nvmrc` stays `26` (local dev line).
- `engines.pnpm: ">=11.9.0"` and `devEngines.runtime` (node `^26`, `onFail: warn`) added. Do **not**
  add `devEngines.packageManager` next to the `packageManager` field — pnpm warns and ignores
  `packageManager`, which Corepack needs.

## Where settings live in pnpm 11 (verified)

- The **`pnpm` field in `package.json` is no longer read** — all behavioral settings move to
  `pnpm-workspace.yaml`. Every repo carries one (single-package: omit `packages:`).
- **`.npmrc` is auth-only.** A committed `${ENV_VAR}` token placeholder is inert since 11.5.3 — never
  commit tokens; use `actions/setup-node` `registry-url` for CI auth.
- `onlyBuiltDependencies` / `neverBuiltDependencies` / `ignoredBuiltDependencies` and
  `packageManagerStrict*` are **removed in v11** — replaced by `allowBuilds` and `pmOnFail`.

## Adopted — supply-chain baseline (every repo's `pnpm-workspace.yaml`)

```yaml
allowBuilds: {} # build scripts off until reviewed (strictDepBuilds default true)
minimumReleaseAge: 1440 # 1-day publish-age gate (v11 default, explicit); 10080 = 1 week
minimumReleaseAgeExclude: []
engineStrict: true # engines.node becomes a hard gate
nodeVersion: "22.13.0" # evaluate dep engines against the floor
pmOnFail: error # running pnpm must match the packageManager pin
```

Engine repos add: `savePrefix: ''` (exact pins), `verifyDepsBeforeRun: warn`,
`strictPeerDependencies: true`. Validated: `allowBuilds: {}` needs **no entries** for
biome/TypeScript/vitest/esbuild/vite — none run install scripts.

## Adopted — CI

`actions/setup-node@v6` with `cache: pnpm` (built-in pnpm store cache; replaces the manual
`actions/cache` + `PNPM_STORE_DIR` dance), `pnpm/action-setup@v6` `version: 11.9.0`, `node-version:
22.13.0`, `pnpm install --frozen-lockfile`, `pnpm check`. pnpm 11 also **fails CI on a lockfile from a
newer pnpm major** — a free safety net.

## Adopted — TypeScript / Git

- **Never set `preserveSymlinks: true`** (breaks type resolution through pnpm's symlinks; use
  `nodeLinker: hoisted` if ever forced). `tsconfig.base.json` is already compliant.
- Resolve `pnpm-lock.yaml` merge conflicts with `pnpm install` + commit (pnpm auto-merges; review the
  diff). `.gitattributes` marks the lockfile generated/`-diff`.

## Decisions

- **enableGlobalVirtualStore: opt-in per engine repo, validated.** Verified ESM caveat: it does not
  work for ESM deps that import packages not declared in their own `package.json` (Node ignores
  `NODE_PATH` under ESM). Engine packages are ESM, so enable it for multi-worktree speed only after the
  gate stays green; remove it on resolution errors. (Softened from an earlier blanket mandate.)
- **minimumReleaseAge: start at 1 day**, raise to 7 (`10080`) once the dep set stabilizes — a 7-day
  gate would block pulling current deps while we are actively adding them.
- **resolutionMode: keep `highest`** (default). `lowest-direct` is a library-author technique, wrong
  fit for non-publishing engines. `time-based` exists if a supply-chain-flavored mode is wanted later.
- **Catalogs** (`catalog:` in `pnpm-workspace.yaml`): adopt as the single source for shared
  biome/TS/vitest versions once ≥2 engine repos exist; agree the `catalog:` (default, unnamed)
  convention now.

## Deferred (named so adoption is turnkey)

- **Changesets** for versioning/changelog/publish once a repo drops `private: true`
  (`@changesets/cli` + `changesets/action` on `main`, `pnpm publish -r`, npm provenance).
- **Docker**: multi-stage + `corepack enable` + `pnpm fetch` (cache-friendly layers) or BuildKit
  `--mount=type=cache,id=pnpm,target=/pnpm/store`, and `pnpm deploy` for slim images — when a repo
  ships a container.
- **SHA-pinned Actions**: apply with a pinning tool (pinact) or Dependabot, not hand-copied hashes.
- **`.pnpmfile.cjs/.mjs`**: reach-for-it only — `readPackage` cannot block install scripts (that is
  `allowBuilds`); add a local pnpmfile only to patch a specific broken transitive dependency.

## Pages reviewed

git-worktrees, using-changesets, continuous-integration, supply-chain-security, typescript, git,
docker, package_json, settings, npmrc, pnpm-workspace_yaml, pnpmfile.
