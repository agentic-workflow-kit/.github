# agentic-workflow-kit community files

This repository owns the public organization profile and default community health files for
[`agentic-workflow-kit`](https://github.com/agentic-workflow-kit).

- [`profile/README.md`](profile/README.md) renders on the organization landing page.
- [`CONTRIBUTING.md`](CONTRIBUTING.md), [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md), and
  [`SECURITY.md`](SECURITY.md) provide defaults for repositories that do not define their own.
- [Issue templates](.github/ISSUE_TEMPLATE) and the
  [pull request template](.github/PULL_REQUEST_TEMPLATE.md) provide the default contribution flow.
- [`REPO-STRUCTURE.md`](REPO-STRUCTURE.md) defines the default repository layout:
  `README.md`, `docs/product`, `docs/design`, and source tier.

Repository-specific files override these defaults when a project needs its own policy.

## Development

This repo is docs and community-health configuration only. Validate it with:

```bash
pnpm install --frozen-lockfile
pnpm check
```

The `check` workflow runs the same command for pull requests and pushes to `main`.
