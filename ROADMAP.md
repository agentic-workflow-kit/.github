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
define / PRD       technical-design   design -> plan     jig (run)          feedback loop
[planned]          [built]            [planned]          [early]            [planned]
```

Each stage produces a **durable, structured artifact** that is the next stage's input. Two
stages already exist as repos (`technical-design`, `jig`); three are planned (Product,
Planning, Learning). `.github` is org infrastructure, not a lifecycle stage.

> The org `profile/README.md` shows a **four-stage** suite spine that folds Planning under the
> delivery handoff (`plan -> jig (run)`). This roadmap breaks Planning out as its **own layer**
> because it is a separate planned repo with its own product/design/implementation arc. Same
> lifecycle, different altitude — no contradiction.

---

## Why this parallelizes — depend on seams, not layers

The layers do **not** form a build chain where each must wait for the previous. They are
coupled only by a small set of **shared contracts (seams)**. Pin down each seam's shape early
and every layer can be designed in parallel against the contract, not against another layer's
internals. Most seams are owned by the two layers that already exist.

| Seam (shared artifact)                                    | Owner              | Consumers                      | Status                                                           |
| --------------------------------------------------------- | ------------------ | ------------------------------ | ---------------------------------------------------------------- |
| **Execution-plan schema** — Jig's one hard input boundary | `jig`              | Planning layer produces to it  | v0 shape: `jig/docs/design/execution-plan-contract-v0.md`        |
| **Observability / event records** — durable run output    | `jig`              | Learning loop consumes         | v0 shape: `jig/docs/design/observability-records-contract-v0.md` |
| **Technical-design document format**                      | `technical-design` | Planning layer consumes        | Exists (`technical-design/docs/design/`)                         |
| **PRD / ID'd acceptance-criteria format**                 | Product layer      | Design + Planning cite the IDs | Planned (v0.7 `define-product` is prior art)                     |

**Sequencing rule of thumb:** the highest-leverage early work is authoring Jig's two seams
(execution-plan schema, observability records), because two downstream layers wait on their
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

### jig — Delivery / execution engine `[early; product drafted]`

- **Role:** run an approved execution plan under policy into reviewed, landed work — or a
  deliberate, inspectable stop.
- **Owns the seams:** the **execution-plan schema** and the **observability / event records**.
  These are the highest-leverage seams in the org; author them early and version them
  deliberately.
- **Depends on:** a valid execution plan (its one hard input boundary). Upstream layers are
  optional strong defaults, not prerequisites.
- **Next step:** author `docs/design/` **fresh** (reconciling to the drafted product layer),
  then implement. The package decomposition is design-owned and intentionally empty until
  then.
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

### Product layer — define-product / PRD `[planned]`

- **Role:** help an owner produce a PRD with ID'd acceptance criteria the downstream layers
  reference.
- **Owns the seam:** the PRD / acceptance-criteria-ID format.
- **Depends on:** nothing upstream. Sits at the head of the lifecycle.
- **Next step (when picked up):** define product -> design its own skills/artifacts ->
  implement. Can start **now**, in parallel; its only outward contract is the
  acceptance-criteria-ID format that Design and Planning cite.
- **References:** legacy `docs/product/supporting-products/define-product.md`; v0.7 skill
  `agentic-workflow-kit:define-product`.

### Planning layer — design -> plan `[planned]`

- **Role:** decompose a technical design into a Jig-ready execution plan in the expected schema.
- **Owns the seam:** none new — it **produces to** Jig's execution-plan schema and **consumes**
  the technical-design document format.
- **Depends on (contract, not internals):** Jig's execution-plan schema; technical-design's
  document format. Both owners already exist — define those two seam shapes and Planning can be
  designed in parallel.
- **Next step (when picked up):** define product -> design -> implement, against the two seam
  contracts above.
- **References:** legacy `docs/product/supporting-products/design-to-plan.md`; legacy
  `docs/implementation-authoring/delivery-pipeline/` and `docs/implementation-authoring/authoring-standard/`;
  v0.7 skill `agentic-workflow-kit:plan-delivery-track` (and `workflow-init`).

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

1. **`jig` design** — author `docs/design/`, leading with the two seams (execution-plan schema,
   observability records). Highest leverage: unblocks Planning and Learning by shape.
2. **`technical-design` hardening** — fully independent.
3. **Product layer** — fully independent; can be picked up and designed today.
4. **Planning layer** — designable in parallel once Jig's execution-plan schema and
   technical-design's document format are pinned (both owners exist).
5. **Learning loop** — designable in parallel once Jig's records seam is pinned.

The only true ordering constraint is _contract-shape_, not _implementation_: Planning and
Learning need the **shape** of Jig's seams, not finished Jig code.

Repo-level plans should derive from the active milestone rather than expanding this roadmap into
a centralized backlog. The repo owner records its local plan in that repo, including what it owns,
what it consumes, and what it must not decide.

---

## Deferred (not scoped here)

Hosted multi-tenant operation; multi-project orchestration; LLM-adjudicated approval autonomy;
legacy run migration. These are recorded as out-of-scope for the current build-out, consistent
with the legacy roadmap and the drafted Jig product layer.
