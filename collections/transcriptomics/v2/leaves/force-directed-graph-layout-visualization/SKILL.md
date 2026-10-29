---
name: force-directed-graph-layout-visualization
description: Use when a single-cell dataset (e.g., a 27,663-cell, 8-time-point scRNA-seq
  time course) has been partitioned with PAGA into clusters and you need a 2D embedding
  that draws both cells and cluster-level edges to visualize trajectory and dead-end
  structure with marker overlays.
when_to_use_negative:
- The input is an unclustered count matrix with no PAGA partition or neighbor graph
  — layout must follow graph construction and clustering, not precede it
- The goal is trajectory pseudotime or GRN inference itself (CellOracle perturbation
  simulation) rather than visualization of an already-partitioned graph
- Data are bulk or pseudobulk profiles with no single-cell graph, since force-directed
  single-cell layouts require cell-level graph nodes
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellOracle
  role: Python pipeline used to embed cells, run PAGA on the neighbor graph, and compute
    the force-directed layout for visualization of the reprogramming trajectories
  repo: https://github.com/morris-lab/CellOracle
- name: igraph
  role: Graph library underlying the force-directed (spring) layout computation on
    the PAGA-partitioned single-cell graph; CellOracle checks its R installation as
    a dependency
  repo: https://github.com/igraph/rigraph
- name: Cell Ranger 6.0.1
  role: Produces the processed count matrix from 10x Chromium data that is the upstream
    input for graph construction and layout
- name: Python
  role: Language in which CellOracle (v3.8, tested with 3.10) implements the layout
    and marker-projection steps
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:9ef9478ef3d8623f5824db1043991f2b9da43d8e9db6dff4e0181b011be9125c
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
  - igraph
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# force-directed-graph-layout-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes a force-directed (spring-embedded) layout on a PAGA-partitioned single-cell neighbor graph and projects marker expression onto it, as done to reproduce the 15-cluster MEF-to-iEP reprogramming map of Figure 1F (Apoa1 vs. Col1a2 projections) using CellOracle and igraph in Python.

## When to use

Use when a single-cell dataset (e.g., a 27,663-cell, 8-time-point scRNA-seq time course) has been partitioned with PAGA into clusters and you need a 2D embedding that draws both cells and cluster-level edges to visualize trajectory and dead-end structure with marker overlays.

## When NOT to use

- The input is an unclustered count matrix with no PAGA partition or neighbor graph — layout must follow graph construction and clustering, not precede it
- The goal is trajectory pseudotime or GRN inference itself (CellOracle perturbation simulation) rather than visualization of an already-partitioned graph
- Data are bulk or pseudobulk profiles with no single-cell graph, since force-directed single-cell layouts require cell-level graph nodes

## Inputs

- Normalized, filtered scRNA-seq count matrix (e.g., Cell Ranger v6.0.1 output from 10x Chromium, 27,663 cells x 8 time points)
- PAGA-partitioned cluster assignments (15 clusters) on the single-cell neighbor graph
- Marker gene expression vectors (Apoa1, Col1a2)

## Outputs

- 2D force-directed graph layout coordinates for all cells with PAGA cluster edges
- Force-directed panel colored by Apoa1 (iEP marker) signal
- Force-directed panel colored by Col1a2 (fibroblast marker) signal, together recapitulating Figure 1F

## How to apply

First build and cluster the neighbor graph: load the Cell Ranger v6.0.1-processed 10x Chromium count matrix, normalize and filter cells, then run PAGA to partition the reprogramming time course into 15 clusters annotated as fibroblast, early transition, transition, dead-end, or reprogrammed iEP identities. Compute a force-directed (Fruchterman-Reingold-style spring) layout on the PAGA-partitioned graph — CellOracle's Python pipeline supports this via igraph, whose R library installation CellOracle explicitly checks. Finally, project marker expression onto the layout coordinates: Apoa1 (iEP marker) and Col1a2 (fibroblast marker), coloring cells by their normalized marker signal. The rationale is that the force-directed layout preserves both local cell-cell similarity and PAGA cluster connectivity, so trajectory topology and marker enrichment can be judged in a single panel comparable to Figure 1F.

## Related tools

- **CellOracle** (Python pipeline used to embed cells, run PAGA on the neighbor graph, and compute the force-directed layout for visualization of the reprogramming trajectories) — https://github.com/morris-lab/CellOracle
- **igraph** (Graph library underlying the force-directed (spring) layout computation on the PAGA-partitioned single-cell graph; CellOracle checks its R installation as a dependency) — https://github.com/igraph/rigraph
- **Cell Ranger 6.0.1** (Produces the processed count matrix from 10x Chromium data that is the upstream input for graph construction and layout)
- **Python** (Language in which CellOracle (v3.8, tested with 3.10) implements the layout and marker-projection steps)

## Examples

```
library(igraph); layout <- layout_with_fr(g)
```

## Evaluation signals

- The layout shows exactly 15 PAGA clusters grouped into the five expected cell types: fibroblasts, early transition, transition, dead-end, and reprogrammed iEPs
- Apoa1 signal is enriched in iEP clusters while Col1a2 signal is enriched in fibroblast clusters on the same layout coordinates
- The composed panel is judged by expert review to reproduce the structure of published Figure 1F (Kamimoto et al., Stem Cell Reports 2023)
- Layout is computed on the PAGA-partitioned graph, so cluster identities and inter-cluster edges are consistent between PAGA output and the drawn layout

## Limitations

- The layout is deterministic only given the same graph and seed; re-running preprocessing or clustering changes cluster number and layout topology
- 15 clusters is a manually annotated partition chosen to recapitulate this specific Biddy MEF-to-iEP dataset — other datasets may require different cluster resolutions
- CellOracle's R-side dependency igraph must be installed correctly (checked via installation script) or layout computation will fail

## Evidence

- [readme] igraph provides the network layout functionality on the R side: [evidence span withheld — anchor status: external_doc]
