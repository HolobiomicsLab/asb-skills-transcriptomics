---
name: holdout-train-test-split-validation
description: Use when benchmarking protein-imputation from paired CITE-seq RNA/ADT
  data requires a random cell-level train/test split, with model selection on testing
  error and per-protein correlation reporting.
when_to_use_negative:
- Prediction for cell types absent from training data — use the out-of-cell-type scheme
  that partitions cells by cell type and holds out all cells of one type instead of
  a random split
- Non-UMI expression input such as TPM or RPKM — cTP-net can only apply to count-based
  (UMI) expression input
- Cells outside the hematopoietic lineage — the paper states imputation of these immune-related
  markers was not tried there
edam_operation: http://edamontology.org/operation_0570
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: cTP-net
  role: Imputes relative surface protein abundances on the held-out test cells via
    a multiple-branch neural network
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises scRNA-seq counts (default parameters) before imputation; without
    it performance retracts by 0.02 in correlation
- name: Seurat
  role: Transforms denoised counts (Λ) with the default LogNormalize function to form
    the network input matrix
- name: R
  role: Runtime for the cTPnet package, Seurat preprocessing, and correlation computation
  repo: https://github.com/zhouzilu/cTPnet
- name: Python
  role: Alternative runtime for cTPnet and training scripts
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:8a6c848128e5a704c02c077f58929f0d45f1aa9466269f8c2eade5e458db31d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - Python
  - R
  - SAVER-X
  - Seurat
  - cTP-net
  edam_operation: http://edamontology.org/operation_0570
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# holdout-train-test-split-validation

## Summary

Evaluates surface-protein imputation methods by randomly holding out 10% of cells as a testing set and training on the remaining 90%, then comparing per-protein Pearson correlations of imputed abundances against raw and SAVER-X-denoised RNA baselines. This is the paper's 90/10 random holdout scheme for validating cTP-net on CITE-seq PBMC data.

## When to use

Use when benchmarking protein-imputation from paired CITE-seq RNA/ADT data requires a random cell-level train/test split, with model selection on testing error and per-protein correlation reporting.

## When NOT to use

- Prediction for cell types absent from training data — use the out-of-cell-type scheme that partitions cells by cell type and holds out all cells of one type instead of a random split
- Non-UMI expression input such as TPM or RPKM — cTP-net can only apply to count-based (UMI) expression input
- Cells outside the hematopoietic lineage — the paper states imputation of these immune-related markers was not tried there

## Inputs

- CITE-seq paired RNA count matrix and ADT protein barcode counts (e.g. CITE-PBMC dataset)
- Cell-level split assignment (90% training / 10% testing)
- Pretrained cTP-net model files

## Outputs

- Per-protein table of cross-cell Pearson correlations on held-out test cells with three columns: raw RNA count, SAVER-X denoised RNA, and cTP-net predicted abundance
- Optimal cTP-net model selected on testing error
- cTP-net imputed relative surface protein abundances for held-out cells

## How to apply

Split cells so 10% are kept as the testing set and 90% are used for training, selecting the optimal model based on testing error. Denoise the training and test scRNA-seq counts with SAVER-X using default parameters, then transform denoised counts with Seurat's default LogNormalize function to produce the normalized denoised count matrix X. Run the pretrained cTP-net multiple-branch neural network on the test-set expression matrix to impute relative surface protein abundances. Fill genes present in training but absent from testing data with zeros to align the feature spaces. Compute per-protein cross-cell Pearson correlations on the held-out cells for three conditions: raw RNA count, SAVER-X denoised RNA level, and cTP-net predicted protein abundance. Restrict method comparisons to the same cells in the validation set to account for variation between subsets.

## Related tools

- **cTP-net** (Imputes relative surface protein abundances on the held-out test cells via a multiple-branch neural network) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises scRNA-seq counts (default parameters) before imputation; without it performance retracts by 0.02 in correlation)
- **Seurat** (Transforms denoised counts (Λ) with the default LogNormalize function to form the network input matrix)
- **R** (Runtime for the cTPnet package, Seurat preprocessing, and correlation computation) — https://github.com/zhouzilu/cTPnet
- **Python** (Alternative runtime for cTPnet and training scripts) — https://github.com/zhouzilu/cTPnet

## Examples

```
R: cTPnet(data, data_type, model_file_path, dprotein)  # after preprocess_seurat2/3 and SAVER-X denoising, evaluate on the 10% held-out cells
```

## Evaluation signals

- Correlation table has one row per benchmarked surface protein with exactly three correlation columns (raw RNA, denoised RNA, cTP-net)
- cTP-net Pearson correlations exceed both the raw-RNA and SAVER-X-denoised-RNA baselines for all benchmarked proteins, as in Fig. 1b and Supplementary Figs. 3a/4a
- Model-selection trajectory shows the final model was chosen by testing error on the 10% held-out cells
- All methods are compared on the identical validation-set cells to control for subset variation
- Feature-space check: any gene in training but not in testing data is zero-filled before prediction

## Limitations

- Comparisons are limited to the same cells that were in the validation set in the holdout scheme to account for variations between subsets
- SAVER-X denoising matters: without it, cTP-net prediction performance retracts by 0.02 in correlation, more than any other parameter tweak
- Random holdout does not test generalization to unseen cell types; the out-of-cell-type scheme (holding out all cells of one type) addresses that separately
- Prediction accuracy for CITE-seq data with TPM and RPKM expression metrics is unknown — such data were not available for testing

## Evidence

- [discussion] 90/10 split and testing-error model selection: "we keep only 10% of the cells as the testing set, and use the other 90% of the cells for training. The optimal model was selected based on the testing error."
- [discussion] Same-cells comparison constraint: "we limit comparisons to the same cells that were in the validation set in the holdout scheme to account for variations between subsets."
- [methods] Quality filtering before splitting/benchmarking: "low-quality gene (<10 counts across cells) and low-quality cells (<200 genes detected) are removed"
- [readme] Practical runtime envelope for the benchmark: "Given a dataset of 4000 cells, the running time of cTP-net should be less than 1 mins. No GPU required for running this algorithm."
- [methods] cTPnet function signature: "`cTPnet(data, data_type, model_file_path, dprotein)`"
