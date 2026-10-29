---
name: benchmark-tool-score-comparison
description: Use when you have an annotated scRNA-seq count matrix with a known stem/progenitor
  compartment and need to compare ORIGINS activity, LandSCENT SR and CytoTRACE scores
  across cell types to judge which tool identifies stem cells correctly.
when_to_use_negative:
- Input scRNA-seq data lacks annotated cell-type labels, so per-type score averages
  and the stem-cell ranking check cannot be computed
- The dataset has no stem/progenitor compartment to benchmark against (e.g. purely
  differentiated tissue), making the 'stem cells rank highest' acceptance check meaningless
- The goal is trajectory inference from stem-cell origins rather than comparing pluripotency
  scoring methods
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ORIGINS
  role: Computes the differentiation-activity score per cell using activity(expression_matrix,
    differentiation_edges) with the bundled differentiation PPIN; benchmarked against
    LandSCENT and CytoTRACE
  repo: https://github.com/danielasenraoka/ORIGINS
- name: LandSCENT
  role: Provides the SR pluripotency score computed on the same expression matrix
    for cross-tool comparison of per-cell-type rankings
- name: CytoTRACE
  role: Provides a third stemness score per cell to compare against ORIGINS activity
    and LandSCENT SR orderings
- name: remotes
  role: Installs the ORIGINS package from GitHub via remotes::install_github("danielasenraoka/ORIGINS")
- name: dplyr
  role: Aggregates per-cell scores into average scores per annotated cell type for
    the benchmark comparison
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:c48be158ad86210672c0330855eaf12296a1825c4ed3463e64e492a94eab2c9a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - CytoTRACE
  - LandSCENT
  - ORIGINS
  - R
  - dplyr
  - remotes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# benchmark-tool-score-comparison

## Summary

Benchmark pluripotency/stemness scoring methods (ORIGINS differentiation activity, LandSCENT SR, CytoTRACE) on annotated healthy human scRNA-seq datasets by computing per-cell scores and comparing per-cell-type averages. It exists to verify whether each tool actually ranks the known stem compartment highest.

## When to use

Use when you have an annotated scRNA-seq count matrix with a known stem/progenitor compartment and need to compare ORIGINS activity, LandSCENT SR and CytoTRACE scores across cell types to judge which tool identifies stem cells correctly.

## When NOT to use

- Input scRNA-seq data lacks annotated cell-type labels, so per-type score averages and the stem-cell ranking check cannot be computed
- The dataset has no stem/progenitor compartment to benchmark against (e.g. purely differentiated tissue), making the 'stem cells rank highest' acceptance check meaningless
- The goal is trajectory inference from stem-cell origins rather than comparing pluripotency scoring methods

## Inputs

- GEO scRNA-seq count/expression matrix (rows = genes, columns = cells) e.g. GSE113197, GSE125970, GSE117498, GSE161383
- Annotated cell-type labels per cell (e.g. basal, L1, L2; stem, TA, differentiated)
- differentiation_edges: PPIN edge data frame bundled within the ORIGINS package
- ORIGINS R package installed from GitHub (plus remotes, dplyr, magrittr)

## Outputs

- Per-cell pluripotency/stemness scores for each method (ORIGINS activity, activity HVF, LandSCENT SR, CytoTRACE)
- Average score per annotated cell type per method
- Summary CSV tables (e.g. pluripotency_scores_breast.csv, colon_stemness_scores.csv)

## How to apply

Download the annotated count matrix from GEO (GSE113197 breast, GSE125970 colon, GSE117498 hematopoietic, GSE161383 lung) and orient it as genes (rows) x cells (columns). Install ORIGINS with remotes::install_github("danielasenraoka/ORIGINS"), load it with library(ORIGINS), and compute per-cell differentiation activity with activity(expression_matrix, differentiation_edges), where differentiation_edges is the PPIN bundled in the package's data directory; optionally also compute the activity HVF variant on highly variable features. Run LandSCENT (SR score) and CytoTRACE on the same expression matrix to obtain comparison pluripotency scores. Aggregate per-cell scores into averages per annotated cell type (e.g. basal/L1/L2 for breast; stem, transit-amplifying, differentiated lineages for colon) using dplyr, compare per-type orderings across methods, and write per-cell and per-type results to CSV (e.g. pluripotency_scores_breast.csv, colon_stemness_scores.csv).

## Related tools

- **ORIGINS** (Computes the differentiation-activity score per cell using activity(expression_matrix, differentiation_edges) with the bundled differentiation PPIN; benchmarked against LandSCENT and CytoTRACE) — https://github.com/danielasenraoka/ORIGINS
- **LandSCENT** (Provides the SR pluripotency score computed on the same expression matrix for cross-tool comparison of per-cell-type rankings)
- **CytoTRACE** (Provides a third stemness score per cell to compare against ORIGINS activity and LandSCENT SR orderings)
- **remotes** (Installs the ORIGINS package from GitHub via remotes::install_github("danielasenraoka/ORIGINS"))
- **dplyr** (Aggregates per-cell scores into average scores per annotated cell type for the benchmark comparison)

## Examples

```
remotes::install_github("danielasenraoka/ORIGINS"); library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- On the colon dataset (GSE125970, Colon-2), transit-amplifying (TA) cells rank first by average score for each of the four methods, reproducing the paper's reported limitation
- On the breast dataset (GSE113197), basal cells show the highest average SR (LandSCENT), activity and activity-HVF scores, with L1 second for activity/activity-HVF
- On the breast dataset, CytoTRACE orders L1 > L2 > basal, disagreeing with the other methods — verify this expected divergence rather than assuming consensus
- Each score vector has one value per cell (column count of the expression matrix), and per-type averages are computed over all cells in each annotation
- ORIGINS runs more efficiently and with less RAM than LandSCENT and CytoTRACE on the same matrices

## Limitations

- All four stemness methods were limited in finding the stem cells in the colon sample: TA cells, not stem cells, showed the highest average score
- CytoTRACE can disagree with the ranking of the other methods (breast: L1 > L2 > basal), so per-method orderings must be reported separately rather than averaged
- ORIGINS activity depends on the differentiation PPIN bundled in the package (differentiation_edges); scores are only defined for genes covered by that network

## Evidence

- [abstract] Benchmarking scope: "We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020)"
- [abstract] Efficiency signal: "We show that our algorithm is more efficient than LandSCENT and requires less RAM memory than the other programs."
- [methods] Core activity computation: "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [supplementary] Dataset accession: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
- [readme] Package purpose: "This package was originally created to identify stem and progenitor cells to select those cells as an origin when performing trajectory inference."
