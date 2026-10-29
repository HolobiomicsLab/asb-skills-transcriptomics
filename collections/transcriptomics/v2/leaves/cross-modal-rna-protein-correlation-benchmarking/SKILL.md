---
name: cross-modal-rna-protein-correlation-benchmarking
description: Use when paired scRNA-seq/ADT data are available and you must quantify
  whether a prediction (or RNA signal itself) tracks measured protein levels cell-by-cell
  — e.g. comparing imputation methods or raw/denoised RNA baselines per protein.
when_to_use_negative:
- The input expression data are not UMI-based counts — the article notes cTP-net can
  only apply to count-based (UMI) input and that accuracy for TPM/RPKM CITE-seq data
  is untested
- You want absolute protein quantification — the benchmark evaluates relative surface
  protein abundances via cross-cell correlation, not absolute concentrations
- The held-out cells are from cell types unrelated to the training data — generalization
  was only tested for cell types absent from but related to those in the training
  data, and not outside the hematopoietic lineage
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_3934
tools:
- name: cTP-net
  role: Imputes relative surface protein abundances from normalized denoised RNA to
    serve as the test track being benchmarked against measured ADT
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises scRNA-seq counts (default parameters) to provide the denoised-RNA
    baseline track and the recommended pre-processing before imputation
- name: Seurat
  role: Applies default LogNormalize to the SAVER-X-denoised counts before imputation
    and baseline computation
- name: R
  role: Implementation environment for the cTPnet package and benchmark vignettes
  repo: https://github.com/zhouzilu/cTPnet
- name: Python
  role: Alternative implementation environment for the cTPnet package
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:583dcd804a59e372c64bb5eb6cdc7c669de9e9c1e2b993b8cc4b6e7907368a5e
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
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Cross-modal RNA–protein correlation benchmarking

## Summary

Benchmarks imputed or RNA-derived estimates of surface protein abundance against measured CITE-seq/REAP-seq ADT counts by computing per-protein cross-cell Pearson correlations on held-out cells. It was used to show cTP-net imputation beats raw and SAVER-X-denoised RNA baselines.

## When to use

Use when paired scRNA-seq/ADT data are available and you must quantify whether a prediction (or RNA signal itself) tracks measured protein levels cell-by-cell — e.g. comparing imputation methods or raw/denoised RNA baselines per protein.

## When NOT to use

- The input expression data are not UMI-based counts — the article notes cTP-net can only apply to count-based (UMI) input and that accuracy for TPM/RPKM CITE-seq data is untested
- You want absolute protein quantification — the benchmark evaluates relative surface protein abundances via cross-cell correlation, not absolute concentrations
- The held-out cells are from cell types unrelated to the training data — generalization was only tested for cell types absent from but related to those in the training data, and not outside the hematopoietic lineage

## Inputs

- Paired CITE-seq/REAP-seq dataset: scRNA-seq UMI count matrix plus ADT protein barcode counts (e.g. CITE-PBMC from Supplementary Table 1)
- Raw RNA counts for genes corresponding to the benchmarked surface proteins
- SAVER-X denoised RNA counts (SAVER-X defaults, then Seurat LogNormalize)
- Imputed surface protein abundances for the same held-out cells (e.g. cTP-net predictions)

## Outputs

- Per-protein Pearson correlation table: one row per benchmarked surface protein, three columns (raw RNA, denoised RNA, imputed protein) vs. measured ADT on held-out cells
- Comparison showing whether imputed abundances exceed raw- and denoised-RNA baselines for each protein

## How to apply

Load the paired RNA count matrix and ADT protein barcode counts (e.g. CITE-PBMC from Supplementary Table 1), then split cells 90/10 into training and testing sets, selecting the model on testing error. For the test cells, build three per-protein prediction tracks: (1) the raw RNA count of the encoding gene, (2) the SAVER-X-denoised RNA level (denoise with default parameters, then transform with Seurat default LogNormalize), and (3) the imputed relative surface protein abundance (e.g. from the pretrained cTP-net multiple-branch neural network run on the normalized denoised test-set expression matrix). Compute per-protein cross-cell Pearson correlations of each track against the measured CITE-seq protein abundances on the held-out cells only — comparisons are limited to the same cells in the validation set to account for variations between subsets. The result is a correlation table with one row per benchmarked surface protein and three correlation columns, reproducible as in Fig. 1b and Supplementary Figs. 3a/4a.

## Related tools

- **cTP-net** (Imputes relative surface protein abundances from normalized denoised RNA to serve as the test track being benchmarked against measured ADT) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises scRNA-seq counts (default parameters) to provide the denoised-RNA baseline track and the recommended pre-processing before imputation)
- **Seurat** (Applies default LogNormalize to the SAVER-X-denoised counts before imputation and baseline computation)
- **R** (Implementation environment for the cTPnet package and benchmark vignettes) — https://github.com/zhouzilu/cTPnet
- **Python** (Alternative implementation environment for the cTPnet package) — https://github.com/zhouzilu/cTPnet

## Examples

```
pred <- cTPnet(data = X_norm, data_type = "matrix", model_file_path = model_path, dprotein = 24)
```

## Evaluation signals

- Correlation table has exactly one row per benchmarked surface protein and three correlation columns (raw RNA, denoised RNA, imputed protein)
- For all examined proteins, cTP-net imputed abundances yield higher cross-cell Pearson correlation to measured protein levels than both raw and SAVER-X-denoised RNA counts, as in Fig. 1b and Supplementary Figs. 3a/4a
- Correlations are computed only on held-out test cells (10% holdout); baseline comparisons restricted to the same validation-set cells across methods
- Removing SAVER-X denoising before imputation should lower imputation correlation by ~0.02, more than any other parameter tweak — a sanity check that the denoised-RNA baseline was constructed correctly
- Held-out RNA/protein features align to the training feature space: genes in training but not in testing data are filled with zeros

## Limitations

- Comparisons are limited to the same cells in the validation set to account for variations between subsets
- cTP-net (and thus the imputed track) only applies to count-based, UMI expression input; prediction accuracy for TPM/RPKM CITE-seq data is unknown since such data were not available for testing
- Generalization outside the hematopoietic lineage was not tested, and generalization to cell types outside but related to the training data is only partially validated
- The pretrained model covers a limited panel (24 immunophenotype markers); a model extending to 100 surface proteins was still in training
- Windows is not recommended for running cTP-net; numpy 1.19.3 may be required to avoid ModuleNotFoundError for the ctpnet module

## Evidence

- [results] Under the 90/10 holdout validation on CITE-seq PBMC data, cTP-net imputed abundances have much higher correlation to measured protein levels than both the denoised and raw RNA counts: [evidence span withheld — anchor status: not_found]
- [methods] Compute per-protein cross-cell Pearson correlations against measured CITE-seq protein abundances on the held-out test cells for the three conditions: [evidence span withheld — anchor status: not_found]
- [discussion] we limit comparisons to the same cells that were in the validation set in the holdout scheme to account for variations between subsets: "we limit comparisons to the same cells that were in the validation set in the holdout scheme to account for variations between subsets."
- [discussion] Denoised counts (Λ) were further transformed with Seurat default LogNormalize function: "Denoised counts (Λ) were further transformed with Seurat default LogNormalize function"
- [discussion] without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] cTP-net can only apply to count-based expression input (UMI-based): "cTP-net can only apply to count-based expression input (UMI-based)."
- [discussion] Genes that are in the training data but not in the testing data are filled with zeros: "Genes that are in the training data but not in the testing data are filled with zeros."
