---
name: ppi-network-differentiation-activity-computation
description: Use when you have a normalized gene-by-cell scRNA-seq count matrix and
  need a per-cell pluripotency/potency score to rank or select stem and progenitor
  cells, e.g. as trajectory-inference origins.
when_to_use_negative:
- You need PPIN activity for a biological process other than differentiation — use
  the extended ORIGINS2 package or build a custom network with build_ppin() instead
- Your input is already a per-cell potency/feature table rather than a gene-by-cell
  expression matrix — the activity() function requires the raw normalized expression
  matrix
- You expect scores to follow the classical HSC-first hematopoietic hierarchy without
  validation — on GSE117498 all three methods (ORIGINS, LandSCENT, CytoTRACE) ranked
  GMP highest, not HSCs
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0128
tools:
- name: ORIGINS
  role: Primary R package whose activity() function computes the per-cell differentiation
    PPIN activity score from the expression matrix using the bundled differentiation_edges
    network
  repo: https://github.com/danielasenraoka/ORIGINS
- name: LandSCENT
  role: Benchmark pluripotency estimator run on the same expression matrix for direct
    comparison with ORIGINS scores
- name: CytoTRACE
  role: Benchmark pluripotency estimator run on the same expression matrix for direct
    comparison with ORIGINS scores
- name: remotes
  role: R package used to install ORIGINS from GitHub via remotes::install_github("danielasenraoka/ORIGINS")
- name: PyOrigins
  role: Python implementation of the same PPIN activity computation, available as
    a Jupyter-notebook-based alternative
  repo: https://github.com/danielasenraoka/PyOrigins
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:40cbecd1d3abb13812ca73dbf7b32fecd35be7ed598791f60ae7620c7a5e4f3a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - CytoTRACE
  - LandSCENT
  - ORIGINS
  - PyOrigins
  - R
  - remotes
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# ppi-network-differentiation-activity-computation

## Summary

Computes a per-cell differentiation activity score by projecting a scRNA-seq expression matrix onto a protein-protein interaction network (PPIN) associated with differentiation, using the ORIGINS R package to quantify cell pluripotency and identify stem/progenitor cells.

## When to use

Use when you have a normalized gene-by-cell scRNA-seq count matrix and need a per-cell pluripotency/potency score to rank or select stem and progenitor cells, e.g. as trajectory-inference origins.

## When NOT to use

- You need PPIN activity for a biological process other than differentiation — use the extended ORIGINS2 package or build a custom network with build_ppin() instead
- Your input is already a per-cell potency/feature table rather than a gene-by-cell expression matrix — the activity() function requires the raw normalized expression matrix
- You expect scores to follow the classical HSC-first hematopoietic hierarchy without validation — on GSE117498 all three methods (ORIGINS, LandSCENT, CytoTRACE) ranked GMP highest, not HSCs

## Inputs

- Normalized scRNA-seq expression matrix or data frame with genes in rows and cells in columns (e.g. GEO GSE117498 hematopoietic bone marrow counts formatted as a matrix)
- differentiation_edges data frame of differentiation-associated PPIN edges (shipped within the ORIGINS package)

## Outputs

- diff_activity: a per-cell differentiation activity (pluripotency) score vector
- Optionally, a per-cell-type summary table of average potency scores and rankings (e.g. hematopoietic_stemness_scores.csv) comparing ORIGINS, LandSCENT, and CytoTRACE

## How to apply

Install ORIGINS with remotes::install_github("danielasenraoka/ORIGINS") and load it with library(ORIGINS); the differentiation PPIN (differentiation_edges) ships inside the package's data directory. Prepare the expression matrix with genes in rows and cells in columns, then compute per-cell differentiation activity with diff_activity <- activity(expression_matrix, differentiation_edges). If benchmarking pluripotency estimators, run LandSCENT and CytoTRACE on the same expression matrix so scores are directly comparable, then aggregate scores by cell type (e.g. HSC, MPP, GMP) and rank types by average potency. The rationale is that ORIGINS uses the differentiation-associated PPI network as a scaffold and is more efficient than LandSCENT and requires less RAM than LandSCENT and CytoTRACE. Note that on the hematopoietic bone marrow dataset (GSE117498) none of the three methods recovered the classical HSC-first hierarchy — GMP scored highest — so validate ordering against the expected biological hierarchy rather than assuming stem cells rank first.

## Related tools

- **ORIGINS** (Primary R package whose activity() function computes the per-cell differentiation PPIN activity score from the expression matrix using the bundled differentiation_edges network) — https://github.com/danielasenraoka/ORIGINS
- **LandSCENT** (Benchmark pluripotency estimator run on the same expression matrix for direct comparison with ORIGINS scores)
- **CytoTRACE** (Benchmark pluripotency estimator run on the same expression matrix for direct comparison with ORIGINS scores)
- **remotes** (R package used to install ORIGINS from GitHub via remotes::install_github("danielasenraoka/ORIGINS"))
- **PyOrigins** (Python implementation of the same PPIN activity computation, available as a Jupyter-notebook-based alternative) — https://github.com/danielasenraoka/PyOrigins

## Examples

```
remotes::install_github("danielasenraoka/ORIGINS"); library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- activity() returns a per-cell numeric vector (diff_activity) with one score per column of the input expression matrix
- Reproducing the benchmark: on GSE117498 (Donor A) all three methods (ORIGINS, LandSCENT, CytoTRACE) should rank GMP as the highest-average-potency progenitor type, and no method should follow the classical HSC-first ordering
- Score table and ranking verdict are written to hematopoietic_stemness_scores.csv for comparison across methods
- Installation succeeds via remotes::install_github("danielasenraoka/ORIGINS") and library(ORIGINS) loads differentiation_edges without extra downloads
- ORIGINS should run with less RAM than LandSCENT and CytoTRACE on the same matrix, per the authors' benchmark

## Limitations

- On the hematopoietic bone marrow dataset (GSE117498), none of ORIGINS, LandSCENT, or CytoTRACE ordered cells according to pluripotency under the classical hematopoietic model — GMP, not HSC, had the highest average potency under all methods
- The base ORIGINS package is limited to the differentiation PPIN; other biological processes require the extended ORIGINS2 package or a custom PPIN built with build_ppin()
- ORIGINS was benchmarked only on four healthy human datasets (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383)

## Evidence

- [methods] diff_activity <- activity(expression_matrix, differentiation_edges): "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [readme] Once you have the remotes package installed you can install the origins package by typing: remotes::install_github("danielasenraoka/ORIGINS"): "Once you have the remotes package installed you can install the origins package by typing: remotes::install_github("danielasenraoka/ORIGINS")"
- [abstract] This tool was implemented as an R package named ORIGINS.: "This tool was implemented as an R package named ORIGINS."
- [readme] This package was originally created to identify stem and progenitor cells to select those cells as an origin when performing trajectory inference.: "This package was originally created to identify stem and progenitor cells to select those cells as an origin when performing trajectory inference."
- [abstract] We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020), for four healthy human data sets: "We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020), for four healthy human data sets"
