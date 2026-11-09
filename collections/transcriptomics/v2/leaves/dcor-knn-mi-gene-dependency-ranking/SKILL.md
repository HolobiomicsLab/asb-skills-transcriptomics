---
name: dcor-knn-mi-gene-dependency-ranking
description: Use when a pseudo-time-ordered single-cell expression matrix is available
  and you must rank genes by nonlinear dependency (dCor, KNN-MI) to recover cycle-related
  genes absent from Cyclebase.
when_to_use_negative:
- The cells are synchronized or their true time ordering is already known — reCAT's
  TSP ordering and HMM segmentation are designed for unsynchronized scRNA-seq data.
- The goal is differentiation or multi-process modeling — reCAT separates cell cycle
  analysis from differentiation, which the authors note may introduce bias, and a
  combined multi-process model is still lacking.
- You need a fully data-driven gene set rather than ranking within the preselected
  378 cell-cycle base genes; the authors prefer semi-supervised selection of cell
  cycle genes from the data.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: reCAT
  role: Supplies the pipeline (get_test_exp, get_ordIndex, get_score, get_hmm_order,
    get_start) that recovers the ordered pseudo time-series and Kalman-smoothed expression
    profiles on which dCor and KNN-MI are computed.
  repo: github.com/tinglab/reCAT
- name: R
  role: Implementation language for reCAT and the dCor / KNN-MI ranking scripts.
- name: TSP
  role: Provides the traveling-salesman solver used by consensus-TSP inside get_ordIndex
    to recover cell order.
- name: doParallel
  role: Parallel backend for threadnum-based get_ordIndex and get_start execution.
- name: mclust
  role: Gaussian mixture model clustering of single cells during reCAT data processing.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:0a13e5de7737c7e46d33581b7de5f70fd41347f6fd02150fff7b938f449a93d3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - TSP
  - doParallel
  - mclust
  - reCAT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# dcor-knn-mi-gene-dependency-ranking

## Summary

Ranks every gene along reCAT's Kalman-smoothed cell-cycle pseudo time-series by distance correlation (dCor) and KNN-based mutual information (KNN-MI) to surface cell-cycle genes missed by curated lists like Cyclebase (e.g., Ncapd2 in young-MPP mHSC cells).

## When to use

Use when a pseudo-time-ordered single-cell expression matrix is available and you must rank genes by nonlinear dependency (dCor, KNN-MI) to recover cycle-related genes absent from Cyclebase.

## When NOT to use

- The cells are synchronized or their true time ordering is already known — reCAT's TSP ordering and HMM segmentation are designed for unsynchronized scRNA-seq data.
- The goal is differentiation or multi-process modeling — reCAT separates cell cycle analysis from differentiation, which the authors note may introduce bias, and a combined multi-process model is still lacking.
- You need a fully data-driven gene set rather than ranking within the preselected 378 cell-cycle base genes; the authors prefer semi-supervised selection of cell cycle genes from the data.

## Inputs

- youngMpp single-cell expression matrix (mHSC cells, TPM values)
- log2(TPM+1) matrix restricted to cell-cycle base genes produced by get_test_exp
- recovered cell time-series order (ordIndex)
- Bayes-scores and mean-scores from get_score
- class-region table (rdata, e.g. youngMpp_region) marking G1, S and G2/M positions
- recovered cycle start point from get_start

## Outputs

- Kalman-smoothed gene-expression profiles along the cell-cycle pseudo time-series
- Per-gene dependency rankings by distance correlation (dCor) and KNN-based mutual information (KNN-MI)
- candidate_genes.csv with top-five dCor and top-five KNN-MI candidate cell-cycle gene lists (including Ncapd2)

## How to apply

Preprocess the youngMpp expression matrix to log2(TPM+1) restricted to cell-cycle base genes with reCAT's get_test_exp. Recover the cells' time-series order with get_ordIndex (threadnum = 10, step_size = 2, base_cycle_range = c(6:9)), then compute Bayes-scores and mean-scores with get_score. Segment the series into G1, S, G2/M (cls_num = 3) with get_hmm_order, supplying the class regions (rdata from youngMpp_region) and the recovered start (get_start), so the Kalman smoother can estimate the underlying smoothed gene-expression profiles. Compute dCor and KNN-MI significance of each gene against the smoothed pseudo time-series and rank all genes. Write the top-five dCor and KNN-MI candidate lists to candidate_genes.csv, confirming that Ncapd2 — highly expressed at S and G2 stages, GO-annotated as cell cycle-related but absent from Cyclebase — is recovered.

## Related tools

- **reCAT** (Supplies the pipeline (get_test_exp, get_ordIndex, get_score, get_hmm_order, get_start) that recovers the ordered pseudo time-series and Kalman-smoothed expression profiles on which dCor and KNN-MI are computed.) — github.com/tinglab/reCAT
- **R** (Implementation language for reCAT and the dCor / KNN-MI ranking scripts.)
- **TSP** (Provides the traveling-salesman solver used by consensus-TSP inside get_ordIndex to recover cell order.)
- **doParallel** (Parallel backend for threadnum-based get_ordIndex and get_start execution.)
- **mclust** (Gaussian mixture model clustering of single cells during reCAT data processing.)

## Examples

```
source('get_ordIndex.R'); ordIndex <- get_ordIndex(test_exp, threadnum = 10, step_size = 2, base_cycle_range = c(6:9))
```

## Evaluation signals

- candidate_genes.csv exists and contains both top-five dCor and top-five KNN-MI candidate lists.
- The known cycle-related gene Ncapd2 appears in the top-five dCor and KNN-MI candidate lists.
- Ncapd2's smoothed profile peaks at the S and G2 stages of the recovered pseudo time-series.
- Bayes-score and mean-score plots along ordIndex show the expected two major expression waves corresponding to the G1 and G2 checkpoints.
- HMM segmentation with cls_num = 3 assigns contiguous G1, S and G2/M blocks consistent with the rdata class regions.

## Limitations

- The method relies on a preselected gene set (378 cell-cycle base genes) rather than data-driven selection; the authors would prefer semi-supervised selection for better performance.
- Bayes-scores and mean-scores are noisy and need improvement to be more informative, which propagates uncertainty into HMM phase segmentation.
- reCAT separates cell cycle analysis from differentiation, which may introduce bias; no combined multi-process model is available.
- get_ordIndex step_size should be at most 5 (default 2); larger step sizes are recommended only for datasets with more than 1000 cells to reduce compute time.

## Evidence

- [readme] in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
- [intro] segment the time-series into G0, G1, S and G2/M, and a Kalman smoother to estimate the underlying gene expression levels: "segment the time-series into G0, G1, S and G2/M, and a Kalman smoother to estimate the underlying gene expression levels"
- [discussion] Instead of the preselected gene set (378 genes), we would prefer semi-supervised selection of cell cycle genes from the data: "Instead of the preselected gene set (378 genes), we would prefer semi-supervised selection of cell cycle genes from the data"
- [methods] you can get its order(cell's time series) easily with ***get_ordIndex*** function: "you can get its order(cell's time series) easily with ***get_ordIndex*** function"
