---
name: stemness-score-benchmark-verification
description: Use when you have a normalized scRNA-seq expression matrix with cell-type
  annotations and must verify that stemness/pluripotency scores (ORIGINS activity,
  LandSCENT SR, CytoTRACE) rank expected stem/progenitor types highly.
when_to_use_negative:
- Input is not a gene-by-cell expression matrix (e.g. already aggregated to a feature
  table or pseudobulk), since activity() requires each row as a gene and each column
  as a cell
- The goal is to compute PPIN activity for biological processes other than differentiation
  — use the extended ORIGINS2 package instead
- Data are not human single-cell samples; the differentiation PPIN and benchmarks
  (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383) are
  human-specific
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ORIGINS
  role: Computes the per-cell differentiation activity score from the expression matrix
    and the bundled differentiation PPIN (differentiation_edges); also evaluated over
    an HVF-derived network
  repo: https://github.com/danielasenraoka/ORIGINS
- name: LandSCENT
  role: Benchmark tool whose Signal Rank (SR) score is computed alongside ORIGINS
    activity to rank cell types by pluripotency
- name: CytoTRACE
  role: Benchmark tool whose differentiation score is computed on the same expression
    matrix and compared via mean per-cell-type ranks
- name: remotes
  role: Used to install the ORIGINS R package from GitHub
  repo: https://github.com/danielasenraoka/ORIGINS
- name: dplyr
  role: Aggregates per-cell scores into mean scores per annotated cell type for ranking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:e117c780c8186e235a55f2b1418ba0a13ceca131f278a10f701c4f54f84751da
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
yes_rate: 0.7692307692307693
n_claims: 13
---

# stemness-score-benchmark-verification

## Summary

Verifies pluripotency/stemness scoring pipelines (ORIGINS differentiation activity, LandSCENT Signal Rank, CytoTRACE) on scRNA-seq data by reproducing per-cell-type mean score rankings, including the known failure case of HVF-derived ORIGINS activity networks.

## When to use

Use when you have a normalized scRNA-seq expression matrix with cell-type annotations and must verify that stemness/pluripotency scores (ORIGINS activity, LandSCENT SR, CytoTRACE) rank expected stem/progenitor types highly.

## When NOT to use

- Input is not a gene-by-cell expression matrix (e.g. already aggregated to a feature table or pseudobulk), since activity() requires each row as a gene and each column as a cell
- The goal is to compute PPIN activity for biological processes other than differentiation — use the extended ORIGINS2 package instead
- Data are not human single-cell samples; the differentiation PPIN and benchmarks (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383) are human-specific

## Inputs

- Normalized scRNA-seq expression matrix (rows = genes, columns = cells), e.g. from GEO GSE161383 for lung cells
- Cell-type annotation vector per cell (~30 cell types for the lung sample)
- differentiation_edges: PPIN edge data frame bundled in the ORIGINS package data directory
- HVF-derived network edges for the comparator ORIGINS activity score

## Outputs

- Per-cell differentiation activity scores (ORIGINS), Signal Rank (LandSCENT), and CytoTRACE scores
- Mean score per cell type ranked under each of the four scores (SR, CytoTRACE, full-PPIN activity, HVF activity)
- Results table written to lung_stemness_scores.csv

## How to apply

Install ORIGINS in R with remotes::install_github("danielasenraoka/ORIGINS") and load it via library(ORIGINS); the differentiation PPIN (differentiation_edges) ships in the package data directory. Run diff_activity <- activity(expression_matrix, differentiation_edges) on the gene-by-cell expression matrix, then compute LandSCENT Signal Rank (SR) and CytoTRACE scores on the same matrix, plus ORIGINS activity over an HVF-derived network as a negative comparator. Aggregate mean scores per annotated cell type with dplyr and rank cell types under each score. On the lung dataset (GEO GSE161383, D122, ~30 cell types), AT2/club-like cells should have the highest mean SR and ORIGINS activity, and cells of interest should fall within the top 8 mean ranks for SR, CytoTRACE and activity. The HVF-based activity score should fail to order cell types by expected pluripotency — its failure is itself a verification signal.

## Related tools

- **ORIGINS** (Computes the per-cell differentiation activity score from the expression matrix and the bundled differentiation PPIN (differentiation_edges); also evaluated over an HVF-derived network) — https://github.com/danielasenraoka/ORIGINS
- **LandSCENT** (Benchmark tool whose Signal Rank (SR) score is computed alongside ORIGINS activity to rank cell types by pluripotency)
- **CytoTRACE** (Benchmark tool whose differentiation score is computed on the same expression matrix and compared via mean per-cell-type ranks)
- **remotes** (Used to install the ORIGINS R package from GitHub) — https://github.com/danielasenraoka/ORIGINS
- **dplyr** (Aggregates per-cell scores into mean scores per annotated cell type for ranking)

## Examples

```
diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- AT2/club-like cells obtain the highest average ORIGINS activity and LandSCENT SR among the ~30 lung cell types
- All expected stem/progenitor cell types of interest fall within the top 8 mean ranks for SR, CytoTRACE and full-PPIN ORIGINS activity
- HVF-derived ORIGINS activity fails to order cell types by expected pluripotency, reproducing the documented approximation failure
- activity() returns one score per cell (length equals number of columns of the expression matrix) when given expression_matrix and differentiation_edges
- Results table lung_stemness_scores.csv contains mean ranks for all four scores across all annotated cell types

## Limitations

- ORIGINS requires less RAM than LandSCENT and CytoTRACE, but LandSCENT remains less efficient overall; ORIGINS was shown to be more efficient than LandSCENT
- The HVF-derived network comparator fails to order cell types by expected pluripotency, so highly-variable-feature approximations are not a valid substitute for the full differentiation PPIN
- The differentiation PPIN is bundled for the differentiation process only; for other biological processes the extended ORIGINS2 package (or PyOrigins) is needed

## Evidence

- [readme] diff_activity <- activity(expression_matrix, differentiation_edges): "Compute activity of a dataset. expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell."
- [readme] The differentiation PPIN is loaded within the package in the data directory.: "The differentiation PPIN is loaded within the package in the data directory."
- [abstract] Benchmarking against LandSCENT and CytoTRACE: "We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020)"
- [abstract] ORIGINS efficiency and memory advantage: [evidence span withheld — anchor status: not_found]
- [methods] Installing ORIGINS via remotes: "remotes::install_github("danielasenraoka/ORIGINS")"
- [intro] ORIGINS objective: compute PPIN differentiation activity: "R package that provides a function to compute activity of a Protein-protein Interaction Network (PPIN) asociated with differentiation for scRNA-seq data."
