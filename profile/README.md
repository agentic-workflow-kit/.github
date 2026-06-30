# agentic-workflow-kit

> A polyrepo family for an agentic software-development lifecycle: product intent -> technical
> design -> delivery -> learning.

`agentic-workflow-kit` is the public umbrella for standalone, composable repositories. Each repo is
independently useful, but together they form a lifecycle for turning software intent into designed,
implemented, reviewed, and improved systems.

This organization carries the methodology and product layers. The workflow engine/tooling itself is
developed separately.

## Why Polyrepo

Each layer has its own maturity curve, release cadence, and audience. Keeping the layers as separate
repositories makes adoption easier: use the design layer without adopting the product layer, or use
the productized experience without pulling in internal delivery tooling.

The trade-off is convention drift. This org keeps shared conventions explicit and lightweight so each
repo can stay focused without inventing its own vocabulary.

## Repositories

| Repo | Role | Status |
| --- | --- | --- |
| [`technical-design`](https://github.com/agentic-workflow-kit/technical-design) | Design layer: frame, author, review-loop, enforce, and orchestrate technical designs. | Ready locally; GitHub shell prepared |
| [`jig`](https://github.com/agentic-workflow-kit/jig) | Productized face of the suite: main package, supporting products, and learning loop. | Early / planned |
| Product layer | Define-product and PRD authoring upstream of design. | Planned |
| Delivery layer | Decompose design into tracker-backed implementation. | Planned |
| Learning loop | Capture run outcomes and feed them back into future work. | Planned |

## Lifecycle

```text
PRODUCT ---------> DESIGN ----------> DELIVERY --------> LEARNING
define / PRD       technical-design   plan / implement   feedback loop
                   repo               on the engine      back into layers
```

The repos are meant to compose without becoming tightly coupled. Each should have a crisp purpose,
clear inputs and outputs, and enough documentation to be useful in isolation.

## Shared Conventions

The shared vocabulary is intentionally small:

- skills use verb-noun names, such as `frame-technical-design` and `plan-delivery-track`;
- skill packs follow the Agent Skills `SKILL.md` shape;
- durable artifacts should be explicit, reviewable, and easy to carry between agents;
- cross-repo snapshots are references, not sources of truth.

## Current Focus

The first public layer is `technical-design`: a set of AI skills for right-sized architecture,
reviewable decisions, and enforceable boundaries. `jig` is the planned product surface that will make
the broader suite approachable as a cohesive experience.

Start with [`technical-design`](https://github.com/agentic-workflow-kit/technical-design) for the
design-stage skills, or follow [`jig`](https://github.com/agentic-workflow-kit/jig) for the product
surface as it takes shape.
