---
name: gaussian-mixture-model-clustering-of-single-cell-transcriptomes
description: Use when you need to partition unsynchronized single-cell transcriptome
  data (e.g. log2(TPM+1) cycle-base-gene expression) into a user-defined number of
  clusters k before reconstructing a pseudo time-series.
when_to_use_negative:
- A single marker gene is not sufficient to give reliable information about cell cycle
  stages, so do not substitute marker-gene binning for GMM clustering of the transcriptome
- Data that is not yet filtered to cycle base genes / normalized to log2(TPM+1) —
  get_test_exp preprocessing is a prerequisite
- Analyses where differentiation state and cell cycle must be modeled jointly; the
  article notes reCAT separates cell cycle analysis from differentiation, which may
  introduce bias
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: reCAT
  role: R package providing get_test_exp and get_ordIndex, which perform the log2(TPM+1)
    filtering and GMM clustering plus consensus-TSP ordering
  repo: github.com/tinglab/reCAT
- name: mclust
  role: R package implementing the Gaussian mixture model used to cluster single cells
    according to a user-defined phase number k
- name: TSP
  role: R package underlying the consensus-TSP heuristic that orders the GMM clusters
    into a traveling salesman cycle
- name: doParallel
  role: Provides the multithreading (threadnum parameter) used to speed up get_ordIndex
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:4e9b264c65b79b9c63aa84c99222739875f3fd73913e3a85afc21016f5b6203b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - TSP
  - cluster
  - doParallel
  - mclust
  - reCAT
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Gaussian mixture model clustering of single-cell transcriptomes

## Summary

Cluster unsynchronized single-cell RNA-seq expression data into k groups with a Gaussian mixture model (via mclust) as the first stage of the reCAT pipeline, so that cell-cycle cluster ordering can subsequently be recovered with consensus-TSP.

## When to use

Use when you need to partition unsynchronized single-cell transcriptome data (e.g. log2(TPM+1) cycle-base-gene expression) into a user-defined number of clusters k before reconstructing a pseudo time-series.

## When NOT to use

- A single marker gene is not sufficient to give reliable information about cell cycle stages, so do not substitute marker-gene binning for GMM clustering of the transcriptome
- Data that is not yet filtered to cycle base genes / normalized to log2(TPM+1) — get_test_exp preprocessing is a prerequisite
- Analyses where differentiation state and cell cycle must be modeled jointly; the article notes reCAT separates cell cycle analysis from differentiation, which may introduce bias

## Inputs

- Single-cell RNA-seq expression matrix for unsynchronized cells, log2(TPM+1) values restricted to cycle base genes (test_exp, e.g. from ola_mES_2i.RData)
- User-defined phase number k for the Gaussian mixture model
- get_ordIndex parameters: threadnum (cores, e.g. 10), step_size (e.g. 2), base_cycle_range (e.g. c(6:9))

## Outputs

- ordIndex: the ordered cell time-series index from the GMM clusters plus consensus-TSP cycle
- An .RData file recording the clustering result (e.g. 'bestEnsembleComplexTSP 10 - 216 Flo .RData') usable by get_cluster_result

## How to apply

Preprocess the expression matrix so test_exp holds log2(TPM+1) values for cycle base genes only, using reCAT's get_test_exp on the raw data (e.g. ola_mES_2i.RData). Run GMM clustering through get_ordIndex, which internally uses mclust to cluster cells according to a user-defined phase number k; for the ola_mES_2i example use threadnum = 10, step_size = 2 (skipping every other k in the consensus path), and base_cycle_range = c(6:9) for the reference cycle. The function also runs the consensus-TSP heuristic on the GMM clusters to recover the cells' time-series order, returning ordIndex and writing an .RData file with the clustering result. Choose step_size up to 5 for performance; for large datasets (>1000 cells) use a bigger step_size to reduce computational time. Validate that the resulting ordering places cells with similar cycle phase adjacently, which is the prerequisite for downstream Bayes-score/mean-score computation and HMM phase segmentation (G1, S, G2/M).

## Related tools

- **reCAT** (R package providing get_test_exp and get_ordIndex, which perform the log2(TPM+1) filtering and GMM clustering plus consensus-TSP ordering) — github.com/tinglab/reCAT
- **mclust** (R package implementing the Gaussian mixture model used to cluster single cells according to a user-defined phase number k)
- **TSP** (R package underlying the consensus-TSP heuristic that orders the GMM clusters into a traveling salesman cycle)
- **doParallel** (Provides the multithreading (threadnum parameter) used to speed up get_ordIndex)

## Examples

```
source("get_ordIndex.R"); ordIndex <- get_ordIndex(test_exp, threadnum = 10, step_size = 2, base_cycle_range = c(6:9))
```

## Evaluation signals

- test_exp contains only cycle base gene values as log2(TPM+1) after get_test_exp filtering
- ordIndex is produced with the expected parameters (threadnum = 10, step_size = 2, base_cycle_range = c(6:9)) and a clustering-result .RData file is written
- Downstream Bayes-scores and mean-scores (get_score) plotted along the ordIndex (plot_bayes/plot_mean) show score peaks aligned with judged phase regions (e.g. G1 at 210-230, S at 170-190, G2/M at 40-60 in ola_mES_2i)
- get_start finds a real start with maximum HMM likelihood, confirming the ordering supports a consistent G1/S/G2/M segmentation with cls_num = 3

## Limitations

- clustering depends on a preselected gene set (378 genes) rather than data-driven gene selection; authors prefer semi-supervised selection for better future performance
- step_size is capped at a recommended maximum of 5 to preserve reCAT performance
- Computational time grows with dataset size; datasets >1000 cells need larger step_size to remain tractable

## Evidence

- [intro] Data processing: quality control, normalization, and clustering of single cells using a Gaussian mixture model (GMM) according to a user-defined phase number k: "The first step is data processing, including quality control, normalization, and clustering of single cells using the Gaussian mixture model (GMM) according to a user-defined phase number k"
- [readme] get_ordIndex orders the cells' time series with GMM and consensus-TSP: "When you preprocessing your test data, you can get its order(cell's time series) easily with ***get_ordIndex*** function."
- [readme] get_ordIndex additional parameters and defaults: "ordIndex <- get_ordIndex(test_exp, threadnum, step_size, base_cycle_range)."
- [readme] Example invocation with parameters: "ordIndex <- get_ordIndex(test_exp, threadnum = 10, step_size = 2, base_cycle_range = c(6:9) )"
- [methods] input data must be log2(TPM+1) filtered to cycle base genes: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
