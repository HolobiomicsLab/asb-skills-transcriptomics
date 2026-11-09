---
name: surface-protein-imputation-model-retraining
description: Use when benchmarking how preprocessing choices (skipping SAVER-X denoising,
  altered normalization) affect cTP-net protein-imputation accuracy on paired CITE-seq
  transcriptome–protein data.
when_to_use_negative:
- The goal is routine protein imputation with best accuracy — in that case use the
  pretrained cTP-net model with SAVER-X-denoised input, which the paper shows performs
  better
- Input scRNA-seq data is not UMI-based count data — the paper states cTP-net can
  only apply to count-based expression input (UMI-based)
- CITE-seq data with TPM or RPKM expression metrics — prediction accuracy is unknown
  because such data were not available for testing
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: cTP-net
  role: Multiple-branch deep neural network retrained on raw vs. denoised input to
    impute surface protein abundances
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoising step being ablated; its removal defines the raw-input arm of the
    comparison
- name: Seurat
  role: Applies default LogNormalize transformation to (denoised or raw) counts and
    default quality filtering
- name: Adam (optimizer)
  role: Stochastic optimizer used for retraining, learning rate 10e-5 for 139 epochs
- name: Python
  role: Runtime for the shared training script training_05152020.py used to retrain
    the network
  repo: https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:f2523b8bad4bddac72a8fd81e5a2762a24b7cf903fe0dcfbb68916ffe3c11fc4
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# surface-protein-imputation-model-retraining

## Summary

Retrain the cTP-net multiple-branch neural network under modified preprocessing (e.g. replacing SAVER-X denoising with raw UMI counts) and quantify the resulting per-protein Pearson correlation retraction against the denoised baseline, as an ablation of the imputation pipeline.

## When to use

Use when benchmarking how preprocessing choices (skipping SAVER-X denoising, altered normalization) affect cTP-net protein-imputation accuracy on paired CITE-seq transcriptome–protein data.

## When NOT to use

- The goal is routine protein imputation with best accuracy — in that case use the pretrained cTP-net model with SAVER-X-denoised input, which the paper shows performs better
- Input scRNA-seq data is not UMI-based count data — the paper states cTP-net can only apply to count-based expression input (UMI-based)
- CITE-seq data with TPM or RPKM expression metrics — prediction accuracy is unknown because such data were not available for testing

## Inputs

- Holdout CITE-seq UMI count matrix with paired ADT surface protein measurements
- Surface protein ADT count matrix (for relative abundance transformation of responses)
- cTP-net training script training_05152020.py from github.com/zhouzilu/cTPnet
- SAVER-X-denoised baseline model or baseline correlation table for comparison

## Outputs

- Retrained cTP-net multiple-branch neural network on raw (undenoised) input
- Per-protein Pearson correlations between predicted and measured surface protein relative abundances
- Correlation retraction summary table (e.g. correlation_retraction_summary.csv) quantifying the ~0.02 drop

## How to apply

Preprocess the holdout CITE-seq UMI count matrix with Seurat default LogNormalize after applying the paper's quality filters (remove genes with <10 counts across cells and cells with <200 detected genes), skipping SAVER-X denoising to obtain the raw-input feature matrix. Retrain the cTP-net multiple-branch network using the shared training script (training_05152020.py), optimizing with Adam at learning rate 10e-5 for 139 epochs, holding out 10% of cells as the testing set and selecting the optimal model on testing error. Compute per-protein Pearson correlations between predicted and measured relative surface protein abundances on the holdout cells. Compare raw-input correlations to the SAVER-X-denoised baseline on the same validation cells (comparisons must be limited to the same validation cells to account for subset variation) and record the overall retraction, expected to be approximately 0.02 in correlation.

## Related tools

- **cTP-net** (Multiple-branch deep neural network retrained on raw vs. denoised input to impute surface protein abundances) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoising step being ablated; its removal defines the raw-input arm of the comparison)
- **Seurat** (Applies default LogNormalize transformation to (denoised or raw) counts and default quality filtering)
- **Adam (optimizer)** (Stochastic optimizer used for retraining, learning rate 10e-5 for 139 epochs)
- **Python** (Runtime for the shared training script training_05152020.py used to retrain the network) — https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py

## Examples

```
python training_05152020.py --input raw_lognormalize_matrix --epochs 139 --lr 10e-5 --test_ratio 0.1
```

## Evaluation signals

- Overall correlation retraction vs. the denoised baseline is approximately 0.02, larger than any other parameter tweak
- Per-protein Pearson correlation fields are present in the summary CSV for all evaluated surface proteins
- Baseline and ablated correlations are computed on the same validation/holdout cells to control for subset variation
- Training used Adam at learning rate 10e-5 for 139 epochs with a 90%/10% train/test cell split and model selection on testing error
- Quality filters applied: genes with <10 counts across cells and cells with <200 detected genes removed before normalization

## Limitations

- cTP-net only applies to count-based expression input (UMI-based); TPM/RPKM inputs are untested
- Benchmark comparisons are limited to the same cells as the validation set due to variation between subsets
- The reported ~0.02 retraction was established on CITE-seq/PBMC-type holdout data; generalization to non-hematopoietic lineages was not tested

## Evidence

- [discussion] SAVER-X denoising improves cTP-net prediction performance: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] Training hyperparameters: "The objective function was optimized stochastically with Adam46 with learning rate set to 10e–5 for 139 epochs"
- [discussion] Train/test split and model selection: "we keep only 10% of the cells as the testing set, and use the other 90% of the cells for training. The optimal model was selected based on the testing error."
- [discussion] Quality filter thresholds: "low-quality gene (<10 counts across cells) and low-quality cells (<200 genes detected) are removed"
- [readme] Denoising recommendation from README: "(1) **SAVERX**, we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance"
