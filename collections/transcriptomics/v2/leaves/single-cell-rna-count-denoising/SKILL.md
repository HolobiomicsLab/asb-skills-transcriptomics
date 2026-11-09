---
name: single-cell-rna-count-denoising
description: Use when raw UMI-based scRNA-seq counts (e.g., CITE-seq/REAP-seq RNA
  matrices, cTPnet_demo 4000-cell demo) must be denoised before protein imputation
  or other downstream modeling.
when_to_use_negative:
- Expression data are already TPM- or RPKM-normalized or otherwise non-UMI — cTP-net
  can only apply to count-based (UMI) input, and accuracy on TPM/RPKM CITE-seq data
  is untested
- The matrix has already been denoised/normalized (e.g., an existing LogNormalize'd
  Seurat object) and only needs downstream imputation
- The goal is protein imputation itself rather than the denoising/normalization preprocessing
  stage
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SAVER-X
  role: Performs the denoising of the UMI-based scRNA-seq count matrix (default parameters)
    to produce Λ; skipping it reduces cTP-net correlation by 0.02
- name: cTP-net
  role: 'Downstream consumer: its SAVERX package tool is strongly recommended for
    denoising before surface protein imputation; demo data and vignettes live here'
  repo: https://github.com/zhouzilu/cTPnet
- name: Seurat
  role: Supplies the default LogNormalize transformation applied to the SAVER-X denoised
    counts Λ
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:754b2cceebdb839f878d53200580a307292b79adac7731bc89b54c0bf79d3d81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - R
  - SAVER-X
  - Seurat
  - cTP-net
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# single-cell-rna-count-denoising

## Summary

Denoises raw UMI-based scRNA-seq count matrices with SAVER-X (default parameters) to obtain more accurate transcript relative-abundance estimates, as the recommended first step of the cTP-net surface-protein-imputation pipeline before Seurat LogNormalize.

## When to use

Use when raw UMI-based scRNA-seq counts (e.g., CITE-seq/REAP-seq RNA matrices, cTPnet_demo 4000-cell demo) must be denoised before protein imputation or other downstream modeling.

## When NOT to use

- Expression data are already TPM- or RPKM-normalized or otherwise non-UMI — cTP-net can only apply to count-based (UMI) input, and accuracy on TPM/RPKM CITE-seq data is untested
- The matrix has already been denoised/normalized (e.g., an existing LogNormalize'd Seurat object) and only needs downstream imputation
- The goal is protein imputation itself rather than the denoising/normalization preprocessing stage

## Inputs

- Raw UMI-based scRNA-seq count matrix (genes x cells), e.g., from the cTPnet_demo dataset shipped with the cTPnet R package
- Optionally, the paired ADT (surface protein) count matrix if continuing into cTP-net preprocessing

## Outputs

- Denoised expression estimate matrix Λ (same cells/genes as input)
- Seurat LogNormalize'd matrix X with scale factor 10000 (X_ij = log(Λ_ij * 10000 / m_j))
- preprocessed_input.rds holding the normalized denoised expression matrix together with the transformed protein abundance matrix

## How to apply

Load the raw UMI-based scRNA-seq count matrix and run SAVER-X with default parameters to produce the denoised expression estimates Λ; skipping this step retracts cTP-net correlation by 0.02, more than any other parameter tweak. Transform Λ with the Seurat default LogNormalize function, computing X_ij = log(Λ_ij * 10000 / m_j) where m_j is the total count for cell j (scale factor 10000). The input must be count-based (UMI) expression, since cTP-net cannot be applied to TPM/RPKM metrics. In the cTP-net package the denoising is exposed via the SAVERX tool, which the authors strongly recommend running before imputation. Save the denoised and normalized matrix (e.g., preprocessed_input.rds) alongside the geometric-mean log-ratio-transformed ADT protein counts for network training.

## Related tools

- **SAVER-X** (Performs the denoising of the UMI-based scRNA-seq count matrix (default parameters) to produce Λ; skipping it reduces cTP-net correlation by 0.02)
- **cTP-net** (Downstream consumer: its SAVERX package tool is strongly recommended for denoising before surface protein imputation; demo data and vignettes live here) — https://github.com/zhouzilu/cTPnet
- **Seurat** (Supplies the default LogNormalize transformation applied to the SAVER-X denoised counts Λ)

## Evaluation signals

- Verify preprocessed_input.rds exists, is a valid RDS file, and contains matrices whose cell count matches the input (e.g., 4000 cells in the demo)
- Check the denoised matrix Λ has the same gene set as the raw UMI count matrix, with response vectors containing no missing genes
- Confirm the input expression was UMI-based counts — cTP-net can only apply to count-based (UMI) input, not TPM/RPKM
- Verify LogNormalize used scale factor 10000: X_ij = log(Λ_ij * 10000 / m_j) with m_j = total count for cell j
- Benchmark sanity: omitting SAVER-X denoising should retract prediction correlation by ~0.02, more than any other parameter tweak — a trained-with-denoising model should outperform this

## Limitations

- SAVER-X denoising is strongly recommended but not mandatory; without it cTP-net prediction performance retracts by 0.02 in correlation
- Denoising is only meaningful for UMI-based count data; TPM/RPKM expression metrics are unsupported and their accuracy is unknown
- On Windows the cTP-net pipeline can hit Python/numPy issues (e.g., requires numpy 1.19.3), and non-Windows platforms are recommended

## Evidence

- [discussion] SAVER-X denoising with default parameters: "scRNA data denoising was performed with SAVER-X using default parameters."
- [discussion] LogNormalize of denoised counts: "Denoised counts (Λ) were further transformed with Seurat default LogNormalize function"
- [discussion] Impact of skipping denoising: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] UMI count input requirement: "cTP-net can only apply to count-based expression input (UMI-based)."
- [readme] README recommendation to denoise first: "This cTPnet package includes two analysis tools: (1) **SAVERX**, we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance"
