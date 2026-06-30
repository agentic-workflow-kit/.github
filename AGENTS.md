# AGENTS.md — .github (org standards & community health)

The contract for working in this repo. **Self-contained:** act on it with only this repo
checked out.

This is the agentic-workflow-kit org's standards and community-health repo. It produces
documentation and policy, not a runtime artifact, and it is the home of the cross-repo
standards:

- **`REPO-STRUCTURE.md`** — repo layout, package/check standard, Node/pnpm policy, repo
  settings, and the self-contained `AGENTS.md` contract standard.
- **`ROADMAP.md`** — the lifecycle, the seam-ownership map, and per-layer status and references.
- **`MILESTONES.md`** — the sequence-based org planning source that repo owners derive
  local plans from.
- **`profile/`** and the shared `CONTRIBUTING` / `CODE_OF_CONDUCT` / `SECURITY` / templates.

When a standard changes, change it **here**. Each product repo keeps a self-contained `AGENTS.md`
that restates the small shared baseline inline rather than depending on this repo — so don't turn
these standards into something another repo must fetch at runtime.

## Gate and conventions

- **`pnpm check`** before claiming any change done; show its output as evidence. The gate is
  `prettier --check` over Markdown/YAML/JSON. Keep standards lean and non-duplicative.
- **`main`-based:** branch from `main`, PR into it, green `check` required, review conversations
  resolved, squash-merge. Conventional commit subjects; no attribution footers; no emojis
  anywhere. Worktrees for non-trivial work are external siblings of this checkout — never nested
  inside it.
