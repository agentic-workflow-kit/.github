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

### M7: Real Provider Integration

- State: done
- Outcome: Enable an operator to turn an approved plan into real landed work — a real agent
  driver doing real edits, a real execution host with proven confinement, real Forge/GitHub
  landing, and real work-source intake — recorded in durable, inspectable records under
  policy. This promotes Jig's M5 `named extension point` seams (agent driver, execution-host
  driver, forge driver, work-source driver, resume, capability attestation) to `exercised`
  behind the same contracts, with real effects replacing the reference adapters.
- Why now: The M5 slice pinned the four provider ports, the composition root, the
  capability-attestation gate, and the driver conformance suite as exercised, Jig-internal
  seams — proven with reference adapters that perform no real behavior. With those contracts
  stable and merged (Jig Phase 5, commit `f59a479`), real drivers can slot behind them without
  churning the seams. Sequence is not serialization: M5 remains `current`, but M7 consumes the
  pinned, merged P5 seams, so its repo planning can proceed now.
- Primary owner: `jig`
- Participating repos: `jig`, `.github`. Learning consumes the resulting real run records.
- Owned seam or artifact: None new. M7 exercises the execution-plan and observability-records
  contracts already owned by M1 and Jig, driving them through real providers rather than
  reference adapters. It introduces no new org-level seam.
- Entry criteria:
  - Jig Phase 5 — provider ports, composition root, capability attestation, and the driver
    conformance suite — is merged to Jig `main` (commit `f59a479`).
  - The outcomes are already committed by Jig's product as ID-bearing guarantees
    (STACK / DRIVE / SEC / MERGE / ISO / RESUME / EARN), so no new PRD or lifecycle layer is
    stood up. This is not a new-layer milestone.
- Exit criteria:
  - Real drivers selected through Jig's composition root perform real effects — real agent
    edits, confined real execution, real Forge/GitHub landing, real work-source intake — under
    policy and against a real capability attestation, not a reference adapter.
  - Those real effects are recorded in durable, inspectable records matching the M1
    observability record shape, including tamper-evidence over the record chain and an active
    re-approval path when an approved plan's basis changes.
  - Isolation is proven per story in parallel workspaces, and freshness is decided by a real
    clock rather than a stubbed constant; secrets are scanned and redacted on the real
    landing path.
  - The driver conformance suite still fails closed on a broken or non-conforming adapter.
  - Jig's Phase-0..4 record goldens stay byte-identical under the default (reference) wiring,
    so real drivers are opt-in and do not silently alter the proven local path.
- Artifacts:
  - Real driver implementations behind the existing agent, execution-host, forge, and
    work-source ports.
  - Real capability-attestation records and the confinement evidence they attest to.
  - Driver conformance-suite results, including the fail-closed case on a broken adapter.
  - Tamper-evidence and secret-redaction evidence over the real landing path.
- Closeout checkpoint:
  - Jig's local M7 real-provider spine through Phases 6-9 is merged through Jig PR #39 at
    commit `3b3d224`.
  - The seam-level driver path and records-integrity path are complete for the M7 repo
    track: real agent/host, Forge, work-source, and integrity behavior are exercised behind
    Jig's existing ports and recorded under policy.
  - Remaining tail items are not automatic continuation work. They require replanning before
    implementation because they carry product-surface, evidence, policy, or package-boundary
    decisions outside the Phases 6-9 spine.
- Post-spine decisions (2026-07-04): the owner's sitting after the closeout checkpoint above.
  These append to M7; they do not rewrite the closeout text.
  1. **D-EXIT (M7 exit evidence) — Option A.** A committed, redacted, citable real-run evidence
     record ("EVRUN") is mandatory for M7 exit either way. The owner accepts that its agent leg
     may be a scripted injected session with an explicit Limitations section
     ("EVRUN-partial"), with the Codex-driven leg ("EVRUN-full") recorded as a **named debt**
     retired by the post-M7 Codex-transport track (N1). M7's `State` above stays `current` and
     flips to `done` only when the EVRUN-partial record is merged — per this file's own rule
     that a milestone is done only when exit criteria are evidenced. Concrete Codex transport
     evidence stays a post-M7 validation adjunct gating any v0 contract freeze, not org-level
     exit. Lifecycle-edge split: no codex-plugin-cc-style lifecycle edges (broker, interrupt,
     cleanup, Windows process-tree) gate M7 exit; all of them gate the transport ADR and,
     through it, the freeze.
     - **Exit evidence — met 2026-07-04.** EVRUN-partial merged as jig PR #45 (squash
       `707d2114f`): a real work-source → Forge → records-integrity run against the private
       `jig-smoke-target` sandbox with a scripted agent leg (real GitHub Issues intake, real
       `open-pr` landing, HMAC integrity sidecar verified), committed as the citable record
       `docs/design/evidence/2026-07-04-evrun-partial-smoke.md` under jig's evidence convention.
       This satisfies the D-EXIT Option A condition, so M7's `State` is flipped to `done` above.
       EVRUN-full (the Codex-driven agent leg) remains the named debt retired by the post-M7
       Codex-transport track (N1); it is not a reopening of M7 exit.
  2. **Packaging (N2) — internal SDK boundary now, no publishing.** Rationale: extensibility and
     single-responsibility; the SDK's first-party consumers are jig's CLI and a future MCP
     surface. Packages stay `private: true`; the posture can flip to public later, and no
     stability promise is created now. The third-party (out-of-repo, installable)
     provider-ecosystem question explicitly **remains open** as a deferred product question —
     the packaging design must not silently assume it either way. Consequence: jig's
     packaging/SDK-boundary design cycle (N3) is authorized to start once this decision is
     recorded in jig's product docs.
  3. **TUI/dashboard — extension-point-only** (per jig guarantee CFG-7). No first-party TUI
     cycle; reopen only on recorded operator pain. Third-party surfaces are enabled through the
     observability-records contract plus the internal SDK boundary (decision 2).
  4. **Smoke target (operational) — hybrid.** A local bare git remote serves repeatable
     push-path smoke tests (no credentials); a dedicated private sandbox repo
     (`jig-smoke-target`) with a repo-scoped fine-grained PAT (env-only, never in records)
     serves the full landing evidence (push + PR + merge + block-surfacing); an independent
     secret-scan of any evidence record is required before it is committed.
  5. **Merge policy for the wave.** The owner authorized autonomous squash-merge for wave-1 PRs
     once review threads are resolved and `pnpm check` is green, except this sequencing PR
     itself (`.github`, "docs: record post-M7 sequencing decisions and authorize wave 1"),
     which is human-reviewed and human-merged.
  6. **Wave-1 authorization.** The following may start once this PR records the decisions above:
     - jig's absorption-ledger absorbed-vs-open update;
     - jig's Codex-transport evidence-capture plan, plus a dated/hashed `evidence/` convention;
     - jig's product amendment recording decision 2 (packaging/SDK boundary);
     - a design-flow item for self-report-only/mock-adequacy conformance vocabulary;
     - the EVRUN-partial evidence task, once the smoke target (decision 4) exists;
     - M8 below, as a **proposal for owner ratification** in this PR's review — it is not
       self-authorizing.

     M6 (Learning Loop Seed) stays `proposed` and is not part of this wave-1 list; pull timing
     remains an open org choice.

  7. **Repo-plan open-question disposition** — `jig/docs/delivery/m7-real-providers/repo-plan-m7.md`
     "Open questions to send back to the org roadmap" #1-#3, each verified against the jig ADRs
     at the time of this sitting:
     - **#1 — sync `describe()` vs async proof: settled by ADR 0022.**
       [`jig/docs/design/decisions/0022-phase-6-real-driver-integration.md`](https://github.com/agentic-workflow-kit/jig/blob/main/docs/design/decisions/0022-phase-6-real-driver-integration.md),
       Decision 3, "`describe()` stays synchronous — prove-then-describe resolves the
       sync/async tension": the confinement proof runs async at compose time, outside
       `describe()`, in the host driver's factory; `describe()` stays a pure getter over an
       already-computed attestation. No P5-pinned seam surface flexes. Confirm this holds at
       the transport-ADR session.
     - **#2 — `action`-union / provenance record impact: settled by ADR 0023 and ADR 0024.**
       [`jig/docs/design/decisions/0023-phase-7-real-forge-landing.md`](https://github.com/agentic-workflow-kit/jig/blob/main/docs/design/decisions/0023-phase-7-real-forge-landing.md),
       Decision 2, repairs `LandingRequest.action` from the single mis-encoded literal
       `'push|open-pr|merge'` to the real union `'push' | 'open-pr' | 'merge'` — a local
       port-type fix that freezes nothing; the dry-run modeled-landing record keeps recording
       the literal token verbatim, so the byte-identity goldens are untouched.
       [`jig/docs/design/decisions/0024-phase-8-real-work-source.md`](https://github.com/agentic-workflow-kit/jig/blob/main/docs/design/decisions/0024-phase-8-real-work-source.md),
       Decision 3, widens `CandidateWorkItem.provenance` from the single literal
       `'jig-validated'` to an origin-bearing shape (source system + candidate identifier),
       legible in the run record as an additive field alongside `run.drivers.workSource` — also
       a local fix, no records-contract freeze required. Both ADRs confirm no
       observability-records v0 field change was needed.
     - **#3 — tamper-evidence contract field: resolved for now by ADR 0025.**
       [`jig/docs/design/decisions/0025-phase-9-records-integrity.md`](https://github.com/agentic-workflow-kit/jig/blob/main/docs/design/decisions/0025-phase-9-records-integrity.md),
       Decision 1 plus Contract Impact Gate Q5: tamper-evidence is computed over existing
       durable evidence but materialized on a separate, non-golden integrity sidecar
       (`runs/<id>/integrity.json`, name design-owned) with an environment-keyed HMAC; no
       digest/HMAC field is added to the observability-records v0 contract. The ADR states
       this explicitly answers open question 3 and that the freeze posture is re-examined at
       the v0-freeze checkpoint (T14), not settled here.
  8. **Contract-owner role named.** The contract owner for the two org seams jig owns
     (execution-plan, observability-records) — the role that receives routed seam-shape
     questions (for example, the transport session-observability question and any
     port-surface flex) and that alone executes any v0 contract freeze at the T14
     checkpoint — is jig's design authority: the org/product owner deciding through jig's
     ADR flow (`docs/design/decisions/`), per M1's seam ownership. Delivery tracks and
     sub-agents do not decide seam shapes (the repo-plan "must not decide" boundary).
- Repo planning handoff:
  - `jig` derives its repo plan directly from this milestone, decomposing the outcome into its
    own phases and stories. Org milestones carry no story list; that decomposition lives in
    Jig's delivery track.
  - The real run records M7 emits strengthen M6 seeding, but M6 does not block on M7 — M6's
    entry is already satisfiable by M1 examples or an M5 record.
  - Concrete Codex transport evidence or an ADR is a post-M7 validation adjunct unless the
    org explicitly decides that shipped transport evidence is required for M7 exit. Decided
    2026-07-04 (Post-spine decision 1): it remains a post-M7 adjunct gating any v0 freeze,
    not org-level exit; M7 exit evidence is the committed EVRUN-partial record.
  - A first-party TUI or dashboard is extension-point-only (Post-spine decision 3,
    2026-07-04): no first-party TUI cycle; reopen only on recorded operator pain.
    Third-party surfaces build on the observability-records contract and the internal SDK
    boundary.
  - A policy analyzer should wait for enough real run-history data to avoid designing against
    synthetic or one-off examples.
  - v0 contract freeze or package extraction is gated by Phase 9 records-integrity evidence
    from Jig PR #39 plus package-boundary and product decisions; if the org requires
    concrete transport evidence for M7 exit, that evidence should also precede any freeze.
    The package-boundary/product decision is recorded (Post-spine decision 2); the
    transport-evidence conditional is resolved by decision 1.
- Risks / kill assumptions:
  - Fails if a real driver can escalate its substrate — argv, credentials, or egress — past
    what the attestation authorized.
  - Fails if attested isolation strength is declared rather than proven by an exercised check.
  - Fails if real landing is not idempotent when re-run against a real effect.
  - Fails if "real" expands into hosted, multi-tenant, or remote operation before the local
    real path proves out. The items under `## Deferred From This Sequence` stay out of M7.
- Evidence when landed:
  - Jig PR(s) merged with real drivers exercised end to end and `pnpm check` green.
  - A real run record — with attestation, tamper-evidence, and redacted secrets — can be cited
    by Learning-loop design.

### M8: Planning-discipline uplift

- State: proposed — recorded here as a proposal for owner ratification in the PR that adds
  this entry; it does not self-authorize on merge of that PR alone.
- Outcome: Enable the Planning layer (and the layers that feed it) to reuse the durable
  planning-discipline ideas the prev-gen authoring standard proved out, de-productized from
  that legacy corpus rather than ported wholesale, so future planning cycles do not
  re-discover the same lessons from scratch.
- Why now: Raised at the 2026-07-04 post-M7 sequencing sitting alongside the M7 exit,
  packaging, TUI, smoke-target, and merge-policy decisions, as the owner's forward-looking
  planning-quality item for the wave after the M7 spine closed.
- Primary owner: `design-to-plan`
- Participating repos: `design-to-plan`, `technical-design`, `jig`, `.github`
- Owned seam or artifact: Not yet pinned. This entry proposes the milestone; the seam or
  artifact it owns (if any) is decided when the milestone is planned, not here.
- Entry criteria:
  - The owner ratifies this milestone (this entry is a proposal pending that ratification).
  - M7's post-spine decisions above are recorded (this PR).
- Exit criteria:
  - Not yet defined. Per this file's Planning Rules, exit criteria are set when the milestone
    is planned in detail, not invented ahead of that work; this proposal intentionally leaves
    them open rather than guessing at scope the owner has not yet specified.
- Artifacts: Not yet defined; derived when the milestone is planned.
- Repo planning handoff:
  - `design-to-plan` derives its own repo plan from M8 once ratified and planned.
  - `technical-design` and `jig` participate where the de-productized ideas touch their
    layers, but do not decide M8's scope unilaterally.
  - Carve-out: jig's `PlanValidator` plan-ingest preflights (the Phase 8 structural intake
    chokepoint and related plan-validation feedback) may proceed jig-locally under M4's
    existing plan-validation-feedback ownership without waiting for M8. M8 does not gate
    work M4 already owns.
- Risks / kill assumptions:
  - Fails if it re-imports the legacy authoring standard wholesale instead of de-productizing
    only the durable ideas.
  - Fails if it duplicates or contests M4's existing plan-validation-feedback ownership rather
    than sitting alongside it.
  - Fails if "planning discipline" expands into a general planning-layer rewrite instead of a
    bounded uplift.
- Evidence when landed: Not yet defined; recorded when the milestone is planned and delivered.

---

## Deferred From This Sequence

These remain outside the current milestone sequence until a future roadmap revision pulls
them in:

- Hosted multi-tenant operation
- Remote execution hosts beyond a ready seam
- Webhook or scheduler-triggered runs
- Model-adjudicated approval autonomy
- Legacy run migration
