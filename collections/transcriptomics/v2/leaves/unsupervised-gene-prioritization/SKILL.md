---
name: unsupervised-gene-prioritization
description: Use when single-cell RNA-seq expression data has been clustered or embedded
  with SIMLR and you need to identify the genes that explain differences across the
  inferred sample populations without pre-selecting gene subsets of known function.
when_to_use_negative:
- The analysis goal is supervised feature selection from a pre-selected gene subset
  of known function — SIMLR_Feature_Ranking is unsupervised and de novo by design
- No SIMLR similarity has been learned yet — the ranking depends on the multi-kernel
  sample-to-sample similarities, so run SIMLR first
- The dataset is too large for the standard implementation and should instead use
  SIMLR_Large_Scale for the similarity learning step
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: SIMLR_Feature_Ranking
  role: Ranks the most important features (genes) for the similarities learned by
    SIMLR; provided as SIMLR_Feature_Ranking.R (R) and SIMLR_Feature_Ranking.m (MATLAB)
  repo: github.com/BatzoglouLabSU/SIMLR
- name: SIMLR
  role: Learns the sample-to-sample similarity measure via multi-kernel learning and
    graph diffusion that feature ranking operates on
  repo: github.com/BatzoglouLabSU/SIMLR
- name: R
  role: Runtime for the SIMLR R implementation; requires the Matrix package (sparse
    matrices) and parallel package (parallel kernel estimation), installed via install_R_libraries.R
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201700232
    title: 'SIMLR: A Tool for Large‐Scale Genomic Analyses by Multi‐Kernel Learning'
  source_content_hash: sha256:eaf5d17858f8c8f18dee9780e2a00b9debee3ed57238ad2b5ffb8ff7848aafd8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - R
  - SIMLR
  - SIMLR_Feature_Ranking
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Unsupervised gene prioritization via SIMLR_Feature_Ranking

## Summary

Rank genes by their concordance with the sample-to-sample similarity learned by SIMLR, using SIMLR_Feature_Ranking with graph diffusion to correct weak similarities caused by noise and dropout events. This offers a de novo, unsupervised downstream application of the learned similarities alongside dimension reduction and visualization.

## When to use

Use when single-cell RNA-seq expression data has been clustered or embedded with SIMLR and you need to identify the genes that explain differences across the inferred sample populations without pre-selecting gene subsets of known function.

## When NOT to use

- The analysis goal is supervised feature selection from a pre-selected gene subset of known function — SIMLR_Feature_Ranking is unsupervised and de novo by design
- No SIMLR similarity has been learned yet — the ranking depends on the multi-kernel sample-to-sample similarities, so run SIMLR first
- The dataset is too large for the standard implementation and should instead use SIMLR_Large_Scale for the similarity learning step

## Inputs

- Single-cell RNA-seq gene-by-cell expression matrix loaded from .RData (e.g. Test_1_mECS.RData)
- Sample-to-sample similarity learned by SIMLR via multi-kernel learning

## Outputs

- Ranked gene table (e.g. feature_ranking.csv) with one importance score per gene, ordered descending
- Set of most important features (genes) for the learned similarities

## How to apply

Load the single-cell expression matrix (e.g. Test_1_mECS.RData) in R after installing the Matrix and parallel packages via install_R_libraries.R, and ensure the SIMLR similarity has been learned by combining multiple kernels. Run the SIMLR_Feature_Ranking.R script from the SIMLR package (GitHub BatzoglouLabSU/SIMLR or Bioconductor), which scores each gene by its concordance with the multi-kernel sample-to-sample similarity and applies graph diffusion to improve weak similarity measures likely to result from noise or dropout events. Sort genes in descending order of importance score and export the ranked table (e.g. feature_ranking.csv). Because SIMLR is unsupervised, ranking proceeds de novo from the data rather than from a curated gene list; the ranking is a downstream application of the learned similarities, not a rerun of the clustering.

## Related tools

- **SIMLR_Feature_Ranking** (Ranks the most important features (genes) for the similarities learned by SIMLR; provided as SIMLR_Feature_Ranking.R (R) and SIMLR_Feature_Ranking.m (MATLAB)) — github.com/BatzoglouLabSU/SIMLR
- **SIMLR** (Learns the sample-to-sample similarity measure via multi-kernel learning and graph diffusion that feature ranking operates on) — github.com/BatzoglouLabSU/SIMLR
- **R** (Runtime for the SIMLR R implementation; requires the Matrix package (sparse matrices) and parallel package (parallel kernel estimation), installed via install_R_libraries.R)

## Examples

```
Rscript SIMLR_Feature_Ranking.R  # after loading Test_1_mECS.RData; then sort genes by descending importance score and write feature_ranking.csv
```

## Evaluation signals

- feature_ranking.csv exists and contains one row per input gene
- Each gene row carries a numeric importance score (no missing or non-numeric entries)
- Rows are ordered by descending importance score
- Top-ranked genes are biologically consistent with the subpopulations SIMLR separated in the clustering step
- The underlying SIMLR similarity used graph diffusion, so weak similarities from dropout events were corrected before ranking

## Limitations

- Weak cell-to-cell similarities caused by high levels of noise, outliers and dropouts in single-cell data can distort gene ranking if graph diffusion is not applied
- Example datasets (e.g. Test_1_mECS.RData) are provided purely as examples after pre-processing and should not be used in place of the ones from the respective publications
- Running the R implementation requires compiling the external C code projsplx_R.c via R CMD SHLIB -c projsplx_R.c; compilation issues may require removing pre-compiled projsplx_R.o and projsplx_R.so files

## Evidence
