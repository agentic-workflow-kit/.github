# agentic-workflow-kit

> A polyrepo family for an agentic software-development lifecycle: product intent -> technical
> design -> planning -> delivery -> learning.

`agentic-workflow-kit` is the public umbrella for standalone, composable repositories. Each repo is
independently useful, but together they form a lifecycle for turning software intent into designed,
implemented, reviewed, and improved systems.

This organization carries the full lifecycle as standalone products, from product definition through
technical design, planning, delivery, and learning. **Jig**, the delivery/execution engine, is the
suite's main runtime product and lives here.

## Why Polyrepo

Each layer has its own maturity curve, release cadence, and audience. Keeping the layers as separate
repositories makes adoption easier: use the design layer on its own, or run the delivery engine
without adopting the upstream layers.

The trade-off is convention drift. This org keeps shared conventions explicit and lightweight so each
repo can stay focused without inventing its own vocabulary.

## Repositories

| Repo                                                                           | Role                                                                                                                                                              | Status                                                                                      |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------- |
| [`define-product`](https://github.com/agentic-workflow-kit/define-product)     | Product layer: PRD authoring and stable, ID'd acceptance criteria upstream of design.                                                                             | Seeded; M3 contract bootstrap                                                               |
| [`technical-design`](https://github.com/agentic-workflow-kit/technical-design) | Design layer: frame, author, review-loop, enforce, and orchestrate technical designs.                                                                             | Built; planning handoff contract pinned                                                     |
| [`jig`](https://github.com/agentic-workflow-kit/jig)                           | Delivery / execution engine: runs an approved plan under policy into reviewed, landed work, or a deliberate stop. The tool you run (`@agentic-workflow-kit/jig`). | Early; local dry-run walking skeleton (M5b Phases 0-2); M7 real-provider integration queued |
| [`design-to-plan`](https://github.com/agentic-workflow-kit/design-to-plan)     | Planning layer: decompose approved designs into the execution-plan shape Jig runs.                                                                                | Seeded; M4 contract bootstrap                                                               |
| Learning loop                                                                  | Capture run outcomes and feed them back into future work.                                                                                                         | Planned                                                                                     |

## Lifecycle

```text
PRODUCT ---------> DESIGN ----------> PLANNING --------> DELIVERY --------> LEARNING
define / PRD       technical-design   design-to-plan     jig (run)          feedback loop
```

The repos are meant to compose without becoming tightly coupled. Each should have a crisp purpose,
clear inputs and outputs, and enough documentation to be useful in isolation.

The org roadmap is sequence-based rather than calendar-based: shared milestones define the next
outcome and seam to stabilize, then each repo derives its own local plan from that milestone.

## Shared Conventions

The shared vocabulary is intentionally small:

- skills use verb-noun names, such as `frame-technical-design` and `plan-delivery-track`;
- skill packs follow the Agent Skills `SKILL.md` shape;
- durable artifacts should be explicit, reviewable, and easy to carry between agents;
- cross-repo snapshots are references, not sources of truth.

## Current Focus

The current org focus is M5: proving a narrow Jig local MVP slice now that Product, Technical
Design, and Planning contract shapes are seeded. M4 seeded `design-to-plan` so approved technical
designs can become Jig-ready execution plans without re-deciding product or design scope.

With Jig's Phase 5 provider ports and capability-attestation gate now merged, M7 (Real Provider
Integration) is ready for repo planning: Jig promotes its agent, execution-host, forge, and
work-source seams from reference adapters to real drivers behind the same contracts.

Start with [`define-product`](https://github.com/agentic-workflow-kit/define-product) for product
intent and acceptance-criteria IDs, [`technical-design`](https://github.com/agentic-workflow-kit/technical-design)
for design-stage skills, [`design-to-plan`](https://github.com/agentic-workflow-kit/design-to-plan)
for Planning-layer docs, or [`jig`](https://github.com/agentic-workflow-kit/jig) for the execution
engine.
