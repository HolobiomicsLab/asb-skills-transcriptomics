---
name: protein-prediction-correlation-comparison
description: Use when benchmarking cTP-net surface-protein imputation on holdout CITE-seq
  cells and you need to isolate the contribution of SAVER-X denoising versus other
  preprocessing or training parameter choices, measured as correlation retraction.
when_to_use_negative:
- Expression data is not UMI-based count data (e.g. TPM/RPKM), since cTP-net can only
  apply to count-based expression input
- You only need a routine cTP-net imputation run — this skill is an ablation comparison
  requiring both a denoised baseline and a retrained raw-input model on matched validation
  cells
- Cells are not of the hematopoietic lineage, which was not tested in the paper's
  benchmarks
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: cTP-net
  role: Multiple-branch neural network retrained on raw (undenoised) input to obtain
    predicted surface protein relative abundances
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoising step that is deliberately omitted in the ablation arm and provides
    the baseline against which the ~0.02 correlation retraction is measured
- name: Seurat
  role: Default LogNormalize transformation applied to the raw counts to construct
    the ablation input feature matrix
- name: Adam (optimizer)
  role: Stochastic optimizer used for retraining, with learning rate 10e-5 for 139
    epochs
- name: Python
  role: Language for the shared training script (training_05152020.py) used to retrain
    cTP-net on raw input
  repo: https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:b0d4fb49aca2ab3f8c8f5dc22bc2ee7c82001f0dd80ab920303d2d2f93f37b0f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - Adam (optimizer)
  - Python
  - SAVER-X
  - Seurat
  - cTP-net
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# protein-prediction-correlation-comparison

## Summary

An ablation-comparison skill for cTP-net: quantify how much per-protein Pearson correlation between predicted and measured surface protein abundance drops when SAVER-X denoising is replaced by raw LogNormalize RNA input, and compare that retraction (~0.02) against other parameter tweaks.

## When to use

Use when benchmarking cTP-net surface-protein imputation on holdout CITE-seq cells and you need to isolate the contribution of SAVER-X denoising versus other preprocessing or training parameter choices, measured as correlation retraction.

## When NOT to use

- Expression data is not UMI-based count data (e.g. TPM/RPKM), since cTP-net can only apply to count-based expression input
- You only need a routine cTP-net imputation run — this skill is an ablation comparison requiring both a denoised baseline and a retrained raw-input model on matched validation cells
- Cells are not of the hematopoietic lineage, which was not tested in the paper's benchmarks

## Inputs

- Holdout CITE-seq UMI count matrix (e.g. CITE-PBMC or CITE-CBMC) with paired surface protein ADT counts
- SAVER-X-denoised baseline cTP-net per-protein correlations on the same validation cells
- Pretrained/training cTP-net multiple-branch neural network code (training_05152020.py)

## Outputs

- Per-protein Pearson correlations for cTP-net predictions from raw (undenoised) input
- correlation_retraction_summary.csv with per-protein and overall correlation retraction versus the denoised baseline (expected overall ~0.02)

## How to apply

Preprocess the holdout CITE-seq UMI matrix with the paper's quality filters (genes with <10 counts across cells and cells with <200 detected genes removed), then apply Seurat default LogNormalize but skip SAVER-X denoising to build the raw-input feature matrix. Retrain the cTP-net multiple-branch neural network using the shared training script (training_05152020.py), optimizing with Adam at learning rate 10e-5 for 139 epochs, holding out 10% of cells for testing and selecting the model on testing error. Compute per-protein Pearson correlations between predicted and measured relative surface protein abundances (e.g. the 24 immunophenotype markers) on the same validation cells used for the denoised baseline, since comparisons are limited to the same validation cells to account for subset variation. Compare against the SAVER-X-denoised baseline correlations per protein and overall; the expected result is a retraction of approximately 0.02 in correlation, larger than any other parameter tweak.

## Related tools

- **cTP-net** (Multiple-branch neural network retrained on raw (undenoised) input to obtain predicted surface protein relative abundances) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoising step that is deliberately omitted in the ablation arm and provides the baseline against which the ~0.02 correlation retraction is measured)
- **Seurat** (Default LogNormalize transformation applied to the raw counts to construct the ablation input feature matrix)
- **Adam (optimizer)** (Stochastic optimizer used for retraining, with learning rate 10e-5 for 139 epochs)
- **Python** (Language for the shared training script (training_05152020.py) used to retrain cTP-net on raw input) — https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py

## Examples

```
python cTPnet/extdata/training_05152020.py  # retrain cTP-net on raw LogNormalize input; Adam, lr=10e-5, 139 epochs; then compute per-protein Pearson correlations vs denoised baseline
```

## Evaluation signals

- Overall correlation retraction is approximately 0.02, consistent with the Discussion and Supplementary Table 5
- The raw-input retraction exceeds the effect of every other parameter tweak tested
- Summary CSV contains per-protein correlation fields for the evaluated surface proteins
- Retraining hyperparameters match the paper: Adam, learning rate 10e-5, 139 epochs, 90%/10% train/test split with model selection on testing error
- Baseline and ablation correlations are computed on the same validation cells

## Limitations

- cTP-net can only apply to count-based expression input (UMI-based); TPM/RPKM accuracy is unknown
- Comparisons are limited to the same cells that were in the validation set to account for variations between subsets
- The paper did not test imputation of immune-related markers on cells outside the hematopoietic lineage

## Evidence

- [discussion] SAVER-X denoising contribution to cTP-net accuracy: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] Training hyperparameters for the retrained model: "The objective function was optimized stochastically with Adam46 with learning rate set to 10e–5 for 139 epochs"
- [readme] Recommendation that denoising precedes imputation: "we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance"
- [discussion] Quality filters applied before constructing the input matrix: "low-quality gene (<10 counts across cells) and low-quality cells (<200 genes detected) are removed"
- [discussion] Train/test split and model selection criterion: "we keep only 10% of the cells as the testing set, and use the other 90% of the cells for training. The optimal model was selected based on the testing error."
