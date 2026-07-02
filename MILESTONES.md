# Milestones — agentic-workflow-kit

This is the org-level planning sequence. It turns the lifecycle in
[`ROADMAP.md`](./ROADMAP.md) into ordered, outcome-focused milestones that repo owners can
derive local plans from later.

`ROADMAP.md` answers: what layers exist, which seams they own, and why the work can
parallelize. This file answers: what sequence of org outcomes should we pursue next.

Milestones are intentionally **sequence-based, not calendar-based**. A milestone is complete
when its exit criteria are met and evidenced, not when a date arrives.

---

## Planning Rules

- **Outcome before output.** Name the user, operator, or downstream repo outcome first; treat
  features, docs, packages, and CLIs as candidate ways to achieve it.
- **Seams before layers.** Pin shared artifact shapes before building large surfaces behind
  them. Consumers need stable contracts earlier than finished implementations.
- **One primary owner.** Every milestone has one owner repo, even when several repos
  participate.
- **Repo plans derive later.** Org milestones do not contain story lists, package
  decompositions, implementation tasks, or sprint commitments. Those belong in the owner
  repo's local plan.
- **Explicit entry and exit.** A milestone is ready to plan when its entry criteria are true.
  It is done only when its exit criteria have evidence.
- **Assumptions are testable.** Each milestone names the load-bearing assumptions that should
  be tested or reviewed before implementation commits harden around them.
- **Sequence is not serialization.** Later milestones can do discovery in parallel, but they
  should not publish repo-local implementation plans until the seams they consume are pinned.
- **Reference, don't copy.** The legacy `workflow-kit` plugin (`agentic-workflow-kit:<skill>`)
  and the local `../workflow-kit` prototype are reference only — read them for prior art, never
  port them in. Re-derive every artifact against the current org standards.
- **Define the product first.** A milestone that stands up a new layer defines its product
  clearly before design or implementation, using the same product standard as every other layer
  (`define-product` / a PRD with ID'd acceptance criteria). New repos are scaffolded from
  `repo-template` and adopt its tooling and the standard `pnpm check` gate.

## Milestone Record Format

Use this shape for new org milestones:

```md
### Mx: Milestone Name

- State:
- Outcome:
- Why now:
- Primary owner:
- Participating repos:
- Owned seam or artifact:
- Entry criteria:
- Exit criteria:
- Artifacts:
- Repo planning handoff:
- Risks / kill assumptions:
- Evidence when landed:
```

Suggested states are `proposed`, `current`, `ready-for-repo-planning`, `done`, and
`deferred`.

## Deriving Repo Plans

When a repo picks up an org milestone, create a local planning artifact in that repo using
this minimal handoff:

```md
# Repo Plan for Mx

- Org milestone:
- This repo owns:
- This repo consumes:
- This repo must not decide:
- Local artifacts to change:
- Local exit criteria:
- Cross-repo dependencies:
- Verification:
- Open questions to send back to the org roadmap:
```

The handoff boundary matters: repo plans may refine local architecture, stories, packages,
tests, and sequencing, but they should not silently change org-owned seams. If a repo plan
finds an org seam is wrong, send that finding back to this file and `ROADMAP.md`.

When a milestone stands up a **new layer in a new repo**, the derivation has two extra steps
before any design or implementation:

1. **Scaffold from `repo-template`.** Create the repo from the template ("Use this template" or
   `apply-repo-standard.sh`) so it inherits the self-contained `AGENTS.md` contract, the
   `pnpm check` gate, and the shared tooling. New repos are not hand-rolled.
2. **Define the product first.** Produce the layer's product brief / PRD with ID'd acceptance
   criteria using the same product standard as every other layer, then derive design and
   implementation from it. The legacy plugin skills and `../workflow-kit` inform this as
   reference only — they are not the starting artifact.

---

## Sequence

### M0: Org Planning Baseline

- State: done
- Outcome: Enable each repo owner to derive local plans from one org sequence so planning
  does not fragment across repositories.
- Why now: The org now has four repos, the lifecycle is documented, and the next work crosses
  repo-owned seams.
- Primary owner: `.github`
- Participating repos: `.github`, `technical-design`, `jig`, `repo-template`
- Owned seam or artifact: Org milestone format and derivation rules
- Entry criteria:
  - `ROADMAP.md` names the lifecycle, layers, and seam owners.
  - The repos are cleanly separated and independently usable.
- Exit criteria:
  - `MILESTONES.md` exists and is linked from the org entry points.
  - The sequence is outcome-focused and contains no repo-local story backlog.
  - The repo derivation contract is explicit.
- Artifacts:
  - `MILESTONES.md`
  - Links from `README.md`, `ROADMAP.md`, `AGENTS.md`, and `profile/README.md`
- Repo planning handoff: None. This milestone creates the handoff model for later
  milestones.
- Risks / kill assumptions:
  - Fails if the file becomes a hidden implementation backlog instead of an org sequence.
  - Fails if repo owners cannot tell what they own versus what another repo owns.
- Evidence when landed:
  - `.github` PR #7 (this PR) — squash-merged into `main` with `pnpm check` green.
    This milestone bootstraps the sequence, so the same PR that delivers `MILESTONES.md`
    marks M0 done and promotes M1 to current; closure finalizes on merge.

### M1: Jig Contract Design v0

- State: done
- Outcome: Enable Planning and Learning to design against Jig without waiting for Jig
  implementation by sketching the high-level shape of Jig's execution-plan and run-record
  seams — enough to orient downstream design, not a frozen schema.
- Why now: These are the highest-leverage seams in the org. Planning produces execution
  plans, and Learning consumes run records.
- Primary owner: `jig`
- Participating repos: `jig`, `technical-design`, `design-to-plan`, future Learning
  loop, `.github`
- Owned seam or artifact:
  - Execution-plan shape — high-level props (v0)
  - Observability / event record shape — high-level props (v0)
- Entry criteria:
  - Jig product docs describe the five guarantees, tracks, stories, runner/worker boundary,
    and run outcomes.
  - The legacy prototype is treated as reference only, not as authority.
  - M0 is adopted.
- Exit criteria:
  - `jig/docs/design/` names the high-level execution-plan properties, derived from Jig's
    product commitments, at design altitude.
  - `jig/docs/design/` names the high-level run/event record properties at design altitude.
  - The docs are explicit that this is a v0 shape, not a frozen schema: field-level detail is
    expected to firm up in design and mutate somewhat during implementation.
  - The design reconciles to Jig product commitments and explicitly calls out any product
    conflict.
  - Planning and Learning can identify what they consume or produce from the high-level shape
    without reading Jig implementation internals.
- Artifacts:
  - Jig design docs for execution plans
  - Jig design docs for observability / event records
  - Illustrative examples of the high-level plan and record shapes
- Repo planning handoff:
  - `jig` derives a local design plan for the two seams first.
  - `technical-design` reviews only the parts that affect downstream design consumption.
  - Future Planning and Learning work may start discovery but should not publish local
    implementation plans until these high-level shapes are drafted and agreed.
- Risks / kill assumptions:
  - Fails if the plan schema becomes an implementation package layout instead of an input
    contract.
  - Fails if event records optimize for logs rather than durable decisions and outcomes.
  - Fails if product guarantees are weakened to match convenient legacy mechanics.
  - Fails if it tries to freeze a field-level schema now instead of leaving the v0 shape room
    to refine through design and implementation.
- Evidence when landed:
  - `jig` PR #6 merged with the execution-plan and observability contract docs; `main`
    `pnpm check` was green.
  - `ROADMAP.md` seam status links the v0 high-level design docs.

### M2: Technical-Design Handoff Contract

- State: done
- Outcome: Enable the Planning layer to consume technical designs consistently by making the
  design document format explicit as a downstream contract.
- Why now: Planning needs a stable design input shape as much as it needs Jig's output plan
  shape.
- Primary owner: `technical-design`
- Participating repos: `technical-design`, `design-to-plan`, `.github`
- Owned seam or artifact: Technical-design document format
- Entry criteria:
  - The current `technical-design` skills, methodology profile, and evals are green.
  - M0 is adopted.
  - M1 has at least a draft execution-plan shape, or has named what Planning must preserve.
- Exit criteria:
  - `technical-design/docs/design/` names the required fields, sections, IDs, and
    invariants a downstream planner can rely on.
  - Examples distinguish required handoff data from methodology-specific detail.
  - The contract states what future methodology profiles must preserve for Planning.
  - The repo has validation or review guidance that prevents the handoff from passing
    vacuously.
- Artifacts:
  - Technical-design handoff contract
  - Example design-to-planning input fixture
  - Any needed skill/template updates
- Repo planning handoff:
  - `technical-design` derives a local docs/skills plan.
  - `design-to-plan` consumes the contract, not the internal DDD profile mechanics.
- Risks / kill assumptions:
  - Fails if the handoff is too DDD-specific for future methodology profiles.
  - Fails if required planning facts are implied by prose instead of named fields or
    sections.
  - Fails if enforcement guidance can pass without seeded violations or concrete examples.
- Evidence when landed:
  - `technical-design` PR #7 merged with the handoff contract, template, fixture, and
    review guidance; `main` `pnpm check` was green.
  - `ROADMAP.md` links the exact handoff contract.

### M3: PRD and Acceptance-Criteria Contract

- State: done
- Outcome: Enable Product, Design, and Planning layers to cite stable product intent through
  ID'd acceptance criteria instead of relying on unstructured product prose.
- Why now: Planning and technical design both need durable "what and why" references before
  work decomposes into implementation plans.
- Primary owner: `define-product`
- Participating repos: `define-product`, `technical-design`, `design-to-plan`, `.github`
- Owned seam or artifact: PRD / acceptance-criteria-ID format
- Entry criteria:
  - M0 is adopted.
  - Existing prior art from the legacy `define-product` workflow has been curated as
    reference only.
  - `technical-design` has named what product fields it consumes today.
- Exit criteria:
  - The Product layer's product docs define the PRD and acceptance-criteria-ID format.
  - The format distinguishes product outcome, acceptance criteria, constraints,
    assumptions, and non-goals.
  - Technical Design and Planning can cite product IDs without depending on Product layer
    implementation internals.
  - The initial repo or artifact home for the Product layer is chosen deliberately.
- Artifacts:
  - `define-product/docs/product/README.md`
  - `define-product/docs/product/prd-contract.md`
  - `define-product/docs/product/examples/minimal-prd.md`
- Repo planning handoff:
  - `define-product` derives its own product -> design -> implementation plan after M3 lands.
  - `technical-design` and Planning treat the format as a cited input contract.
- Risks / kill assumptions:
  - Fails if acceptance criteria become delivery tasks instead of product-owned outcome
    criteria.
  - Fails if the Product layer duplicates `technical-design` or Planning responsibilities.
  - Fails if ID stability is not specified.
- Evidence when landed:
  - `define-product` PR #1 merged with the PRD contract docs and `pnpm check` green.
  - `.github` PR #11 merged with `ROADMAP.md`, `MILESTONES.md`, and profile links updated to the
    exact Product-layer contract.

### M4: Planning Layer Seed

- State: done
- Outcome: Enable an approved technical design to become a Jig-ready execution plan without
  re-deciding product or design scope.
- Why now: Once Jig's plan shape is drafted and the design handoff is pinned, Planning can be designed
  against contracts rather than against another repo's internals.
- Primary owner: `design-to-plan`
- Participating repos: `design-to-plan`, `jig`, `technical-design`, `define-product`,
  `.github`
- Owned seam or artifact: None new. Planning consumes Product and Technical Design contracts
  and produces Jig execution plans.
- Entry criteria:
  - M1 execution-plan shape v0 is drafted and agreed.
  - M2 technical-design handoff contract is pinned.
  - M3 PRD / acceptance-criteria-ID format is pinned or explicitly deferred with a
    compatibility placeholder.
- Exit criteria:
  - Planning product docs define its role, non-goals, and supported input/output contracts.
  - Planning design docs describe how designs become execution plans.
  - The output fixture is checked against Jig's v0 execution-plan contract shape.
  - The layer refuses to invent product scope or implementation package structure not present
    in its inputs.
- Artifacts:
  - Planning-layer product docs
  - Planning-layer design docs
  - Design-to-plan example fixture
- Repo planning handoff:
  - The `design-to-plan` layer derives its own repo plan from M4.
  - `jig` owns only plan validation feedback, not Planning's decomposition method.
  - `technical-design` owns only design input contract feedback, not execution sequencing.
- Risks / kill assumptions:
  - Fails if Planning reopens product/design decisions instead of translating them.
  - Fails if plan output proves shape but not dependency closure.
  - Fails if producer/consumer relationships are implicit.
- Evidence when landed:
  - `design-to-plan` PR #1 merged with the docs-only Planning-layer seed and `check` green.
  - `.github` PR #12 merged with roadmap, milestone, and profile updates and `check` green.
  - The sample plan fixture is traceable from PRD IDs through technical-design IDs to Jig v0 plan
    properties.

### M5: Jig Local MVP Slice

- State: current
- Outcome: Enable an operator to run one minimal valid execution plan under policy and
  receive durable, inspectable run records.
- Why now: After the contracts exist, a narrow vertical slice can prove the execution model
  without locking the whole product surface too early.
- Primary owner: `jig`
- Participating repos: `jig`, `design-to-plan`, `.github`
- Owned seam or artifact:
  - Local runner behavior behind the execution-plan and run-record contracts
  - Policy and approval behavior for the first supported local mode
- Approach: at M5's start, `jig/docs/design/` named only the two seam contracts
  (execution-plan v0, observability-records v0), so entry criterion 3 below was unmet. M5
  split into a design slice before an implementation slice so that gap closed before code
  committed around it (M5a has since landed and closed it):
  - M5a — a jig-local design slice that names the full local-runtime architecture at high
    altitude: plan validation, preview, eligibility/DAG, the runner/worker authority
    boundary, the authorization/fence, the state machine, the record store, policy, and the
    four drivers (agent, execution-host, forge, work-source). M5a marks each seam with a
    posture so later slices know what they are accountable for now versus later, and it is
    what satisfies entry criterion 3.
  - M5b — implement only the seams M5a marks `exercised`, as a thin walking skeleton, with
    fixtures and tests. Seams marked `named extension point` stay design-only until a later
    slice exercises them.

  | Seam                              | M5 posture            |
  | --------------------------------- | --------------------- |
  | Plan validation and preview       | exercised             |
  | Eligibility / DAG resolution      | exercised             |
  | Runner/worker authority boundary  | exercised             |
  | Authorization / fence             | exercised             |
  | State machine (named run states)  | exercised             |
  | Record store (M1 record shape)    | exercised             |
  | Policy (minimum posture for v0)   | exercised             |
  | Agent driver                      | named extension point |
  | Execution-host driver (non-local) | named extension point |
  | Forge driver                      | named extension point |
  | Work-source driver                | named extension point |
  | Resume                            | named extension point |
  | Capability attestation            | named extension point |

  A seam being `exercised` here means built and tested as part of M5b's dry-run path, not
  that every behavior it could ever have is final. The local execution host and the policy
  posture are themselves exercised seams: M5a must name them concretely enough for M5b to
  build the minimal local case, even though richer hosts and policies stay extension points.

  Posture here is the M5 exit target, not a status claim. M5b re-sequenced its delivery into
  client-usable phases (jig's delivery track), which moved when each seam gets exercised; the
  criterion-to-phase mapping below and jig's track README
  (`jig/docs/delivery/m5b-local-mvp-r2/`) are the binding reconciliation. The post-Phase-2
  repository review (`jig/docs/reviews/2026-07-02-post-phase-2-repo-review.md`) is the
  routed-back finding that prompted this amendment, per Deriving Repo Plans above.

- Entry criteria:
  - M1 is done.
  - M4 has produced a sample execution plan shape fixture (the shape, not a ready-to-parse
    instance — see Artifacts).
  - Jig design has named the first local execution host and the minimum policy posture.
    Met: M5a landed (`jig/docs/design/` plus its archived M5a runtime-design note).
- Exit criteria:
  - Jig validates and previews one minimal, machine-readable execution-plan instance.
  - Jig executes that plan as a dry-run under policy: no privileged action (push, PR
    creation, merge) fires.
  - Even in the dry-run, the run invokes the authorization/fence and emits the
    requested -> authorized/denied -> runner-owned records. The authority boundary is
    exercised, not bypassed — this is the line between a meaningful slice and a hollow one.
  - Jig emits durable records matching the M1 observability record shape.
  - The run ends in named, inspectable states.
  - Tests appropriate to the exercised surface. This is jig's first real package: `pnpm check`
    grows from prettier-only to include lint, typecheck, and test, with TDD coverage at 90%+.
- Exit-criteria delivery status (amended 2026-07-02, from jig's delivery track r2):

  | Exit criterion                                          | Status                                                               |
  | ------------------------------------------------------- | -------------------------------------------------------------------- |
  | Validates and previews a plan                           | Validation delivered; preview lands in jig Phase 3                   |
  | Dry-run executes without privileged action              | Delivered (M5b Phases 1-2)                                           |
  | Fence emits requested -> authorized/denied/runner-owned | Lands in jig Phase 3 (boolean gate recorded as ADR 0018 scaffolding) |
  | Records match the M1 shape                              | Partial; convergence lands in jig Phase R (ADR 0017)                 |
  | Named, inspectable states                               | Delivered (M5b Phase 2)                                              |
  | `check` grows to lint+typecheck+test, 90%+ coverage     | Delivered (TypeScript engine-archetype migration)                    |

  M5 is not exit-complete until jig's Phase R and Phase 3 land; the phases and their
  ID-bearing acceptance criteria live in `jig/docs/delivery/m5b-local-mvp-r2/phases.md`.

- Artifacts:
  - Minimal CLI or runnable entry point
  - Plan fixture — the machine-readable execution-plan instance is M5's own artifact; M4
    handed off only the plan shape, not an instance ready to parse.
  - Policy fixture
  - Run-record fixture
  - Tests and verification docs
- Repo planning handoff:
  - `jig` derives implementation stories from its design docs, including the M5a/M5b split
    above.
  - `design-to-plan` provides only the sample plan fixture shape until broader integration is
    planned.
- Risks / kill assumptions:
  - Fails if the MVP bypasses the runner/worker authority boundary for convenience.
  - Fails if records are useful only for debugging and not for Learning consumers.
  - Fails if "minimal" expands into multi-driver portability before the local path proves out.
  - Fails if "design fully, fill later" produces unexercised no-op code stubs for seams the
    dry-run never traverses (resume, capability attestation, multi-driver, forge); those stay
    named extension points in the design doc only, never code, until a later slice exercises
    them. The authorization no-op is different: it is on the executed dry-run path and emits
    its records.
- Evidence when landed:
  - Jig PR merged with tests and `pnpm check` green.
  - The sample run record can be cited by Learning-loop design.

### M6: Learning Loop Seed

- State: proposed
- Outcome: Enable owners to turn Jig run records into source-of-truth hardening
  recommendations for the earliest layer that should have prevented a defect.
- Why now: Learning should consume real run records, but its product/design can be shaped as
  soon as Jig's record contract and sample records exist.
- Primary owner: future Learning loop
- Participating repos: future Learning loop, `jig`, `technical-design`, `define-product`,
  `design-to-plan`, `.github`
- Owned seam or artifact: None new. Learning consumes Jig observability / event records.
- Entry criteria:
  - M1 event-record shape v0 is drafted and agreed.
  - M5 has emitted at least one representative sample record, or M1 examples are sufficient
    for product/design seeding.
- Exit criteria:
  - Learning-loop product docs define between-run scope and per-run non-goals.
  - Design docs map record inputs to recommendation outputs.
  - The loop distinguishes source-of-truth fixes from local workaround suggestions.
  - The first examples show feedback routed to Product, Technical Design, Planning, or Jig.
- Artifacts:
  - Learning-loop product docs
  - Learning-loop design docs
  - Sample retro/recommendation fixture
- Repo planning handoff:
  - The Learning-loop repo derives its own product -> design -> implementation plan.
  - Jig remains responsible for record contract compatibility.
- Risks / kill assumptions:
  - Fails if Learning becomes part of Jig's hot path instead of a between-runs tool.
  - Fails if recommendations are generic retros rather than traceable hardening proposals.
  - Fails if recurring defects are not promoted into earlier mechanical checks.
- Evidence when landed:
  - Learning-loop PR or repo creation merged with checks green.
  - At least one sample recommendation cites the Jig record fields it used.

---

## Deferred From This Sequence

These remain outside the current milestone sequence until a future roadmap revision pulls
them in:

- Hosted multi-tenant operation
- Remote execution hosts beyond a ready seam
- Webhook or scheduler-triggered runs
- Model-adjudicated approval autonomy
- Legacy run migration
