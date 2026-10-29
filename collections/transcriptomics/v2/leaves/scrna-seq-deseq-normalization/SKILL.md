---
name: scrna-seq-deseq-normalization
description: Use when raw scRNA-seq count matrices (e.g., mESC-SMARTer / ola_mES_2i.RData)
  must be normalized and log-transformed to log2(TPM+1) before reCAT clustering, scoring,
  or marker-gene distribution plots.
when_to_use_negative:
- Input is already a reCAT test_exp matrix that is log2(TPM+1) normalized — re-normalizing
  would distort values.
- The goal is stage scoring/ordering (get_ordIndex, get_score, get_hmm_order) on already-preprocessed
  data rather than producing the normalized matrix itself.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: reCAT
  role: R package providing get_test_exp to standardize the log2(TPM+1) input and
    filter to cell cycle base genes
  repo: github.com/tinglab/reCAT
- name: R
  role: implementation environment in which reCAT and the normalization workflow run
- name: ggplot2
  role: used downstream to draw violin plots of normalized marker-gene expression
    split by cell cycle stage
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-017-00039-z
    title: Reconstructing cell cycle pseudo time-series via single-cell transcriptome
      data
  source_content_hash: sha256:bbd47a037da3125d69fdb5c81ce630f65d6b219508a01b74901c2756ed1a268d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - ggplot2
  - reCAT
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# scrna-seq-deseq-normalization

## Summary

Normalize single-cell RNA-seq count matrices with DESeq size-factor normalization and the reCAT log2(TPM+1) convention before cell cycle analyses such as marker-gene violin plots, clustering, and HMM-based cell cycle staging.

## When to use

Use when raw scRNA-seq count matrices (e.g., mESC-SMARTer / ola_mES_2i.RData) must be normalized and log-transformed to log2(TPM+1) before reCAT clustering, scoring, or marker-gene distribution plots.

## When NOT to use

- Input is already a reCAT test_exp matrix that is log2(TPM+1) normalized — re-normalizing would distort values.
- The goal is stage scoring/ordering (get_ordIndex, get_score, get_hmm_order) on already-preprocessed data rather than producing the normalized matrix itself.

## Inputs

- raw scRNA-seq count matrix (cells x genes) from reCAT package data, e.g. ola_mES_2i.RData (mESC-SMARTer dataset)
- cell cycle base gene list used by reCAT's get_test_exp filter

## Outputs

- DESeq size-factor-normalized, log2(TPM+1) expression matrix compatible with reCAT test_exp
- per-cell expression values of marker genes (Ccne1, Pcna, Cdc6, Hjurp) extracted from the normalized matrix for violin-plot analysis

## How to apply

Load the raw expression matrix from the reCAT package data files (e.g., /data/ola_mES_2i.RData), apply DESeq size-factor normalization to correct for library-size differences between cells, and transform values to log2(TPM+1) as reCAT requires for its test_exp input. Pass the log-transformed matrix through the get_test_exp function, which restricts the data to cell cycle base genes (a filter on the original data) so downstream steps such as get_ordIndex, get_score, and the GMM/HMM stages operate on the standard input. For marker-gene analysis, extract genes such as Ccne1, Pcna, Cdc6, and Hjurp from the normalized matrix and keep per-cell values so their distributions can be split by G1/S/G2/M stage. Correct normalization is critical because overlapping per-stage marker distributions (Fig. 1a conclusion) only reflect biology if library-size effects are removed first.

## Related tools

- **reCAT** (R package providing get_test_exp to standardize the log2(TPM+1) input and filter to cell cycle base genes) — github.com/tinglab/reCAT
- **R** (implementation environment in which reCAT and the normalization workflow run)
- **ggplot2** (used downstream to draw violin plots of normalized marker-gene expression split by cell cycle stage)

## Examples

```
source("get_test_exp.R"); load("../data/ola_mES_2i.RData"); test_exp <- get_test_exp(t(test_exp))
```

## Evaluation signals

- The normalized matrix values fall in a log2(TPM+1) range (non-negative, log-scale), matching the reCAT test_exp convention.
- get_test_exp runs without error and returns a matrix restricted to cell cycle base genes.
- Downstream get_ordIndex and get_score complete on the normalized matrix, indicating the format meets reCAT's standard input requirements.
- Violin plots of Ccne1, Pcna, Cdc6, and Hjurp show strongly overlapping distributions across G1, S and G2/M, reproducing the paper's Fig. 1a conclusion that one marker gene is insufficient.

## Limitations

- reCAT assumes the input is log2(TPM+1); supplying raw counts or a different transform violates the tool's input contract.
- get_test_exp filters the original data to cell cycle base genes only, so the output matrix is not suitable for genome-wide analyses outside the cell cycle scope.
- reCAT relies on a preselected gene set (378 genes) rather than data-driven gene selection; the authors note they would prefer semi-supervised selection for better future performance.

## Evidence

- [readme] in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data: "in reCAT, the value in test_exp should be log2(TPM+1), and then you can use ***get_test_exp*** function to get standard input data"
- [readme] in test_exp, there are only cycle base gene's value, we just do a filter on the original data: "in test_exp, there are only cycle base gene's value, we just do a filter on the original data, so that the data is easy to handle in following steps"
- [intro] The first step is data processing, including quality control, normalization, and clustering of single cells using the Gaussian mixture model (GMM) according to a user-defined phase number k: "The first step is data processing, including quality control, normalization, and clustering of single cells using the Gaussian mixture model (GMM) according to a user-defined phase number k"
- [intro] we first show one marker gene is not sufficient to give reliable information about cell cycle stages in scRNA-seq data sets: "we first show one marker gene is not sufficient to give reliable information about cell cycle stages in scRNA-seq data sets"
- [discussion] Instead of the preselected gene set (378 genes), we would prefer semi-supervised selection of cell cycle genes from the data: "Instead of the preselected gene set (378 genes), we would prefer semi-supervised selection of cell cycle genes from the data"
