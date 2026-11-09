---
name: saverx-scrna-count-denoising
description: Use when raw UMI-based scRNA-seq count matrices (e.g., CITE-PBMC, CITE-CBMC,
  REAP-PBMC, HCA MantonCB1/MantonBM1, or van Galen AML counts in Seurat v3 format)
  will be fed to cTP-net protein imputation and must first be denoised with SAVER-X.
when_to_use_negative:
- Input expression values are already normalized metrics such as TPM or RPKM rather
  than UMI-based counts — the article states 'cTP-net can only apply to count-based
  expression input (UMI-based)', so denoising such inputs is out of scope
- The downstream analysis does not involve protein imputation or cTP-net and denoising
  assumptions (e.g., SAVER-X priors) are not appropriate for the method being applied
- Counts have already been denoised in a prior pass — re-denoising denoised counts
  is not part of the reported pipeline
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SAVER-X
  role: Primary denoising tool applied to raw UMI count matrices with default parameters
    before LogNormalize and cTP-net imputation
- name: cTP-net
  role: 'Downstream consumer: its cTPnet package bundles SAVERX as the recommended
    denoising step; denoised, LogNormalized counts are its training/prediction input'
  repo: https://github.com/zhouzilu/cTPnet
- name: Seurat
  role: Provides the default LogNormalize transformation applied to the SAVER-X denoised
    counts (Λ) to form the cTP-net input matrix
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:3fcaedccaf79d861f0b1d9679cdf9fb8863b3831ba42b1308ae02aba5d162075
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
  - cTP-net
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# SAVER-X scRNA-seq count denoising

## Summary

Denoises raw UMI-based scRNA-seq count matrices with SAVER-X (default parameters) to recover true expression levels (Λ) before downstream analysis; in this pipeline, denoising is the mandatory first step of the cTP-net surface-protein imputation workflow because omitting it retracts imputation correlation by 0.02.

## When to use

Use when raw UMI-based scRNA-seq count matrices (e.g., CITE-PBMC, CITE-CBMC, REAP-PBMC, HCA MantonCB1/MantonBM1, or van Galen AML counts in Seurat v3 format) will be fed to cTP-net protein imputation and must first be denoised with SAVER-X.

## When NOT to use

- Input expression values are already normalized metrics such as TPM or RPKM rather than UMI-based counts — the article states 'cTP-net can only apply to count-based expression input (UMI-based)', so denoising such inputs is out of scope
- The downstream analysis does not involve protein imputation or cTP-net and denoising assumptions (e.g., SAVER-X priors) are not appropriate for the method being applied
- Counts have already been denoised in a prior pass — re-denoising denoised counts is not part of the reported pipeline

## Inputs

- Raw UMI-based scRNA-seq count matrix (Seurat v2/v3 object or matrix; e.g., CITE-PBMC paired RNA counts, HCA MantonCB1/MantonBM1 count matrices, van Galen AML Seurat v3 count matrix)
- QC-filtered cell/gene set (low-quality genes <10 counts across cells and cells <200 genes detected already removed)

## Outputs

- SAVER-X denoised count matrix Λ, subsequently transformed via Seurat default LogNormalize into the normalized denoised matrix X that serves as cTP-net input

## How to apply

Take the raw scRNA-seq UMI count matrix (after standard QC removing low-quality genes with <10 counts across cells and low-quality cells with <200 genes detected) and run SAVER-X with default parameters — the article reports denoising was always 'performed with SAVER-X using default parameters', so no parameter tuning is expected. The denoised counts (Λ) are then further transformed with the Seurat default LogNormalize function, and this normalized denoised matrix X becomes the training/prediction input for the subsequent cTP-net multiple-branch neural network. The rationale is empirical: without SAVER-X denoising, cTP-net prediction performance retracts by 0.02 in correlation, a loss more significant than any other parameter tweak, so the decision point is simply denoise-before-impute vs. skip (always denoise). Fill genes present in training but absent from the testing data with zeros after denoising to align feature spaces. Apply this identically to every dataset in the panel (CITE-PBMC, CITE-CBMC, REAP-PBMC, HCA-CBMC, HCA-BMMC, AML) so that cross-tissue and cross-protocol comparisons are made on consistently preprocessed inputs.

## Related tools

- **SAVER-X** (Primary denoising tool applied to raw UMI count matrices with default parameters before LogNormalize and cTP-net imputation)
- **cTP-net** (Downstream consumer: its cTPnet package bundles SAVERX as the recommended denoising step; denoised, LogNormalized counts are its training/prediction input) — https://github.com/zhouzilu/cTPnet
- **Seurat** (Provides the default LogNormalize transformation applied to the SAVER-X denoised counts (Λ) to form the cTP-net input matrix)

## Evaluation signals

- Run SAVER-X with default parameters (no tuning); the article consistently reports 'scRNA data denoising was performed with SAVER-X using default parameters'
- Ablation check: downstream cTP-net prediction correlation computed with vs. without denoising should differ by ~0.02, favoring the denoised run — 'without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation'
- Schema check: output is a denoised counts matrix Λ that is immediately passed to Seurat LogNormalize, whose output X is the cTP-net input — 'The normalized denoised count matrix X is the training input for the subsequent multiple branch neural network'
- Input check: the matrix fed to SAVER-X is raw UMI-based counts (count-based expression input), not TPM/RPKM or log-normalized values
- Feature-space alignment: after denoising, genes present in training but absent in testing data are filled with zeros before imputation

## Limitations

- Omitting SAVER-X denoising retracts cTP-net prediction performance by 0.02 in correlation — more significant than any other parameter tweak — so skipping this step silently degrades all downstream imputation results
- Only count-based (UMI-based) expression input is supported; prediction accuracy for TPM/RPKM expression metrics is unknown since such CITE-seq data is not available for testing
- SAVER-X gains are modest (~0.02 correlation): denoised RNA alone still correlates far worse with measured protein than cTP-net output, so denoising must not be mistaken for the imputation itself

## Evidence

- [discussion] SAVER-X denoising improves cTP-net prediction performance; omission causes 0.02 correlation retraction: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] SAVER-X denoising always run with default parameters in the reported pipeline: "scRNA data denoising was performed with SAVER-X using default parameters."
- [discussion] Denoised counts Λ are transformed with Seurat default LogNormalize before use as cTP-net input: "Denoised counts (Λ) were further transformed with Seurat default LogNormalize function"
- [discussion] The normalized denoised matrix is the neural network's training input: "The normalized denoised count matrix X is the training input for the subsequent multiple branch neural network."
- [readme] README: SAVERX is bundled with cTP-net and strongly recommended before protein imputation: "This cTPnet package includes two analysis tools: (1) **SAVERX**, we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance"
