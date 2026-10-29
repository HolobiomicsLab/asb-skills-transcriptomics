---
name: mapping-quality-filtering
description: Use when cells have been projected into a reference UMAP space via ProjectSVR
  (ref.umap reduction with map-quality metrics attached) and you must retain only
  well-mapped cells before label transfer, pseudotime prediction, or cross-condition
  composition comparisons.
when_to_use_negative:
- The query has not yet been projected through a pre-trained ProjectSVR model — without
  the ref.umap embedding and attached map-quality metrics there is nothing to filter
  on
- You want to detect novel query cell states absent from the reference — the article
  warns fixed-model projections may be misleading for such cells, and the article
  recommends comparing unsupervised clustering to mapping results rather than only
  filtering
tools:
- name: ProjectSVR
  role: Provides MapQuery/AddProjQual to compute map-quality metrics (mean.knn.dist,
    mapQ.p.adj), MapQCPlot to visualise and choose cutoffs, and the subset() workflow
    to drop low-quality mapped cells
  repo: https://github.com/JarningGau/ProjectSVR
- name: Seurat
  role: Stores the ref.umap reduction and quality metrics as metadata and supplies
    the subset() mechanism used to retain well-mapped cells
  repo: https://github.com/JarningGau/ProjectSVR
provenance:
  source_task_ids:
  - task_005
  - task_007
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:6dd03e8e5eb1d5974ba4ffd5548f7899af1969905d0a929af5b01210fcc751be
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
  - ProjectSVR
  - Seurat
  - UCell
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# mapping-quality-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess how well query cells projected onto a ProjectSVR reference atlas actually fit that reference, and drop poorly mapped cells using quantitative quality metrics (mean.knn.dist and adjusted p-value from projection-quality testing) before downstream label transfer or composition analysis.

## When to use

Use when cells have been projected into a reference UMAP space via ProjectSVR (ref.umap reduction with map-quality metrics attached) and you must retain only well-mapped cells before label transfer, pseudotime prediction, or cross-condition composition comparisons.

## When NOT to use

- The query has not yet been projected through a pre-trained ProjectSVR model — without the ref.umap embedding and attached map-quality metrics there is nothing to filter on
- You want to detect novel query cell states absent from the reference — the article warns fixed-model projections may be misleading for such cells, and the article recommends comparing unsupervised clustering to mapping results rather than only filtering

## Inputs

- Seurat query object with a 'ref.umap' reduction and per-cell map-quality metadata columns (mean.knn.dist, mapQ.p.adj) produced by ProjectSVR::MapQuery with add.map.qual = TRUE or ProjectSVR::AddProjQual
- Pre-trained ProjectSVR reference model (.rds, e.g. model.Vento2018.MFI.rds, model.mTCA.rds, model.disco_pbmc.rds) defining the mapping-quality context

## Outputs

- Filtered Seurat query object (seu.q) containing only well-mapped cells
- MapQCPlot diagnostic figure showing the chosen map.q.cutoff relative to the mean.knn.dist distribution
- Downstream-ready cell set for label transfer, LISI evaluation, or pseudotime prediction

## How to apply

After running ProjectSVR::MapQuery with add.map.qual = TRUE (or ProjectNewdata followed by AddProjQual with k = 20, repeats = 1e4), each query cell carries mean.knn.dist and mapQ.p.adj metrics in the ref.umap embedding. Use ProjectSVR::MapQCPlot to visualise the distribution and set a cutoff that separates well-mapped from badly projected cells; the cutoffs are atlas- and query-specific, e.g. mean.knn.dist < 2 for the Vento2018 maternal-fetal interface (RPL decidual NK) query, mean.knn.dist < 2.4 for the mTCA germ-cell (Zfp541-KO) query, mean.knn.dist < 1.6 plus mapQ.p.adj < 0.1 in other vignettes. Subset the Seurat object to keep only cells passing the threshold before running KnnLabelTransfer/MajorityVote or LabelTransfer, computing per-sample subtype frequencies (e.g. dnk_proportions.csv), or predicting pseudotime. The rationale is that reference-mapping methods can force novel query states into reference space, so filtering protects downstream comparisons from cells that do not genuinely correspond to any reference population.

## Related tools

- **ProjectSVR** (Provides MapQuery/AddProjQual to compute map-quality metrics (mean.knn.dist, mapQ.p.adj), MapQCPlot to visualise and choose cutoffs, and the subset() workflow to drop low-quality mapped cells) — https://github.com/JarningGau/ProjectSVR
- **Seurat** (Stores the ref.umap reduction and quality metrics as metadata and supplies the subset() mechanism used to retain well-mapped cells) — https://github.com/JarningGau/ProjectSVR

## Examples

```
seu.q <- subset(seu.q, mean.knn.dist < 2)
```

## Evaluation signals

- Every retained cell satisfies the chosen cutoff (e.g. all cells have mean.knn.dist < 2 or < 2.4, or mapQ.p.adj < 0.1 as set in the vignette)
- MapQCPlot output clearly separates good and bad projections at the chosen map.q.cutoff before subsetting
- The number of dropped cells is non-trivial but a large majority of query cells is retained; near-total loss signals a mismatched reference or wrong cutoff
- Downstream results computed on filtered cells are consistent with published expectations — e.g. dNK1 decreased/dNK3 increased in RPL (Guo2021 Fig. 4e-f) or pseudotime correlation r = 0.973 for Zfp541-KO (Fig. S9b)
- The comparison between unsupervised clustering and mapping results shows no unexplained inconsistencies after filtering

## Limitations

- Cutoffs are dataset-specific (1.6, 2, and 2.4 appear in different vignettes) and must be re-derived per reference/query pair via MapQCPlot rather than copied blindly
- If query cells contain novel states absent in the reference, fixed-model projections may be misleading; users should compare unsupervised clustering to mapping results to detect inconsistencies
- Projecting across platforms (e.g. Smart-seq2 to 10x) degrades performance due to differing UMI distributions and sequencing depth, which inflates mean.knn.dist and can cause over-filtering

## Evidence
