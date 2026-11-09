---
name: lisi-batch-mixing-quantification
description: Use when a query single-cell RNA-seq dataset carries explicit batch or
  technology-of-origin labels (e.g., three 10x protocols 3'v1/3'v2/5') and you need
  a numeric measure of how well batches mix before vs. after projecting cells into
  a reference atlas embedding.
when_to_use_negative:
- The query contains no batch/technology covariate to quantify mixing over (LISI needs
  a categorical label such as protocol, donor, or batch)
- The query cells contain novel cell states absent from the reference atlas — fixed-model
  projections may be misleading, so the LISI-on-mapped-embedding comparison would
  not be interpretable
- You are evaluating batch integration rather than reference mapping and have no pre-built
  model/reference embedding to project into
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ProjectSVR
  role: Projects query cells into the reference UMAP space (ref.umap) whose embeddings
    are then scored with LISI; produces map-quality metrics used to retain cells for
    the LISI evaluation
  repo: https://github.com/jarninggau/ProjectSVR
- name: Seurat
  role: Hosts the query/reference Seurat objects, batch labels, and low-dimensional
    embeddings on which LISI is computed before and after mapping
- name: MapQCPlot
  role: Provides adjusted p-value (mapQ.p.adj) and mean.knn.dist cutoffs to filter
    out poorly mapped cells before the LISI evaluation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:398af134068dd4d95b7f234924fc35667ccf8b9eb078f5a98e319dbe904565e0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jarninggau/ProjectSVR
  repo_url: https://github.com/jarninggau/ProjectSVR
  tools:
  - MapQCPlot
  - ProjectSVR
  - Seurat
  - UCell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# lisi-batch-mixing-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies batch/protocol mixing in a single-cell embedding using the Local Inverse Simpson's Index (LISI), comparing uncorrected vs. reference-mapped embeddings (e.g., ProjectSVR-projected ref.umap) to demonstrate that platform-driven clustering (10x 3'v1/3'v2/5') is corrected in favor of biological identity.

## When to use

Use when a query single-cell RNA-seq dataset carries explicit batch or technology-of-origin labels (e.g., three 10x protocols 3'v1/3'v2/5') and you need a numeric measure of how well batches mix before vs. after projecting cells into a reference atlas embedding.

## When NOT to use

- The query contains no batch/technology covariate to quantify mixing over (LISI needs a categorical label such as protocol, donor, or batch)
- The query cells contain novel cell states absent from the reference atlas — fixed-model projections may be misleading, so the LISI-on-mapped-embedding comparison would not be interpretable
- You are evaluating batch integration rather than reference mapping and have no pre-built model/reference embedding to project into

## Inputs

- Query Seurat object (.qs/.rds, e.g. query_hPBMCs.seurat.slim.qs) with per-cell batch covariate labeling 10x protocol of origin (3'v1, 3'v2, 5')
- Reference atlas (e.g. DISCO_hPBMCs.seurat.slim.qs) and pre-trained reference model (model.disco_pbmc.rds) providing the ref.umap reduction
- Uncorrected low-dimensional embedding of query cells (pre-mapping baseline)
- MapQC metrics per cell (mapQ.p.adj, mean.knn.dist) for retaining well-mapped cells

## Outputs

- Per-cell LISI scores computed on uncorrected and projected embeddings
- Mean LISI values per batch label, pre- and post-mapping (baseline ~1.06 vs. corrected ~2.30 in the PBMC benchmark)
- Comparison table (e.g., lisi_comparison.csv) summarizing uncorrected vs. post-mapping mixing

## How to apply

First compute LISI on the uncorrected low-dimensional embedding of the query cells to establish the pre-mapping baseline (in the PBMC DISCO benchmark this was ~1.06, indicating cells cluster by 10x protocol). Then map the query cells into the reference space — e.g., with ProjectSVR::MapQuery(add.map.qual = TRUE, ncores = 10) projecting into the ref.umap reduction — and restrict the evaluation to well-mapped cells using MapQCPlot thresholds such as mapQ.p.adj < 0.1 and a mean.knn.dist cutoff (values between 1.6 and 2.4 are used in the paper's methods). Recompute LISI on the projected ref.umap embeddings; a corrected result should show markedly higher mixing (mean LISI ~2.30 in the benchmark), matching the reported values. Write the pre/post comparison (e.g., to lisi_comparison.csv) to make the batch-correction effect explicit and auditable.

## Related tools

- **ProjectSVR** (Projects query cells into the reference UMAP space (ref.umap) whose embeddings are then scored with LISI; produces map-quality metrics used to retain cells for the LISI evaluation) — https://github.com/jarninggau/ProjectSVR
- **Seurat** (Hosts the query/reference Seurat objects, batch labels, and low-dimensional embeddings on which LISI is computed before and after mapping)
- **MapQCPlot** (Provides adjusted p-value (mapQ.p.adj) and mean.knn.dist cutoffs to filter out poorly mapped cells before the LISI evaluation)

## Examples

```
seu.q <- ProjectSVR::MapQuery(seu.q, reference, ref.umap, add.map.qual = TRUE, ncores = 10); seu.q <- subset(seu.q, mapQ.p.adj < 0.1); # then recompute LISI on reductions(seu.q)$ref.umap
```

## Evaluation signals

- Post-mapping mean LISI on the ref.umap embedding (~2.30 in the PBMC benchmark) should clearly exceed the uncorrected baseline (~1.06)
- LISI must be recomputed on the projected ref.umap reduction, not on the original uncorrected PCA/UMAP, for the post-mapping value
- Only cells passing map-quality filters (mapQ.p.adj < 0.1 and a mean.knn.dist cutoff, e.g. 1.6–2.4) should be included in the final LISI comparison
- The pre/post comparison should be persisted as a table (e.g., lisi_comparison.csv) with values matching the reported benchmark figures
- Mixing should be by biological identity rather than protocol: post-mapping LISI over the three 10x protocols should approach the reference-mapping performance of leading methods (Seurat, scArches, Symphony, ProjecTILs)

## Limitations

- ProjectSVR performance declines when projecting across platforms (e.g., Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth, which would confound a LISI-based batch-mixing claim
- If query cells contain novel states absent from the reference, fixed-model projections may be misleading; unsupervised clustering should be compared to mapping results before interpreting LISI
- LISI here quantifies mixing of protocol labels only for retained, well-mapped cells; filtering thresholds change the population and thus the metric

## Evidence

- [other] Recompute LISI on the projected ref.umap embeddings to quantify post-mapping mixing of the three 10x protocols.: [evidence span withheld — anchor status: not_found]
