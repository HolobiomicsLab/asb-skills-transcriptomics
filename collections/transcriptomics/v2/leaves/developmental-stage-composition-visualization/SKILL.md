---
name: developmental-stage-composition-visualization
description: Use when query scRNA-seq cells have been mapped onto a developmental
  reference atlas (e.g., mTCA) and you need to show stage/cell-type composition per
  condition (treatment day) to detect meiotic entry, progression, or developmental
  arrest.
when_to_use_negative:
- Query cells contain novel states absent from the reference atlas — fixed-model projections
  may be misleading, so compare unsupervised clustering to mapping results first
- Query data come from a different platform than the reference (e.g., Smart-seq2 projected
  onto 10x data) — ProjectSVR performance declines across platforms due to differing
  UMI distributions and sequencing depth
- The goal is de novo cell-type discovery or annotation without a well-constructed
  reference atlas, rather than placing cells into an existing reference space
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: ProjectSVR
  role: Maps query cells onto the mTCA reference (MapQuery), transfers labels (LabelTransfer,
    MajorityVote), and renders the composition figures (PlotProjection, AlluviaPlot)
  repo: https://github.com/JarningGau/ProjectSVR
- name: Seurat
  role: Holds the mapped query object and provides FeaturePlot for pan-somatic (Vim)
    / pan-germ-cell (Ddx4) label validation on ref.umap
  repo: https://github.com/satijalab/seurat
- name: AUCell
  role: Computes the gene set activity score matrix (0–1, rank-based) whose k-means
    over-clustering is used by MajorityVote to refine transferred stage labels
  repo: https://github.com/JarningGau/ProjectSVR
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:9e89e6bde7bacdb5379b2a641821d1ac058f9c4f937356330974da42612cb7c8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/jarninggau/ProjectSVR
  repo_url: https://github.com/JarningGau/ProjectSVR
  tools:
  - AUCell
  - ProjectSVR
  - Seurat
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# developmental-stage-composition-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualizes how query cell composition across developmental stages changes over conditions (e.g., NRRA treatment day) by projecting cells onto a pre-built reference atlas (mTCA) with ProjectSVR and rendering per-day alluvia and projection plots filled by majority-voted transferred labels.

## When to use

Use when query scRNA-seq cells have been mapped onto a developmental reference atlas (e.g., mTCA) and you need to show stage/cell-type composition per condition (treatment day) to detect meiotic entry, progression, or developmental arrest.

## When NOT to use

- Query cells contain novel states absent from the reference atlas — fixed-model projections may be misleading, so compare unsupervised clustering to mapping results first
- Query data come from a different platform than the reference (e.g., Smart-seq2 projected onto 10x data) — ProjectSVR performance declines across platforms due to differing UMI distributions and sequencing depth
- The goal is de novo cell-type discovery or annotation without a well-constructed reference atlas, rather than placing cells into an existing reference space

## Inputs

- Pre-built reference model RDS file (e.g., model.mTCA.rds from Zenodo record 8350732)
- Query Seurat object in .qs format (e.g., query_NRRA_induced_meiotic_cells.seurat.slim.qs from Zenodo record 8350748)
- Mapped Seurat object with mapQ.p.adj quality scores and a cell_type label column in the reference
- AUCell gene set score matrix used for k-means over-clustering prior to MajorityVote

## Outputs

- Projection plots on ref.umap of query cells split by treatment day (PlotProjection)
- Alluvia plots of meiosis progression: day on one axis, majority-voted predicted cell type as fill, restricted to meiotic stages m11–m15 for the meiosis readout
- Meiotic-entry fraction summary (expected ~4–5% of germ cells after 3 days NRRA)
- Validation FeaturePlot panels of Vim (pan-somatic) and Ddx4 (pan-germ-cell) on ref.umap

## How to apply

First map the query (e.g., query_NRRA_induced_meiotic_cells.seurat.slim.qs) onto the pre-built mTCA model (model.mTCA.rds) with ProjectSVR::MapQuery (add.map.qual = TRUE, ncores = 10), then retain well-mapped cells by subsetting on mapQ.p.adj < 0.1. Transfer reference cell-type labels with ProjectSVR::LabelTransfer (ref.label.col = "cell_type") and refine them with MajorityVote (k = 100, min.prop = 0.3) computed over k-means over-clustering of the AUCell gene set score matrix, so isolated label noise is smoothed by neighborhood consensus. Visualize projected query cells on ref.umap split by treatment day using PlotProjection (ref.color.by = "cell_type", ref.size = 0.5, ref.alpha = 0.3), and summarize progression with AlluviaPlot by day filled by the majority-voted predicted cell type. Validate assignments with Seurat FeaturePlot of the pan-somatic marker Vim and pan-germ-cell marker Ddx4 on ref.umap, flagging wrong assignments such as m9-PGC_Piwil4_arrest; for the NRRA meiosis question, restrict alluvia to meiotic stages m11–m15 and expect roughly 4–5% of germ cells to enter meiosis after 3 days of NRRA treatment.

## Related tools

- **ProjectSVR** (Maps query cells onto the mTCA reference (MapQuery), transfers labels (LabelTransfer, MajorityVote), and renders the composition figures (PlotProjection, AlluviaPlot)) — https://github.com/JarningGau/ProjectSVR
- **Seurat** (Holds the mapped query object and provides FeaturePlot for pan-somatic (Vim) / pan-germ-cell (Ddx4) label validation on ref.umap) — https://github.com/satijalab/seurat
- **AUCell** (Computes the gene set activity score matrix (0–1, rank-based) whose k-means over-clustering is used by MajorityVote to refine transferred stage labels) — https://github.com/JarningGau/ProjectSVR

## Examples

```
ProjectSVR::PlotProjection(seu.q, ref.color.by = "cell_type", ref.size = 0.5, ref.alpha = 0.3)
```

## Evaluation signals

- Only well-mapped cells feed the composition plots: subset on mapQ.p.adj < 0.1 before label transfer and plotting
- FeaturePlot sanity check: Vim signal restricted to somatic reference clusters and Ddx4 restricted to germ-cell clusters; misassignments such as m9-PGC_Piwil4_arrest must be flagged and excluded
- Alluvia restricted to meiotic stages m11–m15 reproduce the reported outcome that roughly 4–5% of germ cells enter meiosis after 3 days of NRRA treatment
- MajorityVote parameters are as specified (k = 100, min.prop = 0.3) over k-means over-clustering of the gene set score matrix, and refined labels are more spatially coherent on ref.umap than raw transferred labels
- Projected day-resolved cells occupy the expected mTCA pseudotime region; in the reference, query cells should be correctly placed along pseudotime as reported for mTCA (Fig. S9b)

## Limitations

- If query cells contain novel states absent in the reference, fixed-model projections may be misleading; compare unsupervised clustering to mapping results to detect inconsistencies
- Performance declines when projecting across platforms (e.g., Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth
- ProjectSVR trains independent SVR models for each UMAP dimension, neglecting correlations among embedding dimensions, which may limit joint structure learning

## Evidence
