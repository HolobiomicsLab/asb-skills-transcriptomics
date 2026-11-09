---
name: knn-distance-null-distribution-permutation-testing
description: Use when projected query cells (e.g. PBMC query mapped onto the DISCO
  PBMC reference via ProjectNewdata) need a statistical per-cell mapping-quality score
  before downstream label transfer or filtering.
when_to_use_negative:
- The query has not yet been projected into the reference UMAP space (AddProjQual
  operates on projected cells, not raw counts)
- Query cells contain novel cell states absent from the reference — a low quality
  score there reflects reference incompleteness, not necessarily bad mapping
- Cross-platform projection with severe depth mismatch (e.g. Smart-seq2 to 10x) —
  the article reports performance declines in this regime, so distance-based quality
  metrics may be miscalibrated
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3934
tools:
- name: ProjectSVR
  role: Provides AddProjQual, ProjectNewdata, and MapQCPlot for computing mean.knn.dist,
    the null distribution, and BH-corrected empirical p-values on projected cells
  repo: https://github.com/JarningGau/ProjectSVR
- name: Seurat
  role: Holds the query Seurat object whose ref.umap reduction and metadata columns
    (mean.knn.dist, mapQ.p.val, mapQ.p.adj) AddProjQual reads and writes; subset()
    applies the resulting filters
  repo: https://github.com/satijalab/seurat
- name: AUCell
  role: Computes the gene set activity score matrix used as input features for the
    projection whose quality is then assessed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbaf586
    title: 'ProjectSVR: mapping single-cell RNA-seq data to reference atlases by supported
      vector regression'
  source_content_hash: sha256:00ad7953ff49002138f2a4d7b454c0c3990db2ec2f9e69d141021b377868dfdc
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
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# kNN-distance null-distribution permutation testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

ProjectSVR's mapping-quality assessment (AddProjQual) scores each projected query cell by its mean k-nearest-neighbour distance (mean.knn.dist, k=20) in the reference UMAP space, builds a null distribution by repeatedly computing the mean distance of K randomly selected cells (repeats = 1e4), and derives empirical p-values with Benjamini–Hochberg correction (mapQ.p.val, mapQ.p.adj) to flag poorly mapped cells.

## When to use

Use when projected query cells (e.g. PBMC query mapped onto the DISCO PBMC reference via ProjectNewdata) need a statistical per-cell mapping-quality score before downstream label transfer or filtering.

## When NOT to use

- The query has not yet been projected into the reference UMAP space (AddProjQual operates on projected cells, not raw counts)
- Query cells contain novel cell states absent from the reference — a low quality score there reflects reference incompleteness, not necessarily bad mapping
- Cross-platform projection with severe depth mismatch (e.g. Smart-seq2 to 10x) — the article reports performance declines in this regime, so distance-based quality metrics may be miscalibrated

## Inputs

- Seurat query object with projected ref.umap reduction (e.g. query_hPBMCs.seurat.slim.qs after ProjectNewdata)
- Pre-built ProjectSVR reference model (e.g. model.disco_pbmc.rds)
- k (default 20) and repeats (default 1e4) for null-distribution construction

## Outputs

- Per-cell metadata columns: mean.knn.dist, mapQ.p.val, mapQ.p.adj
- Per-cell metrics table (map_quality_metrics.csv)
- MapQCPlot visualization with p.adj.cutoff and mean.knn.dist cutoffs

## How to apply

After projecting the query with ProjectNewdata and attaching the ref.umap reduction to the Seurat object, run AddProjQual(object, k = 20, repeats = 1e4): it builds a k-NN network on the projected cells, computes per-cell mean.knn.dist, and generates a null distribution by sampling K random cells 1e4 times, yielding empirical p-values (p.val) that are BH-corrected (p.adj). Store mean.knn.dist, mapQ.p.val, and mapQ.p.adj in query metadata and visualize with MapQCPlot using an adjusted-p cutoff (e.g. p.adj.cutoff = 1e-3) and a mean.knn.dist cutoff (e.g. 1.5–1.6). The article's own workflows use dataset-specific mean.knn.dist thresholds for subset() filtering — mean.knn.dist < 1.6, < 2, or < 2.4 — so calibrate the cutoff to the reference atlas rather than reusing one value blindly.

## Related tools

- **ProjectSVR** (Provides AddProjQual, ProjectNewdata, and MapQCPlot for computing mean.knn.dist, the null distribution, and BH-corrected empirical p-values on projected cells) — https://github.com/JarningGau/ProjectSVR
- **Seurat** (Holds the query Seurat object whose ref.umap reduction and metadata columns (mean.knn.dist, mapQ.p.val, mapQ.p.adj) AddProjQual reads and writes; subset() applies the resulting filters) — https://github.com/satijalab/seurat
- **AUCell** (Computes the gene set activity score matrix used as input features for the projection whose quality is then assessed)

## Examples

```
seu.q <- ProjectSVR::AddProjQual(seu.q, k = 20, repeats = 1e4); seu.q <- subset(seu.q, mean.knn.dist < 1.6)
```

## Evaluation signals

- The fraction of cells with mean.knn.dist > 1.5 should match the reported ~2.6% low-quality mapping fraction on the PBMC demo
- mapQ.p.adj values are BH-corrected empirical p-values within [0, 1]; cells with mapQ.p.adj < 0.1 are retained in the article's own filter
- Null distribution reproducibility: rerunning AddProjQual with k = 20 and repeats = 1e4 should yield stable empirical p-values
- Output metadata must contain exactly the columns mean.knn.dist, mapQ.p.val, and mapQ.p.adj
- Cells subset with mean.knn.dist < 1.6 (or the dataset-appropriate cutoff, e.g. < 2, < 2.4) should form the high-confidence mapped population used for LabelTransfer

## Limitations

- Fixed-model projections may be misleading if query cells contain novel states absent in the reference; the article advises comparing unsupervised clustering to mapping results to detect inconsistencies
- Mean.knn.dist thresholds are atlas-specific (1.6, 2, and 2.4 all appear in the article's workflows) — 'Determine the threshold to distinguish the good and bad projections' is a required per-dataset step
- Performance declines when projecting across platforms (e.g. Smart-seq2 to 10x) due to differing UMI distributions and sequencing depth, which affects the interpretability of distance-based quality scores

## Evidence

- [full_text] Run AddProjQual(object, k = 20, repeats = 1e4) to build the k-NN network on the projected cells, compute per-cell mean.knn.dist, generate the null distribution from randomly sampled cells (1e4 repeats), and derive empirical p-values with Benjamini-Hochberg correction (p.val, p.adj).: [evidence span withheld — anchor status: not_found]
