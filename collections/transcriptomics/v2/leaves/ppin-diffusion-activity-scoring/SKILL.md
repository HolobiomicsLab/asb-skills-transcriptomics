---
name: ppin-diffusion-activity-scoring
description: Use when you have a normalized genes-by-cells scRNA-seq count matrix
  and need a per-cell pluripotency/differentiation score to rank cell types (e.g.
  basal vs. L1/L2) or to root pseudotime inference in the most stem-like cells.
when_to_use_negative:
- You need PPIN activity for a biological process other than differentiation — use
  the extended ORIGINS2 package or build a custom PPIN with build_ppin() instead
- Your input is already a per-cell score/feature table rather than a raw or normalized
  gene-by-cell expression matrix — activity() requires genes as rows and cells as
  columns
- You work outside R and cannot port the matrix — consider the Python implementation
  PyOrigins rather than adapting ORIGINS call conventions incorrectly
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ORIGINS
  role: Primary R package whose activity() function computes the differentiation PPIN
    activity per cell; bundles the differentiation_edges network
  repo: https://github.com/danielasenraoka/ORIGINS
- name: remotes
  role: Used to install ORIGINS from GitHub via remotes::install_github()
- name: LandSCENT
  role: Comparison pluripotency tool (SR score) run on the same expression matrix
    to benchmark ORIGINS activity rankings
- name: CytoTRACE
  role: Comparison pluripotency tool run on the same expression matrix; known to disagree
    with ORIGINS ordering in the breast sample
- name: PyOrigins
  role: Python/Jupyter implementation of the same PPIN activity score for non-R workflows
  repo: https://github.com/danielasenraoka/PyOrigins
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:2e46466375d4cc7ae9ce2dd33272b6a2c8ee36e9fb788ac17bb97ee696b5931f
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# ppin-diffusion-activity-scoring

## Summary

Computes per-cell differentiation activity scores from a scRNA-seq expression matrix using the ORIGINS differentiation-associated Protein-protein Interaction Network (PPIN), quantifying pluripotency so that stem/progenitor cells can be ranked and used as trajectory roots.

## When to use

Use when you have a normalized genes-by-cells scRNA-seq count matrix and need a per-cell pluripotency/differentiation score to rank cell types (e.g. basal vs. L1/L2) or to root pseudotime inference in the most stem-like cells.

## When NOT to use

- You need PPIN activity for a biological process other than differentiation — use the extended ORIGINS2 package or build a custom PPIN with build_ppin() instead
- Your input is already a per-cell score/feature table rather than a raw or normalized gene-by-cell expression matrix — activity() requires genes as rows and cells as columns
- You work outside R and cannot port the matrix — consider the Python implementation PyOrigins rather than adapting ORIGINS call conventions incorrectly

## Inputs

- Normalized scRNA-seq expression matrix (rows = genes, columns = cells), e.g. GEO GSE113197 breast epithelial counts
- Differentiation PPIN edge data frame differentiation_edges (bundled with the ORIGINS package in its data directory)
- Optional per-cell cell-type annotations (basal, L1, L2) for averaging and ranking scores

## Outputs

- Per-cell differentiation activity score vector diff_activity
- Cell-type-averaged pluripotency comparison table (e.g. pluripotency_scores_breast.csv) across ORIGINS activity, activity-HVF, LandSCENT SR and CytoTRACE
- Optional Slingshot root-cell selection (highest-activity cells within the basal cluster) for trajectory inference

## How to apply

Install ORIGINS from GitHub with remotes::install_github("danielasenraoka/ORIGINS") and load it with library(ORIGINS); the differentiation PPIN ships inside the package as the data frame differentiation_edges, so no external network file is needed. Orient the input as an expression_matrix with one row per gene and one column per cell (breast data are GEO GSE113197), then call diff_activity <- activity(expression_matrix, differentiation_edges) to obtain a score per cell normalized by edge count and mean expression. Optionally compute the activity-HVF variant restricted to highly variable features, and benchmark against LandSCENT (SR score) and CytoTRACE on the same matrix. Average the scores within annotated cell types (basal, L1, L2) and verify ordering: in the breast sample, basal ranks highest for SR, activity and activity-HVF, whereas CytoTRACE orders L1 > L2 > basal. For trajectory work, select the basal-cluster cells with the highest differentiation activity as the Slingshot root, which should yield a bifurcating trajectory into L1 and L2 through a shared intermediate state.

## Related tools

- **ORIGINS** (Primary R package whose activity() function computes the differentiation PPIN activity per cell; bundles the differentiation_edges network) — https://github.com/danielasenraoka/ORIGINS
- **remotes** (Used to install ORIGINS from GitHub via remotes::install_github())
- **LandSCENT** (Comparison pluripotency tool (SR score) run on the same expression matrix to benchmark ORIGINS activity rankings)
- **CytoTRACE** (Comparison pluripotency tool run on the same expression matrix; known to disagree with ORIGINS ordering in the breast sample)
- **PyOrigins** (Python/Jupyter implementation of the same PPIN activity score for non-R workflows) — https://github.com/danielasenraoka/PyOrigins

## Examples

```
remotes::install_github("danielasenraoka/ORIGINS"); library(ORIGINS); diff_activity <- activity(expression_matrix, differentiation_edges)
```

## Evaluation signals

- Output vector has one score per cell (length = number of columns of expression_matrix) with no NA values
- Cell-type-averaged scores reproduce the reported breast ordering: basal highest for SR, activity and activity-HVF, with L1 second for activity/HVF; CytoTRACE instead orders L1 > L2 > basal
- Rooting Slingshot at the highest-activity basal cells yields two lineages (basal→L1 and basal→L2) through a shared intermediate state
- Activity values are normalized (by PPIN edge count and per-cell mean expression), enabling cross-sample and cross-PPIN comparisons as documented for the score formula ACT_i = (1/(e*mu_i)) * sum A_jk Y_ij Y_ik
- The same expression matrix fed to LandSCENT/CytoTRACE produces scores that can be written to a single summary table (pluripotency_scores_breast.csv) for side-by-side inspection

## Limitations

- The bundled differentiation_edges PPIN covers only the differentiation process; PPINs for other processes require ORIGINS2 or PyOrigins
- ORIGINS was benchmarked on four healthy human datasets (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383); behavior on diseased or non-human data is not established here
- Pluripotency methods can disagree: in the breast sample CytoTRACE's ordering (L1 > L2 > basal) contradicted ORIGINS and LandSCENT, so a single method should not be treated as definitive
- ORIGINS is more memory-efficient than LandSCENT and CytoTRACE but was not claimed to be more accurate on all inputs

## Evidence

- [readme] expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell: "expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell."
- [readme] diff_activity <- activity(expression_matrix, differentiation_edges): "diff_activity <- activity(expression_matrix, differentiation_edges)"
- [methods] Once you have the remotes package installed you can install the origins package by typing: remotes::install_github("danielasenraoka/ORIGINS"): "Once you have the remotes package installed you can install the origins package by typing: remotes::install_github("danielasenraoka/ORIGINS")"
- [intro] R package that provides a function to compute activity of a Protein-protein Interaction Network (PPIN) asociated with differentiation for scRNA-seq data.: "R package that provides a function to compute activity of a Protein-protein Interaction Network (PPIN) asociated with differentiation for scRNA-seq data."
- [supplementary] available on the GEO database under the accession codes: GSE113197 for breast epithelial cells: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
