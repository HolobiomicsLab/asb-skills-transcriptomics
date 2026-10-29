---
name: imputation-result-table-compilation
description: Use when multiple scRNA-seq imputation methods have been run on identical
  masked inputs and you must quantify imputation accuracy as MSE against ground truth,
  or gtMSE when no ground truth exists.
when_to_use_negative:
- When no reference/ground-truth matrix exists for a masked synthetic dataset and
  gtMSE-against-input would be meaningless (gtMSE against input is only for real datasets
  as in Example 4)
- When the goal is biological downstream analysis (clustering, differential expression,
  co-expression) rather than quantitative method benchmarking
- When imputation methods were run on different masked inputs or different parameter
  settings — cross-method MSE comparison would be invalid
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: LATE
  role: Autoencoder imputation method whose MSE (including the combined gene- and
    cell-dimension MSE) forms the reference column of the benchmark table
  repo: https://github.com/audreyqyfu/LATE
- name: TRANSLATE
  role: Transfer-learning variant of LATE (pre-training on ref_example.mdk50.hd5,
    then translate) compared against LATE via MSE
  repo: https://github.com/audreyqyfu/LATE
- name: numpy
  role: Numerical computation of MSE/gtMSE between imputed and ground-truth matrices
- name: pandas
  role: Assembly of per-dataset, per-method MSE values into a single comparison table
- name: TensorFlow (1.0-1.4)
  role: Backend on which LATE/TRANSLATE imputation results used in the comparison
    are generated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s40484-019-0192-7
    title: Imputation of single‐cell gene expression with an autoencoder neural network
  source_content_hash: sha256:855decf344f017503a4697071420290d3fbcccb2f2d7dfb636c746d5093b1921
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/audreyqyfu/LATE
  tools:
  - ALRA
  - DCA
  - LATE
  - MAGIC
  - Python+
  - SAVER
  - TRANSLATE
  - TensorFlow
  - TensorFlow (1.0-1.4)
  - numpy
  - pandas
  - scImpute
  - scVI
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# imputation-result-table-compilation

## Summary

Compile a per-dataset, per-method comparison table of imputation error (MSE and gtMSE) for scRNA-seq imputation methods (LATE, TRANSLATE, DCA, scVI, SAVER, MAGIC, ALRA, scImpute), mirroring the paper's Supplementary Table S3 so that relative performance claims can be verified numerically.

## When to use

Use when multiple scRNA-seq imputation methods have been run on identical masked inputs and you must quantify imputation accuracy as MSE against ground truth, or gtMSE when no ground truth exists.

## When NOT to use

- When no reference/ground-truth matrix exists for a masked synthetic dataset and gtMSE-against-input would be meaningless (gtMSE against input is only for real datasets as in Example 4)
- When the goal is biological downstream analysis (clustering, differential expression, co-expression) rather than quantitative method benchmarking
- When imputation methods were run on different masked inputs or different parameter settings — cross-method MSE comparison would be invalid

## Inputs

- Masked scRNA-seq input matrices (e.g. example.msk90.hd5 / PBMC_G949_10K) in csv, tsv, or 10x Genomics h5 format
- Ground-truth unmasked count matrices matching the masked inputs
- Imputed matrices from LATE/TRANSLATE (hd5 format, log10(count+1) scale, e.g. step2/imputation.step2.hd5)
- Imputed matrices from baseline methods DCA, scVI, SAVER, MAGIC, ALRA, scImpute run on the identical masked inputs

## Outputs

- Per-dataset, per-method MSE/gtMSE comparison table mirroring Supplementary Table S3
- Combined (gene- and cell-dimension) LATE MSE values
- Ratios of baseline-method MSE to LATE-combined MSE (e.g. DCA 41%/450%, ALRA ~50x)

## How to apply

Load the masked input matrices (e.g. example.msk90.hd5) together with their ground-truth unmasked counterparts, and the imputation outputs (e.g. LATE/TRANSLATE results in step2/imputation.step2.hd5, which are on the log10(count+1) scale), ensuring all methods were run on the identical masked inputs under recommended default settings. For each method, compute mean squared error (MSE) of the imputed matrix against the ground truth on the log10(count+1) scale. For real datasets without ground truth (e.g. PBMC_G5561, PBMC_G9987), follow Example 4 and compute gtMSE with the ground truth set equal to the input. Include the combined (gene- and cell-dimension) LATE MSE alongside per-dataset values, assembling everything into a single table mirroring Supplementary Table S3. Validate by checking the reported relations reproduce: LATE MSE approximately 2% of ALRA's, and DCA at approximately 41% (PBMC_G5561) and 450% (PBMC_G9987) of the LATE-combined MSE, confirming LATE achieves lower error in most cases.

## Related tools

- **LATE** (Autoencoder imputation method whose MSE (including the combined gene- and cell-dimension MSE) forms the reference column of the benchmark table) — https://github.com/audreyqyfu/LATE
- **TRANSLATE** (Transfer-learning variant of LATE (pre-training on ref_example.mdk50.hd5, then translate) compared against LATE via MSE) — https://github.com/audreyqyfu/LATE
- **numpy** (Numerical computation of MSE/gtMSE between imputed and ground-truth matrices)
- **pandas** (Assembly of per-dataset, per-method MSE values into a single comparison table)
- **TensorFlow (1.0-1.4)** (Backend on which LATE/TRANSLATE imputation results used in the comparison are generated)

## Examples

```
python example_analysis.py -mode='analysis' -infile='../data/example.msk90.hd5'
```

## Evaluation signals

- LATE MSE is approximately 2% of ALRA's MSE on real datasets without ground truth
- DCA MSE is approximately 41% of the LATE-combined MSE on PBMC_G5561 and approximately 450% on PBMC_G9987
- LATE and TRANSLATE yield similar MSEs on real datasets, each no more than a third of scVI, SAVER, or MAGIC
- All methods were evaluated against the same masked inputs on the log10(count+1) scale; LATE/TRANSLATE outputs read from step2/imputation.step2.hd5 match the input matrix layout
- Table structure mirrors Supplementary Table S3 with a per-dataset, per-method grid including the combined LATE MSE

## Limitations

- On real datasets without ground truth, gtMSE is computed with ground truth set equal to the input, so it measures deviation from the observed (sparse) data, not true imputation accuracy
- The combined (gene- and cell-dimension) LATE MSE arises from an approach the authors describe as 'rather ad hoc'; joint optimization was not performed
- Hyperparameters for LATE were manually tuned; systematic tuning as in DCA was not explored, so the comparison reflects one operating point per method
- LATE/TRANSLATE assume all zeros are missing values and always impute zeros with a nonzero value, which affects error metrics where biological zeros dominate

## Evidence
