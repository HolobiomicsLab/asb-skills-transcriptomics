---
name: dropout-robust-gene-expression-analysis
description: Use when a heterogeneous single-cell RNA-seq matrix shows high noise/dropout
  and you need to identify the genes that explain differences across inferred sample
  populations, downstream of SIMLR similarity learning.
when_to_use_negative:
- The analysis goal requires supervised feature selection against a known functional
  gene subset — SIMLR is unsupervised and allows only de novo discovery from the data
- Input is a large-scale dataset where the standard implementation is too slow — use
  SIMLR_Large_Scale for the clustering step instead (feature ranking operates on the
  learned similarities)
- Data are bulk RNA-seq or non-single-cell expression profiles for which the noise/dropout
  model of SIMLR was not designed
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SIMLR_Feature_Ranking
  role: scores and ranks the most important genes (features) for the similarities
    learned by SIMLR; available as SIMLR_Feature_Ranking.R and SIMLR_Feature_Ranking.m
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: SIMLR
  role: learns the multi-kernel sample-to-sample similarity and applies graph diffusion,
    providing the similarity structure that feature ranking prioritizes genes against
  repo: https://github.com/BatzoglouLabSU/SIMLR
- name: R
  role: runtime for the R implementation; requires the Matrix package for sparse matrices
    and the parallel package for kernel estimation, plus compilation of projsplx_R.c
    via R CMD SHLIB
  repo: https://bioconductor.org/packages/release/bioc/html/SIMLR.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/pmic.201700232
    title: 'SIMLR: A Tool for Large‐Scale Genomic Analyses by Multi‐Kernel Learning'
  source_content_hash: sha256:5caf9d571296c38e925798b6ab62c24191495445b19d926a87cc4839fd81932f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/BatzoglouLabSU/SIMLR
  tools:
  - R
  - SIMLR
  - SIMLR_Feature_Ranking
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# dropout-robust-gene-expression-analysis

## Summary

Rank genes by their concordance with the sample-to-sample similarities learned by SIMLR's multi-kernel framework, using graph diffusion to correct weak similarities caused by noise and dropout events in single-cell RNA-seq data. This makes feature prioritization de novo and dropout-robust rather than dependent on pre-selected gene subsets.

## When to use

Use when a heterogeneous single-cell RNA-seq matrix shows high noise/dropout and you need to identify the genes that explain differences across inferred sample populations, downstream of SIMLR similarity learning.

## When NOT to use

- The analysis goal requires supervised feature selection against a known functional gene subset — SIMLR is unsupervised and allows only de novo discovery from the data
- Input is a large-scale dataset where the standard implementation is too slow — use SIMLR_Large_Scale for the clustering step instead (feature ranking operates on the learned similarities)
- Data are bulk RNA-seq or non-single-cell expression profiles for which the noise/dropout model of SIMLR was not designed

## Inputs

- single-cell gene expression matrix (e.g. Test_1_mECS.RData example dataset)
- SIMLR-learned multi-kernel sample-to-sample similarity matrix

## Outputs

- ranked gene table (e.g. feature_ranking.csv) with one row per input gene and a numeric importance score, sorted descending
- interpretation of which genes explain differences across inferred cell populations

## How to apply

First learn the sample-to-sample similarity with SIMLR, which combines multiple kernels to fit the statistical distribution of the input scRNA-seq data and applies graph diffusion to improve weak similarity measures resulting from noise or dropout events. Then run SIMLR_Feature_Ranking, which scores each gene by its concordance with the learned multi-kernel similarity matrix. Sort genes in descending importance-score order and export the ranked table (e.g. feature_ranking.csv). Because the approach is unsupervised and does not pre-select gene subsets of known function, it supports de novo discovery from the data.

## Related tools

- **SIMLR_Feature_Ranking** (scores and ranks the most important genes (features) for the similarities learned by SIMLR; available as SIMLR_Feature_Ranking.R and SIMLR_Feature_Ranking.m) — https://github.com/BatzoglouLabSU/SIMLR
- **SIMLR** (learns the multi-kernel sample-to-sample similarity and applies graph diffusion, providing the similarity structure that feature ranking prioritizes genes against) — https://github.com/BatzoglouLabSU/SIMLR
- **R** (runtime for the R implementation; requires the Matrix package for sparse matrices and the parallel package for kernel estimation, plus compilation of projsplx_R.c via R CMD SHLIB) — https://bioconductor.org/packages/release/bioc/html/SIMLR.html

## Examples

```
Rscript SIMLR_Feature_Ranking.R  # after: R CMD SHLIB -c projsplx_R.c
```

## Evaluation signals

- feature_ranking.csv exists and contains one row per input gene with a numeric importance score
- the ranked table is strictly ordered by descending importance score
- the gene set ranked matches the gene universe of the input expression matrix (no missing/extra genes)
- the underlying SIMLR run separates known subpopulations more accurately than existing dimension reduction methods on benchmark single-cell datasets (e.g. Test_1_mECS, Test_2_Kolod, Test_3_Pollen, Test_4_Usoskin)
- graph diffusion visibly improved weak similarity measures attributable to noise or dropout events before ranking

## Limitations

- Ranking quality depends on the learned similarities; weak cell-to-cell similarities caused by dropouts must be corrected by graph diffusion, otherwise concordance scores are degraded
- The example datasets provided with the repository (e.g. Test_1_mECS.RData) are pre-processed purely as examples and should not be used in place of the data from the respective publications
- The external C code (projsplx_R.c) must be compiled; compilation issues can occur if stale pre-compiled files (projsplx_R.o / projsplx_R.so) are present

## Evidence

- [methods] graph diffusion corrects dropout-weakened similarities: "by employing graph diffusion, which improves weak similarity measures that are likely to result from noise or dropout events"
- [methods] feature ranking ranks important features for learned similarities: "*SIMLR_Feature_Ranking* to rank the most important features for the learned similarities"
- [methods] multi-kernel distance learning fits single-cell data statistics: "it learns a distance metric that best fits the structure of the data via combining multiple kernels"
