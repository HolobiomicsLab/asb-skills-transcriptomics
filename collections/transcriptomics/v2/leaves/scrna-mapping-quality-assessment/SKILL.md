---
name: scrna-mapping-quality-assessment
description: Use when query cells have been projected onto a ProjectSVR reference
  (ref.umap attached) and you must decide which cells to retain for label transfer
  or downstream tests, e.g. via mean.knn.dist < 1.6 or mapQ.p.adj < 0.1.
when_to_use_negative:
- Query cells contain novel biological states absent from the reference — fixed-model
  projections may be misleading, so compare against unsupervised clustering instead
  of trusting map-quality filters alone
- The query was generated on a different platform than the reference (e.g. Smart-seq2
  projected onto 10x data) — performance declines due to differing UMI distributions
  and sequencing depth, so cutoffs calibrated on matched-platform data may not transfer
- You need a per-cell mapping confidence metric for a non-ProjectSVR mapper (Seurat,
  scArches, Symphony, ProjecTILs) — this empirical k-NN null procedure is defined
  on the ProjectSVR projected UMAP embedding
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: ProjectSVR
  role: Provides AddProjQual (mean.knn.dist + BH-corrected empirical p-values), MapQCPlot
    visualization, and MapQuery with add.map.qual = TRUE that attaches mapping-quality
    metadata
  repo: https://github.com/JarningGau/ProjectSVR
- name: Seurat
  role: Holds the query object and ref.umap reduction; subset() on mean.knn.dist /
    mapQ.p.adj metadata filters low-quality mapped cells
  repo: https://github.com/satijalab/seurat
- name: AUCell
  role: Computes the gene set activity scores that the projection (and hence the k-NN
    distances being assessed) is built on
provenance:
  source_task_ids:
  - task_008
  - task_004
  - task_006
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:ad3bd13cbded61044b7510fb9506d6c971bdfc502f83ff46602fc58ed088537f
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
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# scrna-mapping-quality-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess whether query cells projected onto a pre-built ProjectSVR reference model were mapped reliably, using the mean k-NN distance in the projected UMAP space and BH-corrected empirical p-values (AddProjQual/MapQCPlot) before filtering to well-mapped cells.

## When to use

Use when query cells have been projected onto a ProjectSVR reference (ref.umap attached) and you must decide which cells to retain for label transfer or downstream tests, e.g. via mean.knn.dist < 1.6 or mapQ.p.adj < 0.1.

## When NOT to use

- Query cells contain novel biological states absent from the reference — fixed-model projections may be misleading, so compare against unsupervised clustering instead of trusting map-quality filters alone
- The query was generated on a different platform than the reference (e.g. Smart-seq2 projected onto 10x data) — performance declines due to differing UMI distributions and sequencing depth, so cutoffs calibrated on matched-platform data may not transfer
- You need a per-cell mapping confidence metric for a non-ProjectSVR mapper (Seurat, scArches, Symphony, ProjecTILs) — this empirical k-NN null procedure is defined on the ProjectSVR projected UMAP embedding

## Inputs

- Query Seurat object with projected ref.umap reduction attached (e.g. query_NRRA_induced_meiotic_cells.seurat.slim.qs, query_hPBMCs.seurat.slim.qs)
- Pre-built ProjectSVR reference model (.rds, e.g. model.mTCA.rds, model.disco_pbmc.rds, model.Zheng2021.CD8Tcell.rds)
- Gene set activity score matrix (AUCell/UCell) used for the projection

## Outputs

- Per-cell metadata columns mean.knn.dist, mapQ.p.val, mapQ.p.adj
- MapQCPlot visualization with chosen p.adj and mean.knn.dist cutoffs
- QC table (e.g. map_quality_metrics.csv) and filtered Seurat object retaining only well-mapped cells

## How to apply

After projecting the query into the reference UMAP space (ProjectSVR::MapQuery with add.map.qual = TRUE, or ProjectNewdata followed by AddProjQual), run AddProjQual(object, k = 20, repeats = 1e4): it builds a k-NN network on the projected cells, computes per-cell mean.knn.dist, generates a null distribution from randomly sampled cells over 1e4 repeats, and derives empirical p-values with Benjamini–Hochberg correction, storing mean.knn.dist, mapQ.p.val and mapQ.p.adj in cell metadata. Visualize with MapQCPlot using an adjusted-p cutoff (e.g. p.adj.cutoff = 1e-3 or 0.1) and/or a mean.knn.dist cutoff (e.g. 1.5–1.6; mTCA applications used 1.6, 2, or 2.4 depending on dataset). Then subset the Seurat object on the chosen cutoff (e.g. subset(seu.q, mean.knn.dist < 1.6)) so that only well-mapped cells feed LabelTransfer, AlluviaPlot, or abundance tests. The rationale is that cells falling far from any reference neighbor in UMAP space are poorly modeled by the fixed SVR projection and can corrupt transferred labels.

## Related tools

- **ProjectSVR** (Provides AddProjQual (mean.knn.dist + BH-corrected empirical p-values), MapQCPlot visualization, and MapQuery with add.map.qual = TRUE that attaches mapping-quality metadata) — https://github.com/JarningGau/ProjectSVR
- **Seurat** (Holds the query object and ref.umap reduction; subset() on mean.knn.dist / mapQ.p.adj metadata filters low-quality mapped cells) — https://github.com/satijalab/seurat
- **AUCell** (Computes the gene set activity scores that the projection (and hence the k-NN distances being assessed) is built on)

## Examples

```
seu.q <- ProjectSVR::AddProjQual(seu.q, k = 20, repeats = 1e4); ProjectSVR::MapQCPlot(seu.q, p.adj.cutoff = 1e-3); seu.q <- subset(seu.q, mean.knn.dist < 1.6)
```

## Evaluation signals

- The fraction of cells with mean.knn.dist > 1.5 in the PBMC quick-start should match the reported ~2.6% low-quality mapping fraction
- Metadata of the query object must contain three new columns: mean.knn.dist, mapQ.p.val, mapQ.p.adj after AddProjQual with k = 20, repeats = 1e4
- MapQCPlot with p.adj.cutoff = 1e-3 (or 0.1) should separate a small tail of low-quality cells; subsets at mean.knn.dist < 1.6 / 2 / 2.4 are all legitimate per-application thresholds seen in the mTCA and ICB workflows
- Downstream label transfer should only run on the filtered object (e.g. seu.q <- subset(seu.q, mean.knn.dist < 1.6) immediately before LabelTransfer), and wrong assignments like m9-PGC_Piwil4_arrest flagged via Vim/Ddx4 FeaturePlots indicate mapping errors rather than biology
- Exported map_quality_metrics.csv should have one row per query cell with the three QC columns and no NAs introduced by the null-distribution step

## Limitations

- Independent SVR models per UMAP dimension neglect correlations among embedding dimensions, which may limit how faithfully mean.knn.dist reflects joint embedding structure
- For query cells with novel states absent in the reference, fixed-model projections may be misleading; users should compare unsupervised clustering to mapping results to detect inconsistencies
- Performance declines when projecting across platforms (e.g. Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth
- The cutoff (p.adj.cutoff vs mean.knn.dist) is application-specific — the paper uses 1.6, 2, 2.4, and 0.1 in different workflows, so thresholds must be justified per dataset

## Evidence
