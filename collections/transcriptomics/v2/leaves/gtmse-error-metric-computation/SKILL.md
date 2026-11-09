---
name: gtmse-error-metric-computation
description: 'Use when quantifying imputation accuracy of scRNA-seq methods: comparing
  MSE/gtMSE against a ground-truth matrix, or evaluating real datasets (e.g. PBMC_G5561,
  PBMC_G9987) where ground truth is set equal to the input.'
when_to_use_negative:
- You need to evaluate clustering, differential expression, or cell-type separation
  quality rather than elementwise imputation error — this metric does not capture
  downstream utility.
- The ground truth for a masked dataset is unavailable and the analysis goal requires
  a true out-of-sample error estimate, not gtMSE with ground truth set to the input
  (which measures reconstruction of observed entries).
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: LATE
  role: Autoencoder imputation method whose MSE/gtMSE is the primary benchmarked quantity
  repo: github.com__audreyqyfu__LATE
- name: TRANSLATE
  role: Transfer-learning variant of LATE evaluated alongside LATE in the MSE/gtMSE
    table
- name: DCA
  role: Baseline imputation method run on identical masked inputs for MSE comparison
- name: scVI
  role: Baseline imputation method for the benchmark table
- name: SAVER
  role: Baseline imputation method for the benchmark table
- name: MAGIC
  role: Baseline imputation method for the benchmark table
- name: ALRA
  role: Baseline imputation method; its MSE is ~50x the LATE MSE in the validation
    relation
- name: scImpute
  role: Baseline imputation method for the benchmark table
- name: numpy
  role: Array arithmetic for elementwise squared-error computation
- name: pandas
  role: Tabulation of per-dataset, per-method MSE/gtMSE values into the comparison
    table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s40484-019-0192-7
    title: Imputation of single‐cell gene expression with an autoencoder neural network
  source_content_hash: sha256:8d9ba8c87fee987ac2a79bef53b7360a99c62122fcd466295d41e9936131965a
schema_version: 0.2.0
metadata:
  license_tier: unknown
  tools:
  - ALRA
  - DCA
  - LATE
  - MAGIC
  - Python+
  - SAVER
  - TRANSLATE
  - TensorFlow
  - numpy
  - pandas
  - scImpute
  - scVI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# gtmse-error-metric-computation

## Summary

Computes mean squared error (MSE) and ground-truth MSE (gtMSE) to benchmark scRNA-seq imputation methods (LATE, TRANSLATE, DCA, scVI, SAVER, MAGIC, ALRA, scImpute) on masked synthetic data and real data without ground truth, as in the LATE paper's Supplementary Table S3 comparison.

## When to use

Use when quantifying imputation accuracy of scRNA-seq methods: comparing MSE/gtMSE against a ground-truth matrix, or evaluating real datasets (e.g. PBMC_G5561, PBMC_G9987) where ground truth is set equal to the input.

## When NOT to use

- You need to evaluate clustering, differential expression, or cell-type separation quality rather than elementwise imputation error — this metric does not capture downstream utility.
- The ground truth for a masked dataset is unavailable and the analysis goal requires a true out-of-sample error estimate, not gtMSE with ground truth set to the input (which measures reconstruction of observed entries).

## Inputs

- Masked scRNA-seq input matrix (e.g. example.msk90.hd5, csv/tsv/h5 10x format)
- Ground-truth unmasked matrix (e.g. example.hd5) in the same format as the input
- Imputed output matrices from each method, e.g. step2/imputation.step2.hd5 on the log10(count+1) scale
- Imputed matrices from baseline methods (DCA, scVI, SAVER, MAGIC, ALRA, scImpute) run on identical masked inputs

## Outputs

- Per-dataset, per-method MSE (synthetic masked datasets with ground truth)
- gtMSE values for real datasets without ground truth (PBMC_G5561, PBMC_G9987, mouse brain 10x)
- Combined gene- and cell-dimension LATE MSE
- Comparison table mirroring Supplementary Table S3

## How to apply

Load the masked input matrix (e.g. example.msk90.hd5) together with its ground-truth unmasked matrix, and the imputed matrices (e.g. LATE/TRANSLATE output in step2/imputation.step2.hd5) — all compared on the log10(count+1) scale. For each imputation method, compute the MSE of the imputed matrix against the ground truth. For real datasets without ground truth (as in the paper's Example 4 on mouse brain 10x data and PBMC datasets), compute gtMSE by setting the ground truth equal to the input matrix. For LATE, also compute the combined MSE that merges the gene-dimension and cell-dimension imputed matrices, since the two-dimension combination is ad hoc but required for the benchmark table. Compile per-dataset, per-method values into a table mirroring Supplementary Table S3, then validate the expected relations: LATE MSE approximately 2% of ALRA's, LATE/TRANSLATE no more than a third of the MSE from scVI, SAVER and MAGIC, and DCA at approximately 41% (PBMC_G5561) and 450% (PBMC_G9987) of the LATE-combined MSE.

## Related tools

- **LATE** (Autoencoder imputation method whose MSE/gtMSE is the primary benchmarked quantity) — github.com__audreyqyfu__LATE
- **TRANSLATE** (Transfer-learning variant of LATE evaluated alongside LATE in the MSE/gtMSE table)
- **DCA** (Baseline imputation method run on identical masked inputs for MSE comparison)
- **scVI** (Baseline imputation method for the benchmark table)
- **SAVER** (Baseline imputation method for the benchmark table)
- **MAGIC** (Baseline imputation method for the benchmark table)
- **ALRA** (Baseline imputation method; its MSE is ~50x the LATE MSE in the validation relation)
- **scImpute** (Baseline imputation method for the benchmark table)
- **numpy** (Array arithmetic for elementwise squared-error computation)
- **pandas** (Tabulation of per-dataset, per-method MSE/gtMSE values into the comparison table)

## Examples

```
python example_analysis.py -mode='analysis' -infile='../data/example.msk90.hd5'
```

## Evaluation signals

- MSE is computed on the log10(count+1) scale, matching the scale of the imputed output (step2/imputation.step2.hd5).
- For datasets without ground truth, the ground-truth matrix is set equal to the input matrix, reproducing the Example 4 convention.
- Reproduced relations hold: LATE MSE approximately 2% of ALRA's MSE; DCA at approximately 41% (PBMC_G5561) and 450% (PBMC_G9987) of the LATE-combined MSE.
- LATE and TRANSLATE MSEs are similar to each other, each no more than a third of the MSE from scVI, SAVER and MAGIC.
- The final table mirrors Supplementary Table S3 and includes the combined gene- and cell-dimension LATE MSE.

## Limitations

- gtMSE with ground truth set equal to the input only measures reconstruction of observed entries, not recovery of true values behind dropouts.
- The combined gene- and cell-dimension LATE imputation approach is ad hoc; joint optimization for both dimensions was not performed, affecting the combined MSE.
- LATE/TRANSLATE assume all zeros are missing values and always impute zeros with a nonzero value, so the metric conflates biological and technical zeros.
- Hyperparameters were manually tuned without systematic tuning (as implemented in DCA), so MSE comparisons may partly reflect tuning effort rather than method quality.

## Evidence

- [results] LATE vs baselines relative MSE: [evidence span withheld — anchor status: not_found]
