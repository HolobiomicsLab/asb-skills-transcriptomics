---
name: cite-seq-protein-imputation-from-scrna
description: Use when you have a UMI-based scRNA-seq count matrix and want per-cell
  relative abundance estimates of surface protein immunophenotype markers, or when
  benchmarking imputation against raw/denoised RNA baselines or Seurat v3 anchor transfer.
when_to_use_negative:
- Expression input is not UMI-based count data (e.g. TPM/RPKM metrics) — cTP-net can
  only apply to count-based expression input (UMI-based)
- Target cells are outside the hematopoietic lineage — generalization of these immune-related
  markers to non-hematopoietic cells was not tested
- You need proteins beyond the ~24 immunophenotype markers in the released model —
  a model extending prediction to 100 surface proteins is still in training
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: cTP-net
  role: Core transfer learning neural network that imputes surface protein abundances
    from the normalized denoised expression matrix
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises scRNA-seq counts with default parameters before imputation; strongly
    recommended preprocessing step
- name: Seurat
  role: LogNormalize transformation of denoised counts; preprocess_seurat2/preprocess_seurat3
    and postprocess functions handle Seurat v2/v3 objects
  repo: https://github.com/zhouzilu/cTPnet
- name: Seurat v3
  role: Baseline anchor-based transfer-learning method for comparison against cTP-net
    imputation correlations
- name: Adam (optimizer)
  role: Optimizes the network objective stochastically with learning rate 10e-5 for
    139 epochs
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:432db01bdbefa9ada8fca63c8784230777acaa4683cba0c8943d932cd0e9c459
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - Adam (optimizer)
  - Python
  - R
  - SAVER-X
  - Seurat
  - Seurat v3
  - cTP-net
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# cite-seq-protein-imputation-from-scrna

## Summary

Imputes relative surface protein abundances from scRNA-seq count data using cTP-net, a transfer learning multiple-branch deep neural network pretrained on paired single-cell multi-omic (CITE-seq/REAP-seq) resources. It is applied after SAVER-X denoising and Seurat LogNormalize transformation of the UMI count matrix.

## When to use

Use when you have a UMI-based scRNA-seq count matrix and want per-cell relative abundance estimates of surface protein immunophenotype markers, or when benchmarking imputation against raw/denoised RNA baselines or Seurat v3 anchor transfer.

## When NOT to use

- Expression input is not UMI-based count data (e.g. TPM/RPKM metrics) — cTP-net can only apply to count-based expression input (UMI-based)
- Target cells are outside the hematopoietic lineage — generalization of these immune-related markers to non-hematopoietic cells was not tested
- You need proteins beyond the ~24 immunophenotype markers in the released model — a model extending prediction to 100 surface proteins is still in training

## Inputs

- UMI-based scRNA-seq count matrix (paired RNA/ADT CITE-seq data for evaluation, e.g. CITE-PBMC from Supplementary Table 1)
- Pretrained cTP-net model file path
- Target surface protein panel (dprotein argument)

## Outputs

- Per-cell relative surface protein abundance matrix (cTP-net imputed abundances)
- Per-protein cross-cell Pearson correlation table vs. measured CITE-seq protein levels for raw RNA, SAVER-X denoised RNA, and cTP-net conditions

## How to apply

First remove low-quality genes (<10 counts across cells) and low-quality cells (<200 genes detected), then denoise the scRNA-seq counts with SAVER-X using default parameters and transform the denoised counts with Seurat default LogNormalize — without SAVER-X denoising, cTP-net performance retracts by 0.02 in correlation. Run the pretrained cTP-net multiple-branch neural network (R/Python package from github.com/zhouzilu/cTPnet) on the normalized denoised matrix to predict relative surface protein abundances, filling genes present in training but not the test data with zeros to align feature spaces. For evaluation, compute per-protein cross-cell Pearson correlations between imputed and measured CITE-seq protein levels on held-out test cells, comparing against (1) raw RNA counts and (2) SAVER-X denoised RNA levels of the corresponding genes. For holdout validation, keep 10% of cells as testing (90% training) with model selection on testing error; for the out-of-cell-type scenario, partition cells by cell type and iteratively hold out all cells of one type. No GPU is required; given a dataset of 4000 cells the running time should be less than 1 minute.

## Related tools

- **cTP-net** (Core transfer learning neural network that imputes surface protein abundances from the normalized denoised expression matrix) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises scRNA-seq counts with default parameters before imputation; strongly recommended preprocessing step)
- **Seurat** (LogNormalize transformation of denoised counts; preprocess_seurat2/preprocess_seurat3 and postprocess functions handle Seurat v2/v3 objects) — https://github.com/zhouzilu/cTPnet
- **Seurat v3** (Baseline anchor-based transfer-learning method for comparison against cTP-net imputation correlations)
- **Adam (optimizer)** (Optimizes the network objective stochastically with learning rate 10e-5 for 139 epochs)

## Examples

```
cTPnet(data, data_type, model_file_path, dprotein)
```

## Evaluation signals

- Correlation table contains one row per benchmarked surface protein with three columns (raw RNA, SAVER-X denoised RNA, cTP-net), and cTP-net correlations exceed both RNA baselines as in Fig. 1b and Supplementary Figs. 3a/4a
- In the out-of-cell-type heatmap, cTP-net correlation entries substantially exceed Seurat v3 anchor-transfer entries (Fig. 2a / Supplementary Fig. 5)
- In the holdout scenario, cTP-net performs comparably to or slightly better than Seurat v3, while vastly improving in the out-of-cell-type scenario
- Network trained on transformed relative protein abundance (geometric mean log-ratio of ADT counts) yields better prediction accuracy than one trained on raw protein barcode counts
- Feature-space alignment check: genes in training but not in testing data are filled with zeros before prediction

## Limitations

- cTP-net can only apply to count-based expression input (UMI-based); accuracy with TPM/RPKM metrics is unknown since such CITE-seq data is not available for testing
- Generalization to cell types outside the hematopoietic lineage was not tested
- Without SAVER-X denoising, prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweak
- A model extending prediction to 100 surface proteins is still in training; the released model covers a smaller immunophenotype panel
- Comparisons across splits must be restricted to the same cells in the validation set to account for variations between subsets
- Windows users may hit ModuleNotFoundError for 'ctpnet'; use numpy 1.19.3, and running on Windows is generally not recommended

## Evidence

- [abstract] cTP-net imputation pipeline: "single cell Transcriptome to Protein prediction with deep neural network (cTP-net), to impute surface protein abundances from scRNA-seq data"
- [discussion] SAVER-X denoising rationale: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] Holdout split scheme: "we keep only 10% of the cells as the testing set, and use the other 90% of the cells for training. The optimal model was selected based on the testing error."
- [discussion] Zero-filling missing genes: "Genes that are in the training data but not in the testing data are filled with zeros."
- [discussion] UMI-only scope: "cTP-net can only apply to count-based expression input (UMI-based)."
- [readme] README pipeline overview: "This cTPnet package includes two analysis tools: (1) **SAVERX**, we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance, and (2) **cTP-net**"
