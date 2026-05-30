# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A self-contained design artifact for the **NECL Brownout v1.2** scenario — one of the EdgeWorks "load-bearing" scenario designs. It demonstrates how a mesh of DataCube field nodes collectively detect a grid disturbance, negotiate a dispatch response, and produce a human-approved execution plan, all represented as a property graph.

There is no build system, no package manager, and no test runner. The deliverables are:

- `necl-explorer.html` — a single-file interactive scenario walkthrough
- `review_import/` — CSV data and a `workflow.json` import spec for loading the scenario into a graph database (Memgraph/Neo4j)

## Running the explorer

Open `necl-explorer.html` directly in a browser — no server required. It uses CDN-loaded React 18, D3 v7, and Babel standalone. The file is entirely self-contained.

## Architecture of necl-explorer.html

The file is a single-page React app written in JSX inside a `<script type="text/babel">` block. Key sections in order:

- **`THEMES`** — dark/light color palettes keyed by node type
- **`PLACES` / `LANE_POS`** — fixed SVG coordinates for the geographic map (left) and decision lane (right)
- **`BEATS`** — ordered array of scenario steps. Each beat specifies `addNodes`, `addEdges`, and optionally `auditChain` (an array of node/edge IDs that form the highlighted audit path in the final beat)
- **`KIND_META`** — per-node-type descriptions and property generators used by the inspector panel
- **`NECLExplorer`** — the main React component; manages beat state, d3-force simulation, zoom/pan, drag, and SVG rendering

The visualization has two spatial regions:
- **Left (~0–880 px)**: geographic map with fixed substation positions (`PLACES`). Nodes with `placeId` are pinned here; the d3 force simulation respects these fixed positions.
- **Right (~950–1400 px)**: "decision lane" for consensus/negotiation nodes. Nodes with `lanePos` are pinned to `LANE_POS` coordinates.

The d3-force simulation runs continuously but nodes with `fx`/`fy` set are frozen in place. New nodes added by a beat get an entry animation (scale + opacity). The audit chain beat highlights a specific traversal path across both regions.

## Graph data model (review_import/)

### Entities (10 node types)
All nodes carry `scenario: "NECL_Brownout_v1.2"` and `node_kernel: "load-bearing"` as static properties.

| Label | ID field | Role |
|---|---|---|
| DataCube | cube_id | Grid edge node at a substation |
| Disturbance | disturbance_id | A detected grid perturbation |
| Observation | observation_id | Single timestamped sensor reading |
| DisturbanceZone | zone_id | Set of cubes affected by a disturbance |
| NegotiationRound | round_id | One round of cube-to-cube coordination |
| Contribution | contribution_id | One cube's proposed dispatch action |
| ConsensusProposal | proposal_id | Agreed dispatch reconfiguration |
| ExecutionPlan | plan_id | Human-reviewable dispatch order document |
| Approval | approval_id | Human operator sign-off |
| Operator | operator_id | Certified grid dispatcher |

### Relationships (12 edge types)
`INTER_CUBE_LINK`, `OBSERVED_BY`, `CONTRIBUTES_TO`, `INCLUDES`, `OCCURS_IN`, `CONTRIBUTED`, `IN_ROUND`, `DERIVED_FROM`, `JUSTIFIED_BY`, `RENDERS`, `GRANTED`, `APPROVES`

### workflow.json
Defines an `IngestStructured` step for importing all CSVs into a Memgraph/Neo4j-compatible graph. File paths in the spec use `data/projects/necl_brownout/` as the root — the `review_import/` folder in this repo is the source; adjust paths when deploying. Includes unique constraints, default indexes, and two fulltext indexes (English analyzer, eventually consistent).

## Scenario narrative (the eight beats)

1. **Idle mesh** — five DataCubes monitoring a stable grid, linked by INTER_CUBE_LINKs
2. **Disturbance** — a data center throws its main switch; a Disturbance node enters
3. **Observations** — affected cubes emit Observation nodes linked to the Disturbance
4. **Disturbance zone** — a DisturbanceZone groups the affected cubes
5. **Negotiation** — a NegotiationRound records each cube's Contribution
6. **Consensus** — a ConsensusProposal is derived from the round, justified by Observations
7. **Execution plan** — an ExecutionPlan renders the proposal for human review
8. **Approval** — a human Operator grants an Approval; the audit chain is complete
