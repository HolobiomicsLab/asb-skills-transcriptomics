---
name: saverx-denoising-ablation-benchmarking
description: Use when you need to measure how much SAVER-X denoising contributes to
  cTP-net protein-imputation accuracy, or to compare the denoising ablation against
  other parameter tweaks on a CITE-seq holdout benchmark.
when_to_use_negative:
- Input is already SAVER-X-denoised or you only want to run the standard cTP-net imputation
  pipeline — this skill deliberately skips denoising
- Expression input is TPM/RPKM rather than UMI-based counts — cTP-net can only apply
  to count-based (UMI) expression input
- You need raw cTP-net protein predictions for downstream use rather than a denoising-vs-raw
  benchmark comparison
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: cTP-net
  role: Multiple-branch neural network retrained on raw (undenoised) input and used
    to predict relative surface protein abundances
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoising step being ablated; its inclusion/exclusion defines the compared
    arms of the benchmark
- name: Seurat
  role: Provides the default LogNormalize transformation applied to counts in both
    arms of the comparison
- name: Adam (optimizer)
  role: Stochastic optimizer used to retrain the network (learning rate 10e-5, 139
    epochs)
- name: Python
  role: Language for the shared training script (training_05152020.py) used to retrain
    the ablated model
  repo: https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:775584298b195b3feaf9b23b58962b2bfb0a9be33fada791f9b909c30d39e585
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# saverx-denoising-ablation-benchmarking

## Summary

Ablation benchmark for cTP-net that replaces the SAVER-X denoising step with raw (undenoised) UMI input to quantify how much imputation performance retracts (~0.02 Pearson correlation, larger than any other parameter tweak). It quantifies the contribution of RNA denoising to surface protein imputation accuracy.

## When to use

Use when you need to measure how much SAVER-X denoising contributes to cTP-net protein-imputation accuracy, or to compare the denoising ablation against other parameter tweaks on a CITE-seq holdout benchmark.

## When NOT to use

- Input is already SAVER-X-denoised or you only want to run the standard cTP-net imputation pipeline — this skill deliberately skips denoising
- Expression input is TPM/RPKM rather than UMI-based counts — cTP-net can only apply to count-based (UMI) expression input
- You need raw cTP-net protein predictions for downstream use rather than a denoising-vs-raw benchmark comparison

## Inputs

- Holdout CITE-seq UMI count matrix (raw scRNA-seq counts plus measured ADT/protein counts)
- SAVER-X-denoised baseline correlation results on the same validation cells
- Pretrained/initial cTP-net model files and protein panel gene list (e.g. gene_names_protein12)

## Outputs

- Per-protein Pearson correlations for the raw-input cTP-net predictions on holdout test cells
- Overall correlation retraction value (expected ~0.02) versus the denoised baseline
- Comparison table (e.g. correlation_retraction_summary.csv) with per-protein correlation fields

## How to apply

Preprocess the holdout CITE-seq UMI count matrix with the paper's quality filters (remove genes with <10 counts across cells and cells with <200 detected genes), apply Seurat default LogNormalize, but skip SAVER-X denoising to obtain the raw-input feature matrix. Retrain the cTP-net multiple-branch neural network using the shared training script (training_05152020.py), optimizing with Adam at learning rate 10e-5 for 139 epochs, keeping 10% of cells as the testing set and 90% for training, with model selection on testing error. Compute per-protein Pearson correlations between predicted and measured relative surface protein abundances on the holdout test cells, then compare against the SAVER-X-denoised baseline correlations on the same validation cells. The rationale is that the paper reports a 0.02 correlation retraction without denoising, exceeding the effect of any other parameter tweak, so the comparison isolates the denoising contribution.

## Related tools

- **cTP-net** (Multiple-branch neural network retrained on raw (undenoised) input and used to predict relative surface protein abundances) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoising step being ablated; its inclusion/exclusion defines the compared arms of the benchmark)
- **Seurat** (Provides the default LogNormalize transformation applied to counts in both arms of the comparison)
- **Adam (optimizer)** (Stochastic optimizer used to retrain the network (learning rate 10e-5, 139 epochs))
- **Python** (Language for the shared training script (training_05152020.py) used to retrain the ablated model) — https://github.com/zhouzilu/cTPnet/blob/master/extdata/training_05152020.py

## Examples

```
git clone https://github.com/zhouzilu/cTPnet && python cTPnet/extdata/training_05152020.py  # retrain cTP-net on raw (skipping SAVER-X) input, lr=10e-5, 139 epochs
```

## Evaluation signals

- The overall correlation retraction in the summary must be approximately 0.02, consistent with the value stated in the Discussion and Supplementary Table 5
- The denoising ablation must show a larger correlation retraction than any other parameter tweak tested
- The output CSV must contain per-protein Pearson correlation fields for the evaluated surface proteins
- Both raw-input and denoised-baseline correlations must be computed on the same validation/holdout cells to account for variations between subsets
- Preprocessing must retain the paper's filters: genes with <10 counts across cells and cells with <200 detected genes removed

## Limitations

- Comparisons in the benchmark are limited to the same cells as in the validation set due to variations between subsets
- cTP-net can only apply to count-based (UMI) expression input, so the ablation cannot be extended to TPM/RPKM data
- SAVER-X denoising is strongly recommended by the developers, so the raw-input model is an ablation reference, not the intended production configuration

## Evidence

- [discussion] Reported retraction from removing SAVER-X: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] Baseline preprocessing transform: "Denoised counts (Λ) were further transformed with Seurat default LogNormalize function"
- [discussion] Quality filters applied before the ablation: "low-quality gene (<10 counts across cells) and low-quality cells (<200 genes detected) are removed"
- [discussion] Training/evaluation split and model selection: "we keep only 10% of the cells as the testing set, and use the other 90% of the cells for training. The optimal model was selected based on the testing error."
- [readme] Developer emphasis on denoising: "we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance"
- [discussion] Optimizer settings for retraining: "The objective function was optimized stochastically with Adam46 with learning rate set to 10e–5 for 139 epochs"
