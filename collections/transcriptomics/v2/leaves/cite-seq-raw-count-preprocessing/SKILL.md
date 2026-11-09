---
name: cite-seq-raw-count-preprocessing
description: Use when a holdout CITE-seq UMI count matrix must be quality-filtered
  and LogNormalized without SAVER-X denoising to run the denoising-ablation of cTP-net
  and measure the correlation drop versus the denoised baseline.
when_to_use_negative:
- The goal is best-possible imputation accuracy rather than a denoising ablation —
  the README strongly recommends SAVER-X denoising before imputation, and skipping
  it costs ~0.02 correlation
- Input expression data is TPM or RPKM rather than UMI-based counts — cTP-net can
  only apply to count-based (UMI) expression input and TPM/RPKM accuracy is untested
- Counts have already been SAVER-X denoised and LogNormalized — running these filters
  again is redundant; the skill exists precisely for the raw-input branch of the comparison
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3308
tools:
- name: Seurat
  role: Provides the default LogNormalize transformation applied to the (undenoised)
    UMI counts after quality filtering
  repo: https://github.com/zhouzilu/cTPnet
- name: cTP-net
  role: Multiple-branch neural network retrained on the raw-input feature matrix to
    measure the performance retraction without denoising
  repo: https://github.com/zhouzilu/cTPnet
- name: Adam (optimizer)
  role: Stochastic optimizer used to retrain cTP-net on the raw input at learning
    rate 10e-5 for 139 epochs
- name: Python
  role: Environment for running the shared training script (training_05152020.py)
    that retrains cTP-net on raw input
  repo: https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py
- name: SAVER-X
  role: Denoising step deliberately skipped in this ablation branch to produce the
    raw-input comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:561c3a8b5337b8ca5543ab9d60b38d85015cf9086db4ca9a0b44a9cab734a549
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cite-seq-raw-count-preprocessing

## Summary

Preprocesses raw CITE-seq UMI count matrices with the cTP-net paper's quality filters and Seurat default LogNormalize while deliberately skipping SAVER-X denoising, so that the resulting raw-input feature matrix can be used to train cTP-net and quantify how much prediction performance depends on the denoising step (reported as ~0.02 correlation retraction).

## When to use

Use when a holdout CITE-seq UMI count matrix must be quality-filtered and LogNormalized without SAVER-X denoising to run the denoising-ablation of cTP-net and measure the correlation drop versus the denoised baseline.

## When NOT to use

- The goal is best-possible imputation accuracy rather than a denoising ablation — the README strongly recommends SAVER-X denoising before imputation, and skipping it costs ~0.02 correlation
- Input expression data is TPM or RPKM rather than UMI-based counts — cTP-net can only apply to count-based (UMI) expression input and TPM/RPKM accuracy is untested
- Counts have already been SAVER-X denoised and LogNormalized — running these filters again is redundant; the skill exists precisely for the raw-input branch of the comparison

## Inputs

- Raw CITE-seq UMI gene-by-cell count matrix (e.g. PBMC/CBMC holdout data)
- Measured surface protein ADT counts for the paired cells (for relative abundance transformation and Pearson correlation targets)
- Gene name annotations used to align feature spaces with the cTP-net training set

## Outputs

- Quality-filtered, LogNormalized raw (undenoised) RNA feature matrix input to cTP-net
- Per-protein Pearson correlations between predicted and measured surface protein relative abundances on holdout test cells
- Comparison table (e.g. correlation_retraction_summary.csv) of raw-input vs. SAVER-X-denoised baseline correlations with overall retraction (~0.02 expected)

## How to apply

Apply the paper's quality filters — remove genes with <10 counts across cells and cells with <200 detected genes — then transform the raw UMI counts with the Seurat default LogNormalize function, explicitly skipping the SAVER-X denoising step that the standard cTP-net pipeline uses. Retrain the cTP-net multiple-branch neural network on this raw-input matrix using the shared training script (extdata/training_05152020.py in the cTPnet repo), optimizing with Adam at learning rate 10e-5 for 139 epochs with a 90%/10% train/test cell split and model selection on testing error. Fill genes present in training but missing from testing data with zeros to align feature spaces, and train against the relative-abundance-transformed (geometric mean log-ratio) surface protein ADT counts. Compute per-protein Pearson correlations between cTP-net-predicted and measured surface protein relative abundances on the holdout test cells, then compare against the SAVER-X-denoised baseline on the same validation cells to quantify the per-protein and overall correlation retraction. The rationale: the Discussion reports that without SAVER-X denoising, cTP-net performance retracts by 0.02 in correlation — more than any other parameter tweak — making this a sensitive ablation for isolating the denoising contribution.

## Related tools

- **Seurat** (Provides the default LogNormalize transformation applied to the (undenoised) UMI counts after quality filtering) — https://github.com/zhouzilu/cTPnet
- **cTP-net** (Multiple-branch neural network retrained on the raw-input feature matrix to measure the performance retraction without denoising) — https://github.com/zhouzilu/cTPnet
- **Adam (optimizer)** (Stochastic optimizer used to retrain cTP-net on the raw input at learning rate 10e-5 for 139 epochs)
- **Python** (Environment for running the shared training script (training_05152020.py) that retrains cTP-net on raw input) — https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py
- **SAVER-X** (Denoising step deliberately skipped in this ablation branch to produce the raw-input comparison)

## Examples

```
cTPnet(data, data_type, model_file_path, dprotein)
```

## Evaluation signals

- The overall correlation retraction versus the SAVER-X-denoised baseline is approximately 0.02 and exceeds the effect of any other parameter tweak, consistent with the Discussion and Supplementary Table 5
- The quality filter removed genes with <10 counts across cells and cells with <200 detected genes before LogNormalize
- The training config matches the paper: Adam, learning rate 10e-5, 139 epochs, 90%/10% train/test split, model selected on testing error
- The summary CSV contains per-protein Pearson correlation fields for the evaluated surface proteins, computed on the same validation cells as the denoised baseline
- Genes present in training but missing from testing data are filled with zeros, keeping feature spaces aligned

## Limitations

- cTP-net can only apply to UMI-based count expression input; prediction accuracy for TPM/RPKM CITE-seq data is unknown
- Skipping SAVER-X denoising degrades performance by ~0.02 in correlation, so raw-input results are strictly an ablation baseline, not a recommended production pipeline
- Comparisons should be limited to the same cells that were in the validation set in the holdout scheme to account for variations between subsets
- On Windows the Python module 'ctpnet' may fail to import; numpy 1.19.3 is recommended, and non-Windows platforms are preferred

## Evidence

- [discussion] without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] low-quality gene (<10 counts across cells) and low-quality cells (<200 genes detected) are removed: "low-quality gene (<10 counts across cells) and low-quality cells (<200 genes detected) are removed"
- [discussion] Denoised counts (Λ) were further transformed with Seurat default LogNormalize function: "Denoised counts (Λ) were further transformed with Seurat default LogNormalize function"
- [discussion] we keep only 10% of the cells as the testing set, and use the other 90% of the cells for training. The optimal model was selected based on the testing error.: "we keep only 10% of the cells as the testing set, and use the other 90% of the cells for training. The optimal model was selected based on the testing error."
- [readme] we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance: "we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance"
