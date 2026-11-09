---
name: scrna-seq-stemness-scoring
description: Use when a gene-by-cell scRNA-seq expression matrix must be ranked by
  pluripotency/stemness to identify stem or progenitor cells, e.g. to pick trajectory-inference
  origins or to check whether a potency method recovers a known cellular hierarchy
  (HSC > MPP > GMP, stem vs TA).
when_to_use_negative:
- When the goal is PPIN activity for a biological process other than differentiation
  — use the extended ORIGINS2 package (build_ppin with full_human_ppin) or PyOrigins
  instead, since base ORIGINS ships only the differentiation PPIN.
- When only a pre-computed feature table or per-cell annotation is available with
  no gene-by-cell expression matrix — activity(expression_matrix, differentiation_edges)
  requires the expression matrix itself.
- 'When the biological question is whether a stemness method correctly recovers the
  classical hierarchy (e.g. HSC-first ordering): the paper''s findings show all methods
  failed this on the colon and hematopoietic benchmarks (TA cells and GMP scored highest),
  so results should not be assumed to equal true stem-cell identity.'
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ORIGINS
  role: 'Primary R tool: computes the differentiation-activity pluripotency score
    per cell via activity(expression_matrix, differentiation_edges) using the shipped
    differentiation PPIN'
  repo: https://github.com/danielasenraoka/ORIGINS
- name: LandSCENT
  role: Benchmark stemness-scoring method run on the same expression matrix for comparison
    with ORIGINS
- name: CytoTRACE
  role: Benchmark potency-scoring method run on the same expression matrix for comparison
    with ORIGINS
- name: remotes
  role: 'R package used to install ORIGINS from GitHub: remotes::install_github("danielasenraoka/ORIGINS")'
- name: dplyr
  role: Used to aggregate per-cell scores into average scores per annotated cell type
- name: ORIGINS2
  role: Extended version computing PPIN activity for any biological process with custom
    PPIN construction via build_ppin(genes_bp, ppi_edges_complete = full_human_ppin)
  repo: https://github.com/danielasenraoka/ORIGINS2
- name: PyOrigins
  role: Python implementation of the PPIN activity score for interactive exploration
    in a Jupyter notebook
  repo: https://github.com/danielasenraoka/PyOrigins
provenance:
  source_task_ids:
  - task_003
  - task_004
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:11cee39a932e7cb7955b821a23808b043a2834361dba73eae31b49973dda8171
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
  - dplyr
  - remotes
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# scRNA-seq-stemness-scoring

## Summary

Compute per-cell pluripotency/stemness scores from scRNA-seq count matrices using the ORIGINS differentiation-activity function (activity(expression_matrix, differentiation_edges)) and benchmark them against LandSCENT and CytoTRACE, as done in the Senra et al. 2022 MethodsX workflow on breast (GSE113197), colon (GSE125970), hematopoietic (GSE117498) and lung (GSE161383) datasets.

## When to use

Use when a gene-by-cell scRNA-seq expression matrix must be ranked by pluripotency/stemness to identify stem or progenitor cells, e.g. to pick trajectory-inference origins or to check whether a potency method recovers a known cellular hierarchy (HSC > MPP > GMP, stem vs TA).

## When NOT to use

- When the goal is PPIN activity for a biological process other than differentiation — use the extended ORIGINS2 package (build_ppin with full_human_ppin) or PyOrigins instead, since base ORIGINS ships only the differentiation PPIN.
- When only a pre-computed feature table or per-cell annotation is available with no gene-by-cell expression matrix — activity(expression_matrix, differentiation_edges) requires the expression matrix itself.
- When the biological question is whether a stemness method correctly recovers the classical hierarchy (e.g. HSC-first ordering): the paper's findings show all methods failed this on the colon and hematopoietic benchmarks (TA cells and GMP scored highest), so results should not be assumed to equal true stem-cell identity.

## Inputs

- scRNA-seq expression/count matrix (genes = rows, cells = columns; normalized expression expected per ORIGINS2 README)
- differentiation PPIN edge list (differentiation_edges) shipped inside the ORIGINS package data directory
- per-cell type annotations (e.g. stem cells, TA cells, HSC, MPP, GMP) for score aggregation
- GEO count matrices, e.g. GSE113197 (breast), GSE125970 (colon), GSE117498 (hematopoietic bone marrow), GSE161383 (lung)

## Outputs

- Per-cell differentiation activity / stemness score vector (one value per cell) for each method (ORIGINS, LandSCENT, CytoTRACE, ORIGINS activity HVF)
- Per-cell-type average score table with method rankings (e.g. colon_stemness_scores.csv, hematopoietic_stemness_scores.csv)
- Selected origin cells (stem/progenitor candidates) for downstream trajectory inference

## How to apply

Install the R package with remotes::install_github("danielasenraoka/ORIGINS") (remotes installed via install.packages("remotes")), then library(ORIGINS). Format the input as an expression matrix with genes in rows and cells in columns (normalized expression is expected by the ORIGINS2/PyOrigins description). Call diff_activity <- activity(expression_matrix, differentiation_edges); differentiation_edges is the differentiation PPIN shipped inside the package data directory, and in ORIGINS2 an optional logical 'normalization' parameter controls whether activity is divided by the number of PPIN edges. For comparability, run LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020) on the same matrix, then aggregate per-cell scores by annotated cell type (e.g. stem, transit-amplifying, HSC, MPP, GMP) with dplyr and rank types by average score. ORIGINS is claimed to be more efficient than LandSCENT and needs less RAM than LandSCENT and CytoTRACE; the ORIGINS2 extension supports custom PPINs via build_ppin(genes_bp, ppi_edges_complete = full_human_ppin) and PyOrigins provides a Python implementation of the normalized activity score ACT_i = (1/(e*mu_i)) * sum A_jk Y_ij Y_ik.

## Related tools

- **ORIGINS** (Primary R tool: computes the differentiation-activity pluripotency score per cell via activity(expression_matrix, differentiation_edges) using the shipped differentiation PPIN) — https://github.com/danielasenraoka/ORIGINS
- **LandSCENT** (Benchmark stemness-scoring method run on the same expression matrix for comparison with ORIGINS)
- **CytoTRACE** (Benchmark potency-scoring method run on the same expression matrix for comparison with ORIGINS)
- **remotes** (R package used to install ORIGINS from GitHub: remotes::install_github("danielasenraoka/ORIGINS"))
- **dplyr** (Used to aggregate per-cell scores into average scores per annotated cell type)
- **ORIGINS2** (Extended version computing PPIN activity for any biological process with custom PPIN construction via build_ppin(genes_bp, ppi_edges_complete = full_human_ppin)) — https://github.com/danielasenraoka/ORIGINS2
- **PyOrigins** (Python implementation of the PPIN activity score for interactive exploration in a Jupyter notebook) — https://github.com/danielasenraoka/PyOrigins

## Examples

```
install.packages("remotes"); remotes::install_github("danielasenraoka/ORIGINS"); library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- The activity() call returns a per-cell score vector whose length equals the number of columns (cells) of the expression matrix.
- For the colon benchmark (GSE125970, Colon-2), the acceptance check is that transit-amplifying (TA) cells rank first by average score for each of the four methods (LandSCENT, CytoTRACE, ORIGINS activity, ORIGINS activity HVF).
- For the hematopoietic benchmark (GSE117498, Donor A), GMP must have the highest average potency under all methods, and no method's ordering should follow the classical HSC-first hierarchy.
- Per-cell and per-type score tables are written out (e.g. colon_stemness_scores.csv / hematopoietic_stemness_scores.csv) and contain one ranked row per annotated cell type per method.
- Cross-method sanity: all methods should produce scores on the same cells so averages and rankings are directly comparable; disagreements in efficiency/RAM favoring ORIGINS over LandSCENT/CytoTRACE should be observable on large matrices.

## Limitations

- On the colon dataset all four methods identified TA cells, not stem cells, as highest-scoring; on the hematopoietic dataset all methods placed GMP above HSCs and none ordered cells according to the classical hierarchy — so high score does not guarantee true stem identity.
- Base ORIGINS is tied to the differentiation PPIN shipped in the package; other biological processes require ORIGINS2 or a custom build_ppin call.
- ORIGINS2/PyOrigins expect a normalized expression matrix and the activity is normalized by the number of PPIN edges (e) and per-cell mean expression (mu_i) to allow comparisons across samples and PPINs; using unnormalized input or skipping the normalization flag may distort cross-dataset comparisons.

## Evidence

- [abstract] ORIGINS quantifies pluripotency from scRNA-seq data using a differentiation-associated PPI network and the expression matrix: "This approach uses the protein-protein interaction (PPI) network associated with the differentiation process as a scaffold and the gene expression matrix to calculate a score"
- [abstract] ORIGINS benchmarked against LandSCENT and CytoTRACE on four healthy human datasets: "We benchmark the performance of our algorithm with two previously published tools, LandSCENT (Chen et al., 2019) and CytoTRACE (Gulati et al., 2020), for four healthy human data sets: breast, colon,"
- [methods] Installation of ORIGINS via remotes from GitHub: "Once you have the remotes package installed you can install the origins package by typing: remotes::install_github("danielasenraoka/ORIGINS")"
- [methods] Core activity call with expression matrix and shipped differentiation PPIN: "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [readme] README: expression matrix orientation and differentiation_edges shipped in package data: "expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell. differentiation_edges is a data frame containing the edges of the Protein-protein"
- [readme] ORIGINS2 normalization parameter controlling edge-count normalization: [evidence span withheld — anchor status: not_found]
- [supplementary] GEO accession codes for the four benchmark datasets: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
- [intro] Original purpose: selecting origin cells for trajectory inference: "This package was originally created to identify stem and progenitor cells to select those cells as an origin when performing trajectory inference."
