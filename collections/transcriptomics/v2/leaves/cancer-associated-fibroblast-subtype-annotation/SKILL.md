---
name: cancer-associated-fibroblast-subtype-annotation
description: Use when a scRNA-seq stromal/CAF clustering partition contains low-ROGUE
  (impure) clusters and finer subtypes — such as myCAF splits or CD74/MHC-II+ apCAFs
  — must be resolved and annotated.
when_to_use_negative:
- The input partition already consists of high-ROGUE (pure) clusters near the 0.9
  purity threshold — no re-clustering is needed.
- Data are low-quality or continuous/trajectory-like, where the 0.9 ROGUE threshold
  is not appropriate and the cutoff must instead be determined from global ROGUE values.
- The goal is to assess purity of integrated cell populations from different protocols/platforms
  — the paper states ROGUE does not yet support this.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: ROGUE
  role: Entropy-based statistic used to quantify cluster purity (matr.filter, entropy_fit,
    DetermineK, CalculateRogue) and guide re-clustering of impure CAF clusters
  repo: https://github.com/PaulingLiu/ROGUE
- name: tidyverse
  role: Dependency R package that must be installed before installing ROGUE
- name: devtools
  role: Used to install ROGUE from GitHub via devtools::install_github("PaulingLiu/ROGUE")
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-16904-3
    title: An entropy-based metric for assessing the purity of single cell populations
  source_content_hash: sha256:8b9b8b473044c35f12a399fae9497a3a71b19b2a0f697f66ace4bde3bb759cef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PaulingLiu/ROGUE
  tools:
  - R
  - ROGUE
  - devtools
  - tidyverse
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cancer-associated-fibroblast-subtype-annotation

## Summary

ROGUE-guided re-clustering of low-purity cancer-associated fibroblast (CAF) clusters to resolve purer subtypes (e.g., splitting the myCAF population into RGS5/ACTA2/GPX3 clusters and identifying an antigen-presenting apCAF cluster), then annotating subtypes by marker genes.

## When to use

Use when a scRNA-seq stromal/CAF clustering partition contains low-ROGUE (impure) clusters and finer subtypes — such as myCAF splits or CD74/MHC-II+ apCAFs — must be resolved and annotated.

## When NOT to use

- The input partition already consists of high-ROGUE (pure) clusters near the 0.9 purity threshold — no re-clustering is needed.
- Data are low-quality or continuous/trajectory-like, where the 0.9 ROGUE threshold is not appropriate and the cutoff must instead be determined from global ROGUE values.
- The goal is to assess purity of integrated cell populations from different protocols/platforms — the paper states ROGUE does not yet support this.

## Inputs

- lung-tumor stromal scRNA-seq count matrix (e.g., Kim et al.) with original 7-cluster assignments
- original cluster partition for ROGUE purity calculation
- min.cells and min.genes thresholds for matr.filter

## Outputs

- re-clustered 11-subtype CAF partition with higher average ROGUE than the original 7 clusters
- per-cluster ROGUE purity table for both partitions
- annotated subtypes (myCAF: BF_C01_RGS5, BF_C02_ACTA2, BF_C03_GPX3; apCAF: CD74/MHC-II+) saved as caf_rogue_subtypes.csv
- UMAP embedding of re-clustered stromal cells

## How to apply

Load the lung-tumor stromal scRNA-seq count matrix with the original 7-cluster assignments and pre-filter genes/cells using ROGUE's matr.filter (min.cells, min.genes). Fit the entropy model with ROGUE's entropy_fit and DetermineK to select S-E (significantly entropic) variable genes for downstream clustering. Normalize counts (Seurat negative-binomial regression), batch-correct the embedding with BBKNN, and re-cluster the low-ROGUE stromal clusters with leiden, visualizing by UMAP. Compute per-cluster purity with ROGUE's CalculateRogue on both the original 7-cluster and the new partitions, and re-cluster further only for clusters below the purity threshold. Annotate subtypes by markers: myCAFs by RGS5/ACTA2/GPX3 and apCAFs by CD74 and MHC class-II gene expression.

## Related tools

- **ROGUE** (Entropy-based statistic used to quantify cluster purity (matr.filter, entropy_fit, DetermineK, CalculateRogue) and guide re-clustering of impure CAF clusters) — https://github.com/PaulingLiu/ROGUE
- **tidyverse** (Dependency R package that must be installed before installing ROGUE)
- **devtools** (Used to install ROGUE from GitHub via devtools::install_github("PaulingLiu/ROGUE"))

## Examples

```
if (!requireNamespace("devtools", quietly=TRUE)) install.packages("devtools"); devtools::install_github("PaulingLiu/ROGUE")
```

## Evaluation signals

- Re-clustering must yield 11 subtypes from the original 7 low-ROGUE lung-tumor fibroblast clusters, with higher average ROGUE than the original partition.
- The original myCAF cluster (AF_C02_COL4A1, ROGUE = 0.81) must split into BF_C01_RGS5 (ROGUE ≈ 0.84), BF_C02_ACTA2 (≈ 0.87), and BF_C03_GPX3 (≈ 0.94).
- An apCAF cluster must be present with high CD74 and MHC class-II gene expression.
- Per-cluster ROGUE values are computed for both the original and re-clustered partitions and compared before/after; the results table caf_rogue_subtypes.csv is saved.
- Subtype annotations are consistent with the marker panels (myCAF: RGS5/ACTA2/GPX3; apCAF: CD74/MHC-II).

## Limitations

- The recommended 0.9 purity threshold may not be suitable for low-quality or continuous data; it should be determined from global ROGUE values in those cases.
- ROGUE does not yet assess purity of integrated cell populations from different protocols and platforms (proposed as future work).
- The role of apCAFs in the lung-cancer microenvironment is unclear and requires further investigation, so subtype annotation does not imply functional interpretation.

## Evidence

- [methods] ROGUE-guided re-clustering yields 11 subtypes with myCAF split and apCAF: [evidence span withheld — anchor status: not_found]
- [results] myCAF cluster split with ROGUE values: [evidence span withheld — anchor status: not_found]
- [results] apCAF marker definition: [evidence span withheld — anchor status: not_found]
- [methods] ROGUE workflow functions: "matr.filter(expr, min.cells, min.genes)"
- [readme] ROGUE install from GitHub: "devtools::install_github("PaulingLiu/ROGUE")"
- [intro] entropy model selects variable genes: "we present an entropy-based model to measure the randomness of gene expression in single cells"
