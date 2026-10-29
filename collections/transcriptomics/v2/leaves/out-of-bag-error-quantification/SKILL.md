---
name: out-of-bag-error-quantification
description: 'Use when you need an imputation-free supervised benchmark of cell-type
  separability: you have labeled scRNA-seq expression matrices (raw and imputed) and
  want a single quantitative error metric per method.'
when_to_use_negative:
- Input matrices lack cell-type labels — random-forest OOB error cannot be computed
  without a class target.
- You need zero-imputation accuracy on individual entries (e.g., true vs. biological
  zero recovery) rather than downstream cell-class separability.
- Datasets are very large (100,000+ cells) where methods like SAVER and scImpute are
  extremely slow and a classifier-based benchmark on downsampled data is more tractable.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ranger
  role: Random-forest classifier trained with default settings to predict cell class;
    its out-of-bag error is the benchmark metric.
- name: ALRA
  role: Primary imputation method whose effect on OOB error is quantified; rank auto-selected
    by choose_k, quantile.prob = 0.001.
  repo: https://github.com/KlugerLab/ALRA
- name: MAGIC
  role: Baseline imputer (negative values set to zero) compared in the OOB error table.
- name: SAVER
  role: Baseline imputer run with do.fast = TRUE on the same normalized matrices.
- name: DCA
  role: Baseline deep-count-autoencoder imputer included in the comparison.
- name: scImpute
  role: Baseline imputer; run on a subset where needed for tractability.
- name: Seurat
  role: R toolkit for single-cell genomics within which ALRA is integrated as RunALRA();
    used for handling scRNA-seq objects.
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-27729-z
    title: Zero-preserving imputation of single-cell RNA-seq data
  source_content_hash: sha256:ce4363cb91030ea307566465522f56e5a4f3a756e12fb33e282d029769da559a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KlugerLab/ALRA
  tools:
  - ALRA
  - DCA
  - MAGIC
  - SAVER
  - Seurat
  - ranger
  - scImpute
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# out-of-bag-error-quantification

## Summary

Quantifies whether scRNA-seq imputation (e.g., ALRA) improves cell-type separability by training a ranger random forest on raw vs. imputed expression matrices and comparing average out-of-bag classification error across the Zeisel, Baron, Chen, and La Manno downsampled datasets.

## When to use

Use when you need an imputation-free supervised benchmark of cell-type separability: you have labeled scRNA-seq expression matrices (raw and imputed) and want a single quantitative error metric per method.

## When NOT to use

- Input matrices lack cell-type labels — random-forest OOB error cannot be computed without a class target.
- You need zero-imputation accuracy on individual entries (e.g., true vs. biological zero recovery) rather than downstream cell-class separability.
- Datasets are very large (100,000+ cells) where methods like SAVER and scImpute are extremely slow and a classifier-based benchmark on downsampled data is more tractable.

## Inputs

- Downsampled scRNA-seq expression matrices (Zeisel 1799 cells, Baron 1076 cells, Chen 7712 cells, La Manno 947 cells) from Huang et al. 2018 (SAVER)
- Cell-identity class labels provided by the original authors
- Normalized (10,000-count library normalization, log-transformed) raw and imputed matrices with cells as rows and genes as columns

## Outputs

- Out-of-bag classification error per dataset and per imputation method (ALRA, MAGIC, SAVER, DCA, scImpute, raw baseline)
- Comparison table written to oob_error_table.csv with averages across the four datasets

## How to apply

Load the downsampled expression matrices distributed with Huang et al. 2018 (SAVER) — Zeisel (1799 cells), Baron (1076 cells), Chen (7712 cells), La Manno (947 cells) — together with the original authors' cell-identity labels. Normalize each matrix (library normalization to 10,000 counts and log-transformation via ALRA's normalize_data) and produce imputed versions with each method, letting choose_k select the ALRA rank (expected k = 12, 21, 15, 16 for Baron, Chen, La Manno, Zeisel) and using quantile.prob = 0.001. For each raw and imputed matrix, train a ranger random forest with default settings to predict cell class from expression values; the out-of-bag error is computed internally by ranger without a held-out test set, making the comparison cheap and reproducible. Record the OOB error per dataset and method, average across the four datasets, and write the comparison table to oob_error_table.csv. In the reference result, ALRA reduced average error from 16.4% (raw) to 8%, beating MAGIC (10.3%), SAVER (9.8%), and DCA (9.4%), while scImpute increased error to 28%.

## Related tools

- **ranger** (Random-forest classifier trained with default settings to predict cell class; its out-of-bag error is the benchmark metric.)
- **ALRA** (Primary imputation method whose effect on OOB error is quantified; rank auto-selected by choose_k, quantile.prob = 0.001.) — https://github.com/KlugerLab/ALRA
- **MAGIC** (Baseline imputer (negative values set to zero) compared in the OOB error table.)
- **SAVER** (Baseline imputer run with do.fast = TRUE on the same normalized matrices.)
- **DCA** (Baseline deep-count-autoencoder imputer included in the comparison.)
- **scImpute** (Baseline imputer; run on a subset where needed for tractability.)
- **Seurat** (R toolkit for single-cell genomics within which ALRA is integrated as RunALRA(); used for handling scRNA-seq objects.) — https://github.com/satijalab/seurat

## Examples

```
k_choice <- choose_k(A_norm); A_norm_completed <- alra(A_norm, k=k_choice$k, quantile.prob=0.001)[[3]]; rf <- ranger(class ~ ., data=df); rf$prediction.error
```

## Evaluation signals

- OOB errors recorded for every dataset x method combination with no missing cells in oob_error_table.csv.
- Raw (unimputed) baseline average OOB error falls near the reference 16.4%, confirming the matrices and labels are loaded correctly.
- ALRA-imputed average OOB error is substantially below raw (reference: 8% vs. 16.4%) and below MAGIC (10.3%), SAVER (9.8%), and DCA (9.4%).
- ALRA ranks match choose_k expectations: k = 12, 21, 15, 16 for Baron, Chen, La Manno, Zeisel respectively.
- Matrices passed to ranger have cells as rows and genes as columns, as required by ALRA conventions.

## Limitations

- scImpute increased OOB error to 28%, so imputation method choice materially changes the benchmark; a method can hurt separability.
- SAVER and scImpute may be extremely slow on datasets with over 100,000 cells, requiring downsampling or subset runs for tractability.
- MAGIC treats all zeros as missing data and outputs a matrix in which every gene is expressed in every cell, so negative or non-zero outputs must be clipped before comparison.

## Evidence

- [results] ALRA compared to baselines on classification error: "several imputation methods have been developed, including DCA5, MAGIC6, SAVER7, and scImpute8"
- [intro] MAGIC treats zeros as missing: "Some methods such as DCA and MAGIC treat all zeros as “missing data” and output a matrix in which every gene is expressed in every cell"
- [intro] Scalability limitation of baseline imputers: "SAVER and scImpute may be extremely slow on scRNA-seq datasets with over 100,000 cells"
- [readme] ALRA usage from README: "Please be sure to pass ALRA a matrix where the cells are rows and genes are columns."
- [readme] Normalization step from README: "# We use library and log normalization, but other approaches may also work well.
result.completed <- alra(A_norm)"
