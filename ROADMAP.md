# Roadmap — agentic-workflow-kit

This is the org-level map of the lifecycle: which layers exist, which are planned, what each
**owns**, what it **depends on**, and the **references** each one designs from. It is
deliberately lightweight. It does not create repos and it does not author any layer's product
content — its job is to let each owner pick up its slice and run `product -> design ->
implementation` independently.

The org contract is [`AGENTS.md`](./AGENTS.md); the repo layout standard is
[`REPO-STRUCTURE.md`](./REPO-STRUCTURE.md).

For execution order, use [`MILESTONES.md`](./MILESTONES.md). This roadmap names the layers
and seams; milestones define the sequence of org outcomes and the handoff model each repo
uses to derive its local plan.

---

## The lifecycle

```text
PRODUCT ---------> DESIGN ----------> PLANNING --------> DELIVERY --------> LEARNING
define / PRD       technical-design   design-to-plan     jig (run)          feedback loop
[seeded]           [built]            [seeded]           [early]            [planned]
```

Each stage produces a **durable, structured artifact** that is the next stage's input. Four stages
already exist as lifecycle repos (`define-product`, `technical-design`, `design-to-plan`, `jig`);
one is planned (Learning). `.github` is org infrastructure, not a lifecycle stage. Of the existing
repos, `define-product` is seeded, `technical-design` is built, `design-to-plan` is seeded, and
`jig` is early.

> Planning is its own layer because it has a separate repo and product/design/implementation arc. It
> still owns no upstream seam: it consumes Product and Technical Design contracts and produces to
> Jig's execution-plan contract shape.

---

## Why this parallelizes — depend on seams, not layers

The layers do **not** form a build chain where each must wait for the previous. They are
coupled only by a small set of **shared contracts (seams)**. Pin down each seam's shape early
and every layer can be designed in parallel against the contract, not against another layer's
internals. The currently pinned seams are owned by existing layer repos.

| Seam (shared artifact)                                            | Owner              | Consumers                      | Status                                                                                                                           |
| ----------------------------------------------------------------- | ------------------ | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------- |
| **Execution-plan contract shape** — Jig's one hard input boundary | `jig`              | Planning layer produces to it  | v0 shape: `jig/docs/design/contracts/execution-plan-contract-v0.md`; M7 real-driver spine merged through Jig PR #39              |
| **Observability / event records** — durable run output            | `jig`              | Learning loop consumes         | v0 shape: `jig/docs/design/contracts/observability-records-contract-v0.md`; M7 records-integrity spine merged through Jig PR #39 |
| **Technical-design document format**                              | `technical-design` | Planning layer consumes        | v0 handoff: `technical-design/docs/design/technical-design-handoff-contract.md`                                                  |
| **PRD / ID'd acceptance-criteria format**                         | `define-product`   | Design + Planning cite the IDs | v0 contract: `define-product/docs/product/prd-contract.md`                                                                       |

**Sequencing rule of thumb:** the highest-leverage early work is authoring Jig's two seams
(execution-plan contract shape, observability records), because two downstream layers wait on their
_shape_ — not their implementation. Define the contracts first; build behind them in
parallel. The current sequence is tracked in [`MILESTONES.md`](./MILESTONES.md).

---

## Reference conventions

- **Legacy prototype** = `workflow-kit` (local at `~/repos/workflow-kit`, org `workflow-kit`).
  It is **retiring and reference-only** — a source of lessons and prior engineering, never an
  authority. Paths below are relative to that repo.
- **v0.7.0 prior art** = the shipped `agentic-workflow-kit` plugin skills (installed locally;
  invokable as `agentic-workflow-kit:<skill>`). These are working reference implementations of
  the planned upstream layers — design from them, do not assume they are the target shape.
- Curate references **per layer**; do not import the whole legacy corpus into a repo.
- **New layers start fresh, to standard.** Scaffold a new repo from `repo-template` (inheriting
  the `AGENTS.md` contract, tooling, and `pnpm check` gate), then define its product first
  (`define-product` / PRD with ID'd acceptance criteria) before design and implementation. The
  legacy prototype and plugin skills inform this only as reference.

---

## Layers

### technical-design — Design stage `[built]`

- **Role:** frame, author (DDD-first), review-loop, enforce, and orchestrate technical designs.
- **Owns the seam:** the technical-design document format the Planning layer consumes.
- **Depends on:** nothing upstream is required; consumes a brief/PRD when available.
- **Next step:** harden and dogfood; optionally deepen its own `docs/design/`. Proceeds
  **independently** of every other track.
- **References:** its own [`docs/design/`](https://github.com/agentic-workflow-kit/technical-design/tree/main/docs/design)
  (profile contract, formats, lessons ledger); legacy `docs/product/supporting-products/product-to-design.md`;
  v0.7 skill `agentic-workflow-kit:design-technical-solution`.

### jig — Delivery / execution engine `[early; walking skeleton running]`

- **Role:** run an approved execution plan under policy into reviewed, landed work — or a
  deliberate, inspectable stop.
- **Owns the seams:** the **execution-plan contract shape** and the **observability / event
  records**. These are the highest-leverage seams in the org; author them early and version them
  deliberately.
- **Depends on:** a valid execution plan (its one hard input boundary). Upstream layers are
  optional strong defaults, not prerequisites.
- **Next step:** the design layer is live (both seam contracts, state tables, and an ADR log
  reconciling to product), and the local delivery spine now runs from the M5b walking
  skeleton through the M7 real-provider track. Jig PR #39 merged the Phase 9
  records-integrity closeout at `3b3d224`, completing the M7 repo-track spine for real
  agent/host, Forge, work-source, and records-integrity behavior behind the same contracts.
  Per the 2026-07-04 post-M7 sequencing decisions (`MILESTONES.md`), the tail is now
  replanned rather than open-ended: a Codex-transport evidence-capture track and an internal
  SDK packaging boundary (no publishing) are authorized to start; a first-party TUI/dashboard
  stays an extension-point-only non-goal; and any v0 contract freeze still waits on M7 exit
  evidence plus the packaging and transport decisions.
- **References (curated):**
  - Product (own, drafted): `jig/docs/product/jig.md`, `guarantees.md`, `use-cases.md`,
    `concepts.md`.
  - Legacy design corpus (reference only): `docs/design/10-architecture/` (architecture,
    `provider-seams.md`, `event-log-and-state.md`, `evidence-gates-and-merge.md`,
    `capability-attestation.md`, `recovery-and-reconciliation.md`),
    `docs/design/20-sdk-and-packaging/`, `docs/design/30-domain-reference/`,
    `docs/design/40-decisions/accepted-decisions.md`.
  - Legacy gate/CI mechanics (reference only): `docs/engineering/`.
  - Legacy implementation sequencing (reference only): `docs/implementation/` (epic/domain DAGs).

### define-product — Product layer `[seeded]`

- **Role:** help an owner produce a PRD with ID'd acceptance criteria the downstream layers
  reference.
- **Owns the seam:** the PRD / acceptance-criteria-ID format.
- **Depends on:** nothing upstream. Sits at the head of the lifecycle.
- **Next step:** keep future Product-layer design and implementation derived from its own product
  docs. Its outward contract is the acceptance-criteria-ID format that Design and Planning cite.
- **References:** its own
  [`docs/product/prd-contract.md`](https://github.com/agentic-workflow-kit/define-product/blob/main/docs/product/prd-contract.md)
  and
  [`docs/product/examples/minimal-prd.md`](https://github.com/agentic-workflow-kit/define-product/blob/main/docs/product/examples/minimal-prd.md);
  legacy `docs/product/supporting-products/define-product.md`; v0.7 skill
  `agentic-workflow-kit:define-product`.

### design-to-plan — Planning layer `[seeded]`

- **Role:** decompose a technical design into a Jig-ready execution plan in the expected contract
  shape.
- **Owns the seam:** none new — it **produces to** Jig's execution-plan contract shape and
  **consumes** the Product PRD / acceptance-criteria-ID contract plus the technical-design document
  format.
- **Depends on (contract, not internals):** `define-product`'s PRD / acceptance-criteria-ID
  contract, Jig's execution-plan contract shape, and technical-design's document format.
- **Next step:** keep future Planning implementation work derived from its own product and design
  docs. Do not freeze Jig's field-level schema from Planning.
- **References:** its own
  [`docs/product/design-to-plan.md`](https://github.com/agentic-workflow-kit/design-to-plan/blob/main/docs/product/design-to-plan.md),
  [`docs/design/design-to-plan-contract.md`](https://github.com/agentic-workflow-kit/design-to-plan/blob/main/docs/design/design-to-plan-contract.md),
  and
  [`docs/design/examples/minimal-design-to-plan.md`](https://github.com/agentic-workflow-kit/design-to-plan/blob/main/docs/design/examples/minimal-design-to-plan.md);
  legacy `docs/product/supporting-products/design-to-plan.md`; legacy
  `docs/implementation-authoring/delivery-pipeline/` and
  `docs/implementation-authoring/authoring-standard/`; v0.7 skill
  `agentic-workflow-kit:plan-delivery-track` (and `workflow-init`).

### Learning loop — feedback `[planned]`

- **Role:** suite-level, between-runs tool that consumes Jig's run records and, via human-led
  root-cause retro, hardens the earliest layer that should have prevented a defect (promoting
  recurring defects into mechanical checks). Stays out of Jig's per-run hot path.
- **Owns the seam:** none new — it **consumes** Jig's observability / event records.
- **Depends on (contract, not internals):** Jig's observability records. Define that seam shape
  and Learning can be designed in parallel.
- **Next step (when picked up):** define product -> design -> implement, against Jig's records
  contract.
- **References:** legacy `docs/product/supporting-products/learning-loop.md`; legacy
  `docs/implementation-authoring/lessons-ledger.md`.

---

## What can start in parallel now

1. **`jig` design** — author `docs/design/`, leading with the two seams (execution-plan contract
   shape, observability records). Highest leverage: unblocks Planning and Learning by shape.
2. **`technical-design` hardening** — fully independent.
3. **`define-product`** — product contract is seeded; future design and implementation can derive
   from its own product docs without blocking M4.
4. **`design-to-plan`** — seeded against the Product PRD / acceptance-criteria-ID contract, Jig's
   execution-plan contract shape, and technical-design's document format. Future implementation work
   derives from its own product and design docs.
5. **Learning loop** — designable in parallel once Jig's records seam is pinned.

The only true ordering constraint is _contract-shape_, not _implementation_: Planning needs the
**shape** of Product, Technical Design, and Jig input/output seams, and Learning needs the
**shape** of Jig's records seam, not finished upstream implementations.

Repo-level plans should derive from the active milestone rather than expanding this roadmap into
a centralized backlog. The repo owner records its local plan in that repo, including what it owns,
what it consumes, and what it must not decide.

---

## Deferred (not scoped here)

Hosted multi-tenant operation; multi-project orchestration; LLM-adjudicated approval autonomy;
legacy run migration. These are recorded as out-of-scope for the current build-out, consistent
with the legacy roadmap and the drafted Jig product layer.
