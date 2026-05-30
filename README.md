# NECL Brownout v1.2 — Scenario Design

**[Live demo →](https://data2usa.github.io/demo_brownout/)**

An interactive scenario design artifact showing how a mesh of edge-deployed **DataCubes** detect a grid disturbance, negotiate a dispatch response, and produce a human-approved execution plan — all modelled as a live property graph.

This is one scenario in the **EdgeWorks** "load-bearing" design library. It is intended for demonstration, review, and graph import — not production deployment.

---

## The scenario

A data center throws its main switch. An unannounced +180 MW load hits the Northeast Corridor grid. Five substation DataCubes feel it. The scenario walks through eight beats:

| Beat | What happens |
|------|-------------|
| Idle mesh | Five DataCubes monitoring a stable grid, all inter-cube links nominal |
| Disturbance | Sudden load event detected near Riverbend 138 kV |
| Observations | Affected cubes emit timestamped sensor readings (MW, voltage, frequency) |
| Disturbance zone | Cubes reporting deviations are grouped into a DisturbanceZone |
| Negotiation | A NegotiationRound records each cube's proposed dispatch action |
| Consensus | A ConsensusProposal is derived, justified by the supporting observations |
| Execution plan | The proposal is rendered into a human-reviewable ExecutionPlan |
| Approval | A certified grid dispatcher signs off; the full audit chain is complete |

---

## Files

```
index.html                  Interactive scenario walkthrough (open in browser, or visit GitHub Pages)
review_import/
  workflow.json             Graph import spec (Memgraph / Neo4j IngestStructured)
  entities/                 10 CSV files — one per node type
  relationships/            12 CSV files — one per edge type
```

---

## Running the explorer

Open `index.html` directly in a browser, or visit the live GitHub Pages link above. No server, no install, no build step required. The file is entirely self-contained (React 18, D3 v7, and Babel are loaded from CDN).

**Live links by theme:**

| Theme | URL |
|-------|-----|
| Dark (default) | [demo_brownout/?mode=dark](https://data2usa.github.io/demo_brownout/?mode=dark) |
| Light | [demo_brownout/?mode=light](https://data2usa.github.io/demo_brownout/?mode=light) |

Locally, append `?mode=dark` or `?mode=light` to the file URL, e.g. `file:///path/to/index.html?mode=light`.

Use the **Next / Prev** buttons to step through beats. Click any node to inspect its properties. Toggle **Audit chain** on the final beat to highlight the full evidence path from raw observation to approved execution plan.

---

## Graph data model

### Node types (10)

| Label | ID field | Description |
|-------|----------|-------------|
| DataCube | `cube_id` | Ruggedized edge node at a substation; holds a local graph logbook |
| Disturbance | `disturbance_id` | A detected grid perturbation |
| Observation | `observation_id` | Single timestamped sensor reading with provenance |
| DisturbanceZone | `zone_id` | Dynamic region of cubes affected by a Disturbance |
| NegotiationRound | `round_id` | One round of cube-to-cube coordination |
| Contribution | `contribution_id` | One cube's proposed dispatch action within a round |
| ConsensusProposal | `proposal_id` | Agreed dispatch reconfiguration signed off by all participants |
| ExecutionPlan | `plan_id` | Human-reviewable dispatch order document |
| Approval | `approval_id` | Human operator sign-off; carries its own lineage in the graph |
| Operator | `operator_id` | Certified grid dispatcher on shift |

All nodes carry `scenario: "NECL_Brownout_v1.2"` and `node_kernel: "load-bearing"` as static properties.

### Relationship types (12)

`INTER_CUBE_LINK` · `OBSERVED_BY` · `CONTRIBUTES_TO` · `INCLUDES` · `OCCURS_IN` · `CONTRIBUTED` · `IN_ROUND` · `DERIVED_FROM` · `JUSTIFIED_BY` · `RENDERS` · `GRANTED` · `APPROVES`

### Audit chain (end-to-end traversal)

```
Observation → CONTRIBUTES_TO → Disturbance → OCCURS_IN → DisturbanceZone
  → INCLUDES → DataCube → CONTRIBUTED → Contribution → IN_ROUND → NegotiationRound
  → DERIVED_FROM ← ConsensusProposal → RENDERS ← ExecutionPlan
  → APPROVES ← Approval ← GRANTED ← Operator
```

A single Cypher traversal can replay the full decision chain years after the event.

---

## Loading into a graph database

`review_import/workflow.json` defines a Memgraph/Neo4j-compatible `IngestStructured` import. The spec includes:

- Unique constraints on all node ID fields
- Default property indexes on key fields
- Two fulltext indexes (English analyzer, eventually consistent)

The CSV paths in the spec use `data/projects/necl_brownout/` as the root. Adjust to match your deployment layout, or copy `review_import/entities/` and `review_import/relationships/` to that path before importing.

---

## Repository

[https://github.com/data2usa/demo_brownout](https://github.com/data2usa/demo_brownout)
