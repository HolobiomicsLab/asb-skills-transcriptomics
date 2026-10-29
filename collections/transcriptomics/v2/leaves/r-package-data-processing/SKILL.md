---
name: r-package-data-processing
description: Use when you have a genes-by-cells normalized count matrix from human
  scRNA-seq data and need a per-cell pluripotency/differentiation-activity score to
  rank cell types or select stem/progenitor cells as trajectory-inference origins.
when_to_use_negative:
- Input is not a gene-by-cell expression matrix (e.g. already-aggregated feature tables,
  bulk RNA-seq, or unnormalized raw counts without the expected orientation)
- You need PPIN activity for a biological process other than differentiation using
  the original ORIGINS package — use ORIGINS2 or PyOrigins for cell cycle or custom
  BP PPINs
- The goal is trajectory inference itself rather than selecting origin cells; ORIGINS
  was created to pick stem/progenitor cells as origins, not to build the trajectory
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ORIGINS
  role: Core R package; its activity() function computes the differentiation PPIN
    activity score per cell
  repo: https://github.com/danielasenraoka/ORIGINS
- name: remotes
  role: Required to install ORIGINS from GitHub via remotes::install_github()
- name: LandSCENT
  role: Benchmark potency-score method run on the same expression matrix for cross-method
    comparison
- name: CytoTRACE
  role: Benchmark potency-score method run on the same expression matrix for cross-method
    comparison
- name: ORIGINS2
  role: Extended version computing PPIN activity for 18 biological processes and custom
    PPINs
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: PyOrigins
  role: Python implementation of the PPIN activity score
  repo: https://github.com/danielasenraoka/PyOrigins
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:9c5f096ab73e3beee9a0aeb981f464b7fe838f1d61f47a538632d450193f73ad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - CytoTRACE
  - LandSCENT
  - ORIGINS
  - ORIGINS2
  - PyOrigins
  - R
  - remotes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# r-package-data-processing

## Summary

Install and drive the ORIGINS R package (Senra et al., MethodsX 2022) to compute a differentiation-activity pluripotency score per cell from a scRNA-seq expression matrix, using the differentiation PPIN edges shipped inside the package.

## When to use

Use when you have a genes-by-cells normalized count matrix from human scRNA-seq data and need a per-cell pluripotency/differentiation-activity score to rank cell types or select stem/progenitor cells as trajectory-inference origins.

## When NOT to use

- Input is not a gene-by-cell expression matrix (e.g. already-aggregated feature tables, bulk RNA-seq, or unnormalized raw counts without the expected orientation)
- You need PPIN activity for a biological process other than differentiation using the original ORIGINS package — use ORIGINS2 or PyOrigins for cell cycle or custom BP PPINs
- The goal is trajectory inference itself rather than selecting origin cells; ORIGINS was created to pick stem/progenitor cells as origins, not to build the trajectory

## Inputs

- Normalized scRNA-seq expression matrix or data frame (genes in rows, cells in columns), e.g. GSE117498 hematopoietic bone marrow counts
- differentiation_edges data frame (differentiation PPIN edges, bundled inside the ORIGINS package data directory)

## Outputs

- diff_activity: a per-cell differentiation activity (pluripotency) score vector
- Aggregated score/ranking table per cell type, e.g. hematopoietic_stemness_scores.csv

## How to apply

Install the package with remotes::install_github("danielasenraoka/ORIGINS") after installing the remotes dependency, then load it with library(ORIGINS). Format the input as an expression matrix (matrix or data frame) with genes in rows and cells in columns; the differentiation PPIN edges are already loaded within the package as the data frame differentiation_edges. Call diff_activity <- activity(expression_matrix, differentiation_edges) to obtain a per-cell activity score that uses the PPI network as a scaffold and the expression matrix to compute the score. To compare against published benchmarks, run LandSCENT and CytoTRACE on the same expression matrix, aggregate scores by cell type (e.g. HSC, MPP, GMP), and rank types by average potency; note ORIGINS is reported to be more efficient than LandSCENT and requires less RAM than both alternatives. An extended version, ORIGINS2, computes PPIN activity for 18 biological processes and supports custom PPINs via build_ppin(genes_bp, ppi_edges_complete = full_human_ppin), and a Python implementation (PyOrigins) also exists.

## Related tools

- **ORIGINS** (Core R package; its activity() function computes the differentiation PPIN activity score per cell) — https://github.com/danielasenraoka/ORIGINS
- **remotes** (Required to install ORIGINS from GitHub via remotes::install_github())
- **LandSCENT** (Benchmark potency-score method run on the same expression matrix for cross-method comparison)
- **CytoTRACE** (Benchmark potency-score method run on the same expression matrix for cross-method comparison)
- **ORIGINS2** (Extended version computing PPIN activity for 18 biological processes and custom PPINs) — https://github.com/danielasenraoka/ORIGINS2
- **PyOrigins** (Python implementation of the PPIN activity score) — https://github.com/danielasenraoka/PyOrigins

## Examples

```
library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- activity() runs without error on the expression matrix and returns one activity value per cell (same length as the number of columns)
- Per-cell-type averages are computable and comparable across ORIGINS, LandSCENT, and CytoTRACE on the identical expression matrix
- Ranking verdict recorded: on the GSE117498 hematopoietic dataset, GMP had the highest average potency under all methods, and no method followed the classical HSC-first hierarchy — deviations from this expected pattern indicate a processing error
- Score table and ranking verdict written to the output CSV (e.g. hematopoietic_stemness_scores.csv) with all four methods' scores aggregated by progenitor type
- Package loads successfully from the documented GitHub source and the differentiation_edges data frame is present in the data directory

## Limitations

- Benchmarked on four healthy human datasets (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383); performance outside these tissues is not established
- On the hematopoietic dataset, ORIGINS, LandSCENT, and CytoTRACE all placed GMP above HSC, i.e. the scores did not recover the classical hematopoietic pluripotency hierarchy
- The original ORIGINS package only carries the differentiation PPIN; other biological processes require ORIGINS2 or custom PPIN construction

## Evidence

- [methods] activity(expression_matrix, differentiation_edges): "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [readme] Install via remotes from GitHub: "remotes::install_github("danielasenraoka/ORIGINS")"
- [readme] Input orientation: genes in rows, cells in columns: "expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell"
- [abstract] Benchmarking against LandSCENT and CytoTRACE on four human datasets: "We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020)"
- [intro] Purpose: identify stem and progenitor cells as trajectory origins: "This package was originally created to identify stem and progenitor cells to select those cells as an origin when performing trajectory inference."
