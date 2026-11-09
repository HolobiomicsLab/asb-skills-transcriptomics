---
name: preprocessing-for-protein-imputation
description: Use when you have raw UMI-based scRNA-seq counts plus paired ADT (CITE-seq/REAP-seq)
  protein counts and need the normalized denoised expression matrix and relative protein
  abundance response vectors required to run or train cTP-net.
when_to_use_negative:
- Expression input is TPM or RPKM — cTP-net can only apply to count-based UMI expression
  input, and CITE-seq data with TPM/RPKM metrics has unknown prediction accuracy
- The scRNA-seq data is already SAVER-X denoised and LogNormalized — re-denoising
  or double-normalizing would distort the Λ estimates
- You need raw CITE-seq protein measurements rather than imputed ones — this skill
  prepares inputs for imputation, not direct measurement
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
tools:
- name: SAVER-X
  role: Denoises the raw UMI scRNA-seq count matrix with default parameters to obtain
    denoised expression estimates Λ used as the network input
- name: Seurat
  role: Provides the default LogNormalize function applied to the SAVER-X denoised
    counts (scale factor 10000)
- name: cTP-net
  role: Package hosting the SAVERX wrapper and preprocessing functions (preprocess_matrix,
    preprocess_seurat2/3) that produce the imputation-ready inputs
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:8c701e95c6199038e977cccf1ae013ffdb044bbd652234a83f681bf2dc0dcb83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - R
  - SAVER-X
  - Seurat
  - cTP-net
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# preprocessing-for-protein-imputation

## Summary

Prepares paired scRNA-seq and CITE-seq/REAP-seq ADT data for cTP-net surface protein imputation by SAVER-X denoising of UMI counts, Seurat LogNormalize of the denoised matrix, and geometric-mean log-ratio transformation of protein counts.

## When to use

Use when you have raw UMI-based scRNA-seq counts plus paired ADT (CITE-seq/REAP-seq) protein counts and need the normalized denoised expression matrix and relative protein abundance response vectors required to run or train cTP-net.

## When NOT to use

- Expression input is TPM or RPKM — cTP-net can only apply to count-based UMI expression input, and CITE-seq data with TPM/RPKM metrics has unknown prediction accuracy
- The scRNA-seq data is already SAVER-X denoised and LogNormalized — re-denoising or double-normalizing would distort the Λ estimates
- You need raw CITE-seq protein measurements rather than imputed ones — this skill prepares inputs for imputation, not direct measurement

## Inputs

- Raw UMI-based scRNA-seq gene count matrix (e.g., 4000-cell cTPnet_demo dataset)
- Paired surface protein ADT count matrix (CITE-seq/REAP-seq antibody-derived tag counts)

## Outputs

- Denoised (SAVER-X Λ) and Seurat LogNormalize-transformed expression matrix X, log(Λ_ij * 10000 / m_j)
- Protein relative-abundance response matrix, ln(p_ic / geometric-mean) per cell
- Combined preprocessed_input.rds containing both matrices

## How to apply

Load the raw UMI-based scRNA-seq count matrix and the paired ADT count matrix (e.g., the cTPnet_demo dataset shipped with the package). Denoise the gene count matrix with SAVER-X using default parameters to obtain denoised estimates Λ — this matters because skipping SAVER-X retracts cTP-net correlation performance by 0.02. Apply the Seurat default LogNormalize transformation to the denoised counts, computing X_ij = log(Λ_ij * 10000 / m_j) where m_j is the total count for cell j (scale factor 10000), yielding the training input for the multiple-branch neural network. Transform each cell's ADT count vector p_ic into relative abundances via the Stoeckius et al. log-ratio transformation, ln(p_ic / geometric-mean), since the network trained on this transformed relative abundance yields better accuracy than training on raw protein barcode counts. Save the normalized denoised expression matrix and the transformed protein response matrix together (e.g., as preprocessed_input.rds) for downstream imputation.

## Related tools

- **SAVER-X** (Denoises the raw UMI scRNA-seq count matrix with default parameters to obtain denoised expression estimates Λ used as the network input)
- **Seurat** (Provides the default LogNormalize function applied to the SAVER-X denoised counts (scale factor 10000))
- **cTP-net** (Package hosting the SAVERX wrapper and preprocessing functions (preprocess_matrix, preprocess_seurat2/3) that produce the imputation-ready inputs) — https://github.com/zhouzilu/cTPnet

## Examples

```
preprocess_matrix(matrix_data, dprotein)
```

## Evaluation signals

- preprocessed_input.rds exists, is a valid RDS file, and contains both the expression and protein matrices with cell counts matching the input (e.g., the 4000-cell demo)
- Expression values equal X_ij = log(Λ_ij * 10000 / m_j), i.e., Seurat LogNormalize applied after SAVER-X, not to raw counts
- Protein response vectors contain no missing genes; genes present in training but missing from testing data are filled with zeros
- SAVER-X was run with default parameters; ablation check: omitting denoising should cost ~0.02 in correlation vs. CITE-seq measured abundance
- Protein responses are log-ratios relative to the geometric mean per cell (relative abundances), not raw ADT barcode counts

## Limitations

- cTP-net (and hence this preprocessing) only applies to UMI-based count expression input; accuracy for TPM/RPKM CITE-seq data is unknown because such data was not available for testing
- Without SAVER-X denoising, cTP-net prediction performance retracts by 0.02 in correlation, more than any other parameter tweak — so skipping this step degrades imputation
- Known demo failure mode: 'Error in dim(X) <- c(n, length(X)/n)' dims mismatch — set options(stringsAsFactors = FALSE) before running

## Evidence

- [discussion] scRNA data denoising was performed with SAVER-X using default parameters.: "scRNA data denoising was performed with SAVER-X using default parameters."
- [discussion] Denoised counts (Λ) were further transformed with Seurat default LogNormalize function: "Denoised counts (Λ) were further transformed with Seurat default LogNormalize function"
- [discussion] For the surface protein counts, we adopted the relative abundance transformation from Stoeckius et al.: "For the surface protein counts, we adopted the relative abundance transformation from Stoeckius et al."
- [discussion] The network trained using this transformed relative protein abundance as the response vector yields better prediction accuracy than the network trained using raw protein barcode counts.: "The network trained using this transformed relative protein abundance as the response vector yields better prediction accuracy than the network trained using raw protein barcode counts."
- [readme] we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance: "we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance"
