---
name: masked-data-ground-truth-mse-evaluation
description: Use when comparing imputation methods (LATE, TRANSLATE, DCA, scVI, SAVER,
  MAGIC, ALRA, scImpute) on masked scRNA-seq data (e.g., example.msk90.hd5) with an
  unmasked ground truth available for error quantification.
when_to_use_negative:
- The goal is cell-type separation quality (BCSS/WCSS after t-SNE on labeled data
  like PBMC_G949_10K) rather than imputation accuracy against a ground truth
- The imputed matrix is not on the same log10(count+1) scale as the ground truth,
  since MSE values are not comparable across scales
- Only one imputation method is available and no baseline comparison (DCA, scVI, SAVER,
  MAGIC, ALRA, scImpute) is possible
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: LATE
  role: Autoencoder imputation method under evaluation (random initialization, trained
    on masked input); produces step2/imputation.step2.hd5
  repo: https://github.com/audreyqyfu/LATE
- name: TRANSLATE
  role: Transfer-learning variant of LATE pre-trained on ref_example.mdk50.hd5, evaluated
    alongside LATE for MSE/gtMSE
  repo: https://github.com/audreyqyfu/LATE
- name: DCA
  role: Baseline imputation method run on identical masked inputs with recommended
    defaults
- name: scVI
  role: Baseline imputation method run on identical masked inputs with recommended
    defaults
- name: SAVER
  role: Baseline imputation method run on identical masked inputs with recommended
    defaults
- name: MAGIC
  role: Baseline imputation method run on identical masked inputs with recommended
    defaults
- name: ALRA
  role: Baseline imputation method run on identical masked inputs with recommended
    defaults
- name: scImpute
  role: Baseline imputation method run on identical masked inputs with recommended
    defaults
- name: TensorFlow
  role: Backend (1.x, e.g., 1.15) required to train the LATE/TRANSLATE autoencoders
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1007/s40484-019-0192-7
    title: Imputation of single‐cell gene expression with an autoencoder neural network
  source_content_hash: sha256:a83bfac15ec662c438e8c6eaa504155c61a58e8dcec7cd7b0445950aa106fbae
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
  - matplotlib
  - numpy
  - pandas
  - scImpute
  - scVI
  - scipy
  - seaborn
  - sklearn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 20
---

# masked-data-ground-truth-mse-evaluation

## Summary

Benchmark scRNA-seq imputation accuracy by computing mean squared error (MSE) of imputed matrices against ground truth on masked datasets, and gtMSE on real datasets without ground truth, following the LATE/TRANSLATE paper's Supplementary Table S3 design.

## When to use

Use when comparing imputation methods (LATE, TRANSLATE, DCA, scVI, SAVER, MAGIC, ALRA, scImpute) on masked scRNA-seq data (e.g., example.msk90.hd5) with an unmasked ground truth available for error quantification.

## When NOT to use

- The goal is cell-type separation quality (BCSS/WCSS after t-SNE on labeled data like PBMC_G949_10K) rather than imputation accuracy against a ground truth
- The imputed matrix is not on the same log10(count+1) scale as the ground truth, since MSE values are not comparable across scales
- Only one imputation method is available and no baseline comparison (DCA, scVI, SAVER, MAGIC, ALRA, scImpute) is possible

## Inputs

- Masked scRNA-seq count matrix (hd5/csv/tsv format, e.g., example.msk90.hd5, PBMC_G949_10K)
- Ground-truth unmasked count matrix in the same format (e.g., example.hd5)
- Imputed output matrices from each method (e.g., step2/imputation.step2.hd5, log10(count+1) scale)
- Reference dataset for TRANSLATE pre-training (e.g., ref_example.mdk50.hd5, 10,000 genes x 30,000 cells)

## Outputs

- Per-dataset, per-method MSE/gtMSE comparison table mirroring Supplementary Table S3
- Combined (gene- and cell-dimension) LATE MSE value

## How to apply

Load the masked input matrix (e.g., example.msk90.hd5 from masking 90% of example.hd5, i.e., PBMC_G949_10K) together with its ground-truth unmasked matrix, transforming counts with log10(count+1) for imputation so all methods are compared on the same scale. Run LATE/TRANSLATE (autoencoder, TensorFlow 1.x, parameters from global_params.py) plus baseline methods DCA, scVI, SAVER, MAGIC, ALRA and scImpute with recommended defaults on identical masked inputs. For each method compute MSE of the imputed matrix (step2/imputation.step2.hd5, log10(count+1) scale) against ground truth; for real datasets without ground truth (PBMC_G5561, PBMC_G9987) compute gtMSE by setting ground truth equal to the input as in Example 4. Compile per-dataset, per-method MSE/gtMSE into a table mirroring Supplementary Table S3, including the combined (gene- and cell-dimension) LATE MSE. Validate the reproduction by checking that LATE's MSE is approximately 2% of ALRA's and that DCA is approximately 41% (PBMC_G5561) and 450% (PBMC_G9987) of the LATE-combined MSE.

## Related tools

- **LATE** (Autoencoder imputation method under evaluation (random initialization, trained on masked input); produces step2/imputation.step2.hd5) — https://github.com/audreyqyfu/LATE
- **TRANSLATE** (Transfer-learning variant of LATE pre-trained on ref_example.mdk50.hd5, evaluated alongside LATE for MSE/gtMSE) — https://github.com/audreyqyfu/LATE
- **DCA** (Baseline imputation method run on identical masked inputs with recommended defaults)
- **scVI** (Baseline imputation method run on identical masked inputs with recommended defaults)
- **SAVER** (Baseline imputation method run on identical masked inputs with recommended defaults)
- **MAGIC** (Baseline imputation method run on identical masked inputs with recommended defaults)
- **ALRA** (Baseline imputation method run on identical masked inputs with recommended defaults)
- **scImpute** (Baseline imputation method run on identical masked inputs with recommended defaults)
- **TensorFlow** (Backend (1.x, e.g., 1.15) required to train the LATE/TRANSLATE autoencoders)

## Examples

```
python example_analysis.py -mode='analysis' -infile='../data/example.msk90.hd5'
```

## Evaluation signals

- LATE MSE is approximately 2% of ALRA's MSE on the real no-ground-truth datasets
- DCA's MSE is approximately 41% (PBMC_G5561) and approximately 450% (PBMC_G9987) of the LATE-combined MSE
- LATE and TRANSLATE produce similar MSEs, each no more than a third of the MSE from scVI, SAVER and MAGIC
- All imputed matrices and ground truth are compared on the log10(count+1) scale
- gtMSE for real datasets without ground truth is computed with ground truth set equal to the input (Example 4 convention)

## Limitations

- gtMSE on real datasets without ground truth sets ground truth equal to the input, so it is not a true accuracy measure of recovery
- The combined (gene- and cell-dimension) LATE MSE is an ad hoc combination; joint optimization for both dimensions was not performed
- LATE and TRANSLATE assume all zeros are missing values and always impute zeros with a nonzero value, so biological zeros inflate apparent error
- Hyperparameters were manually tuned; systematic hyperparameter tuning (as implemented in DCA) was not explored, which may affect fair comparison

## Evidence
