---
name: scrna-seq-dropout-imputation-autoencoder
description: Use when a sparse scRNA-seq count matrix (h5/csv/tsv, >80-95% zeros)
  needs dropout imputation before downstream analyses such as clustering, cell-type
  separation, differential expression, or MSE benchmarking against other imputers.
when_to_use_negative:
- The data contain biological zeros that must be preserved (LATE/TRANSLATE assume
  all zeros are missing values and impute zeros always with a nonzero value)
- The environment only supports TensorFlow 2 (LATE is incompatible with TensorFlow
  2 and requires TF 1.x in a virtual environment)
- Joint optimization of gene- and cell-dimension imputation or systematic hyperparameter
  tuning is required (the current dimension combination is ad hoc and hyperparameters
  were only manually tuned)
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: LATE
  role: Trains an autoencoder with random initial parameter values de novo on the
    sparse scRNA-seq input data to impute dropout zeros (mode='late')
  repo: https://github.com/audreyqyfu/LATE
- name: TRANSLATE
  role: 'Transfer-learning variant: pre-trains the autoencoder on a labeled reference
    dataset (step1/) and uses those weights as initialization to impute the input
    dataset (step2/)'
  repo: https://github.com/audreyqyfu/LATE
- name: DCA
  role: Baseline imputation method compared against LATE/TRANSLATE on masked inputs
    using recommended defaults
- name: scVI
  role: Baseline imputation method compared against LATE/TRANSLATE in MSE and cell-type
    separation benchmarks
- name: TensorFlow (1.0-1.4)
  role: Deep learning backend on which the LATE/TRANSLATE autoencoder implementation
    is built (install TF 1.15 in a virtualenv)
- name: MulticoreTSNE
  role: t-SNE embedding (after PCA projection) of imputed data colored by cell type
    for the BCSS/WCSS separation analysis
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1007/s40484-019-0192-7
    title: Imputation of single‐cell gene expression with an autoencoder neural network
  source_content_hash: sha256:d30d3651c7ef2725ba7b891c352180403bac56f5c513ed9f535b40068095b1d8
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/audreyqyfu/LATE
  tools:
  - ALRA
  - DCA
  - LATE
  - MAGIC
  - MulticoreTSNE
  - Python+
  - SAVER
  - TRANSLATE
  - TensorFlow
  - TensorFlow (1.0-1.4)
  - matplotlib
  - numpy
  - pandas
  - scImpute
  - scVI
  - scipy
  - seaborn
  - sklearn
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6111111111111112
n_claims: 18
---

# scrna-seq-dropout-imputation-autoencoder

## Summary

Imputes dropout zeros in highly sparse scRNA-seq count matrices using the LATE autoencoder (random initialization) or TRANSLATE (transfer learning from a labeled reference dataset), treating zeros as missing values and leveraging gene/cell dependence to recover true expression levels.

## When to use

Use when a sparse scRNA-seq count matrix (h5/csv/tsv, >80-95% zeros) needs dropout imputation before downstream analyses such as clustering, cell-type separation, differential expression, or MSE benchmarking against other imputers.

## When NOT to use

- The data contain biological zeros that must be preserved (LATE/TRANSLATE assume all zeros are missing values and impute zeros always with a nonzero value)
- The environment only supports TensorFlow 2 (LATE is incompatible with TensorFlow 2 and requires TF 1.x in a virtual environment)
- Joint optimization of gene- and cell-dimension imputation or systematic hyperparameter tuning is required (the current dimension combination is ad hoc and hyperparameters were only manually tuned)

## Inputs

- Masked or raw scRNA-seq count matrix in h5 (10x Genomics sparse matrix, genes in rows and cells in columns), csv, or tsv format (e.g., example.msk90.hd5 / PBMC_G949_10K)
- Labeled reference scRNA-seq dataset for TRANSLATE pre-training (e.g., ref_example.mdk50.hd5, 10,000 genes x 30,000 cells)
- Ground-truth count matrix in the same format for analysis (set equal to the input when no ground truth exists)
- Cell type labels (e.g., 10 PBMC cell types) for BCSS/WCSS cell-type separation analysis

## Outputs

- Imputed data matrix in hd5 format at step2/imputation.step2.hd5, on the log10(count+1) scale with the same layout as the input
- Pre-trained autoencoder parameters in step1/ (TRANSLATE step 1)
- Eval/ folder with summary and visualization of imputation results (PCA+t-SNE embeddings, BCSS/WCSS tables, MSE/gtMSE values)

## How to apply

Set up a Python 3.5+ virtual environment with TensorFlow 1.x (e.g., 1.15, since LATE is incompatible with TensorFlow 2) and pandas 0.25.1 plus numpy, matplotlib, scipy, seaborn, tables, sklearn and MulticoreTSNE. For LATE, run the imputation mode on the masked input (e.g., example.msk90.hd5) with random parameter initialization; for TRANSLATE, first pre-train the autoencoder on a labeled reference dataset (ref_example.mdk50.hd5) to produce step1/, then translate on the input using those weights as initialization to produce step2/. Inputs are log10(count+1)-transformed read-count matrices (h5: genes in rows, cells in columns; csv/tsv: cells in rows, genes in columns), and imputed values are written to step2/imputation.step2.hd5 on the same log10(count+1) scale. Default parameters live in global_params.py and may be overridden in the example_*.py scripts; run mode='analysis' against a ground-truth matrix (or the input itself when no ground truth exists) to generate the Eval/ folder. Judge success by low MSE/gtMSE versus ground truth and, for PBMC_G949_10K, by high BCSS and low WCSS across the 10 labeled cell types with tSNE plots resembling ground truth.

## Related tools

- **LATE** (Trains an autoencoder with random initial parameter values de novo on the sparse scRNA-seq input data to impute dropout zeros (mode='late')) — https://github.com/audreyqyfu/LATE
- **TRANSLATE** (Transfer-learning variant: pre-trains the autoencoder on a labeled reference dataset (step1/) and uses those weights as initialization to impute the input dataset (step2/)) — https://github.com/audreyqyfu/LATE
- **DCA** (Baseline imputation method compared against LATE/TRANSLATE on masked inputs using recommended defaults)
- **scVI** (Baseline imputation method compared against LATE/TRANSLATE in MSE and cell-type separation benchmarks)
- **TensorFlow (1.0-1.4)** (Deep learning backend on which the LATE/TRANSLATE autoencoder implementation is built (install TF 1.15 in a virtualenv))
- **MulticoreTSNE** (t-SNE embedding (after PCA projection) of imputed data colored by cell type for the BCSS/WCSS separation analysis)

## Examples

```
python example_late.py -mode='late' -infile='../data/example.msk90.hd5'
```

## Evaluation signals

- step2/imputation.step2.hd5 exists and its values are on the log10(count+1) scale with the same layout (rows/columns) as the input matrix
- MSE/gtMSE of the imputed matrix against ground truth is lower than baselines (LATE MSE approximately 2% of ALRA's; no more than a third of scVI/SAVER/MAGIC on real datasets)
- On PBMC_G949_10K, tSNE (after PCA) colored by the 10 cell types resembles the ground-truth embedding with high BCSS and low WCSS for LATE/TRANSLATE (and DCA/scVI), unlike SAVER, scImpute, MAGIC and ALRA
- TRANSLATE step 1 wrote pre-trained parameters to step1/ before step 2 used them as initialization for step2/
- Imputation ran under TensorFlow 1.x (e.g., 1.15) without TF2 incompatibility errors, with default parameters sourced from global_params.py

## Limitations

- LATE and TRANSLATE assume all zeros in the scRNA-seq data are missing values and always impute zeros with a nonzero value; biological zeros are not distinguished from technical zeros
- Hyperparameters were manually tuned; systematic hyperparameter tuning (as implemented in DCA) was not explored
- Combining the two imputed data matrices (gene- and cell-dimension) is rather ad hoc; joint optimization for both dimensions was not performed
- LATE is incompatible with TensorFlow 2 and requires a virtual environment with TensorFlow 1.x (e.g., 1.15) and pandas 0.25.1

## Evidence
