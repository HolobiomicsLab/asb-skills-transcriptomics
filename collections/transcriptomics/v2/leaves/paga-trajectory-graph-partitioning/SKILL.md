---
name: paga-trajectory-graph-partitioning
description: Use when reprocessing a published scRNA-seq count matrix (e.g., 10x Chromium
  data processed with Cell Ranger v6.0.1) and the goal is to partition cells into
  manually annotated clusters that recapitulate expected transition and dead-end trajectories.
when_to_use_negative:
- The analysis goal is gene regulatory network inference or in silico TF perturbation
  simulation rather than trajectory graph partitioning — use CellOracle's GRN-inference
  workflow instead.
- Input is a bulk RNA-seq matrix or a reference-based cell identity classification
  task — use Capybara's quadratic-programming classification instead.
- The dataset lacks a temporal or pseudotrajectory structure (e.g., a single-condition
  snapshot), so PAGA partitioning into transition/dead-end trajectories is not meaningful.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellOracle
  role: Provides the Python analysis framework (scanpy/PAGA-compatible) used to normalize,
    embed, run PAGA on the neighbor graph, and generate the force-directed layout
    with marker projections
  repo: https://github.com/morris-lab/CellOracle
- name: Cell Ranger 6.0.1
  role: Upstream processing of 10x Chromium scRNA-seq data into the count matrix that
    is the input to PAGA partitioning
- name: Python
  role: Implementation language for the CellOracle-based PAGA and force-directed layout
    workflow (tested with python 3.8 and 3.10)
  repo: https://github.com/morris-lab/CellOracle
- name: 10x Chromium
  role: Platform that generated the single-cell libraries underlying the reprocessed
    count matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:42d107bf3806f7b520973fff93bb7e144a5586a32dddef1056dd4a7dae89ec8a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/morris-lab/CellOracle
  repo_url: https://github.com/morris-lab/CellOracle
  tools:
  - 10x Chromium
  - Cell Ranger 6.0.1
  - CellOracle
  - Python
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# paga-trajectory-graph-partitioning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partition a single-cell kNN neighbor graph into a PAGA-derived cluster structure and render it as a force-directed layout, enabling trajectory recapitulation of a cell fate conversion time course (e.g., the 27,663-cell, 8-time-point MEF-to-iEP reprogramming dataset) with marker projection validation.

## When to use

Use when reprocessing a published scRNA-seq count matrix (e.g., 10x Chromium data processed with Cell Ranger v6.0.1) and the goal is to partition cells into manually annotated clusters that recapitulate expected transition and dead-end trajectories.

## When NOT to use

- The analysis goal is gene regulatory network inference or in silico TF perturbation simulation rather than trajectory graph partitioning — use CellOracle's GRN-inference workflow instead.
- Input is a bulk RNA-seq matrix or a reference-based cell identity classification task — use Capybara's quadratic-programming classification instead.
- The dataset lacks a temporal or pseudotrajectory structure (e.g., a single-condition snapshot), so PAGA partitioning into transition/dead-end trajectories is not meaningful.

## Inputs

- Published scRNA-seq count matrix processed with the Cell Ranger v6.0.1 pipeline (e.g., 27,663-cell, 8-time-point MEF-to-iEP dataset from 10x Chromium)
- Marker genes for identity projection (e.g., Apoa1 for iEPs, Col1a2 for fibroblasts)
- Precomputed single-cell neighbor graph after normalization, filtering, and embedding

## Outputs

- PAGA-partitioned clustering of cells (e.g., 15 clusters) with manual identity annotations (fibroblast, early transition, transition, dead-end, reprogrammed iEP)
- Force-directed graph layout of the partitioned single-cell graph
- Marker expression projections (Apoa1, Col1a2) overlaid on the force-directed layout, reproducing Figure 1F

## How to apply

Load the published count matrix (e.g., 27,663 cells across 8 reprogramming time points generated on the 10x Chromium platform and processed with Cell Ranger v6.0.1), then normalize, filter, and embed the cells. Run PAGA on the precomputed neighbor graph to partition the reprogramming time course into discrete clusters (15 in the reference implementation) and annotate each cluster by identity (fibroblast, early transition, transition, dead-end, or reprogrammed iEP). Compute a force-directed graph layout on the PAGA-partitioned single-cell graph for visualization. Project marker genes onto the layout — Apoa1 as the iEP marker and Col1a2 as the fibroblast marker — coloring cells by marker signal. Validate by confirming the output matches the published Figure 1F structure: 15 clusters present, Apoa1 signal enriched in iEP clusters and Col1a2 signal enriched in fibroblast clusters, confirmed by expert review against the published panel.

## Related tools

- **CellOracle** (Provides the Python analysis framework (scanpy/PAGA-compatible) used to normalize, embed, run PAGA on the neighbor graph, and generate the force-directed layout with marker projections) — https://github.com/morris-lab/CellOracle
- **Cell Ranger 6.0.1** (Upstream processing of 10x Chromium scRNA-seq data into the count matrix that is the input to PAGA partitioning)
- **Python** (Implementation language for the CellOracle-based PAGA and force-directed layout workflow (tested with python 3.8 and 3.10)) — https://github.com/morris-lab/CellOracle
- **10x Chromium** (Platform that generated the single-cell libraries underlying the reprocessed count matrix)

## Examples

```
import celloracle as co; co.app.is Available= ...
```

## Evaluation signals

- The PAGA partition yields exactly 15 clusters grouped into five cell types: fibroblasts, early transition, transition, dead-end, and reprogrammed iEPs.
- Apoa1 (iEP marker) expression projection is enriched in iEP clusters on the force-directed layout.
- Col1a2 (fibroblast marker) expression projection is enriched in fibroblast clusters on the force-directed layout.
- Expert review confirms the rendered figure reproduces the structure of the published Figure 1F panel.
- The input dataset matches the expected scale: 27,663 cells across 8 reprogramming time points.

## Limitations

- Cluster number (15) and cell-type groupings were manually annotated in the reference implementation; reproducing them requires matching the published annotation scheme, not an automatic parameter.
- Validation of the reproduction is by expert review against the published panel, which introduces subjectivity.
- Velocyto.py, occasionally used for trajectory-related analyses in this ecosystem, is no longer maintained; the README recommends STARSolo instead.

## Evidence

- [results] Workflow steps 1–4 from source card: [evidence span withheld — anchor status: not_found]
- [readme] Velocyto deprecation caveat: [evidence span withheld — anchor status: external_doc]
