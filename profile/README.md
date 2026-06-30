# agentic-workflow-kit

> A polyrepo family for an agentic software-development lifecycle: product intent -> technical
> design -> delivery -> learning.

`agentic-workflow-kit` is the public umbrella for standalone, composable repositories. Each repo is
independently useful, but together they form a lifecycle for turning software intent into designed,
implemented, reviewed, and improved systems.

This organization carries the full lifecycle as standalone products — from product definition through
technical design to delivery. **Jig**, the delivery/execution engine, is the suite's main product and
lives here.

## Why Polyrepo

Each layer has its own maturity curve, release cadence, and audience. Keeping the layers as separate
repositories makes adoption easier: use the design layer on its own, or run the delivery engine
without adopting the upstream layers.

The trade-off is convention drift. This org keeps shared conventions explicit and lightweight so each
repo can stay focused without inventing its own vocabulary.

## Repositories

| Repo                                                                           | Role                                                                                                                                                              | Status                                         |
| ------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------- |
| [`technical-design`](https://github.com/agentic-workflow-kit/technical-design) | Design layer: frame, author, review-loop, enforce, and orchestrate technical designs.                                                                             | Ready locally; GitHub shell prepared           |
| [`jig`](https://github.com/agentic-workflow-kit/jig)                           | Delivery / execution engine: runs an approved plan under policy into reviewed, landed work, or a deliberate stop. The tool you run (`@agentic-workflow-kit/jig`). | Early; product layer drafted, design/impl next |
| Product layer                                                                  | Define-product and PRD authoring upstream of design.                                                                                                              | Planned                                        |
| Planning layer                                                                 | Decompose design into the execution plan Jig runs.                                                                                                                | Planned                                        |
| Learning loop                                                                  | Capture run outcomes and feed them back into future work.                                                                                                         | Planned                                        |

## Lifecycle

```text
PRODUCT ---------> DESIGN ----------> DELIVERY --------> LEARNING
define / PRD       technical-design   plan -> jig (run)  feedback loop
                   repo               this org           back into layers
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

The first design-stage layer is `technical-design`: a set of AI skills for right-sized architecture,
reviewable decisions, and enforceable boundaries. `jig`, the suite's execution engine, now has its
product layer drafted; its engineering design and implementation are being built next.

Start with [`technical-design`](https://github.com/agentic-workflow-kit/technical-design) for the
design-stage skills, or follow [`jig`](https://github.com/agentic-workflow-kit/jig) — the execution
engine — as it takes shape.
