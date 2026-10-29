---
name: log-normalize-count-transformation
description: Use when UMI-based scRNA-seq counts must be log-normalized (scale factor
  10000) after SAVER-X denoising, or CITE-seq/REAP-seq ADT protein counts must be
  converted to relative-abundance log-ratios before cTP-net protein imputation.
when_to_use_negative:
- Expression data are TPM or RPKM rather than UMI-based counts — cTP-net can only
  apply to count-based (UMI) input and TPM/RPKM accuracy is untested
- Goal is imputing proteins without paired multi-omic training data — LogNormalize
  alone does not predict protein abundance
- Input is already normalized/denoised — re-applying LogNormalize to non-count values
  invalidates the per-cell total m_j in X_ij = log(Λ_ij × 10000 / m_j)
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SAVER-X
  role: Denoises the scRNA-seq UMI counts (default parameters) to produce Λ before
    LogNormalize is applied
- name: Seurat
  role: Provides the default LogNormalize function applied to the denoised counts
    Λ
- name: cTP-net
  role: Consumes the normalized denoised expression matrix X and transformed protein
    response vectors as training input
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:be53111311b37ddd6a199ccb5bd992d5ac0729159e13ac750b78c23e7c14104a
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# log-normalize-count-transformation

## Summary

Seurat-default LogNormalize of SAVER-X-denoised UMI counts (X_ij = log(Λ_ij × 10000 / m_j)) paired with the Stoeckius et al. relative-abundance log-ratio transform of surface-protein ADT counts, producing the paired training input and response vectors for cTP-net.

## When to use

Use when UMI-based scRNA-seq counts must be log-normalized (scale factor 10000) after SAVER-X denoising, or CITE-seq/REAP-seq ADT protein counts must be converted to relative-abundance log-ratios before cTP-net protein imputation.

## When NOT to use

- Expression data are TPM or RPKM rather than UMI-based counts — cTP-net can only apply to count-based (UMI) input and TPM/RPKM accuracy is untested
- Goal is imputing proteins without paired multi-omic training data — LogNormalize alone does not predict protein abundance
- Input is already normalized/denoised — re-applying LogNormalize to non-count values invalidates the per-cell total m_j in X_ij = log(Λ_ij × 10000 / m_j)

## Inputs

- raw UMI-based scRNA-seq count matrix (required by cTP-net)
- SAVER-X denoised expression estimates Λ
- paired ADT surface-protein count matrix (e.g., CITE-seq / REAP-seq protein barcode counts)

## Outputs

- LogNormalize-scaled denoised expression matrix X with scale factor 10000
- relative-abundance transformed protein response vectors ln(p_ic / geometric mean)
- paired normalized expression and transformed protein matrices (e.g., preprocessed_input.rds)

## How to apply

First denoise the raw UMI count matrix with SAVER-X using default parameters to obtain the denoised expression estimates Λ. Then apply the Seurat default LogNormalize transformation to Λ, computing X_ij = log(Λ_ij × 10000 / m_j), where m_j is the total count for cell j (scale factor 10000); this normalized denoised matrix X is the training input for the multiple-branch neural network. For surface protein ADT counts p_ic, adopt the relative abundance transformation from Stoeckius et al., ln(p_ic / geometric mean across proteins within each cell), yielding the response vectors. Train on the transformed relative protein abundance rather than raw protein barcode counts, since the transformed target yields better prediction accuracy. Genes present in training data but missing from testing data are filled with zeros to align feature spaces.

## Related tools

- **SAVER-X** (Denoises the scRNA-seq UMI counts (default parameters) to produce Λ before LogNormalize is applied)
- **Seurat** (Provides the default LogNormalize function applied to the denoised counts Λ)
- **cTP-net** (Consumes the normalized denoised expression matrix X and transformed protein response vectors as training input) — https://github.com/zhouzilu/cTPnet

## Examples

```
seu <- NormalizeData(seu, normalization.method = "LogNormalize", scale.factor = 10000)  # applied to SAVER-X denoised counts Λ
```

## Evaluation signals

- Each normalized entry equals log(Λ_ij × 10000 / m_j) with the scale factor fixed at 10000 and m_j equal to cell j's total denoised count
- After the Stoeckius log-ratio transform, the geometric mean of transformed protein values within each cell is 0 (ratios have geometric mean 1)
- Normalized expression matrix and transformed protein matrix contain matching cells (e.g., 4000 cells in the cTPnet_demo dataset) with no missing genes in the response vectors
- Downstream cTP-net correlation should be ~0.02 higher than the same run without SAVER-X denoising, since skipping denoising retracts performance by 0.02 in correlation
- Training on transformed relative protein abundance should outperform training on raw protein barcode counts

## Limitations

- cTP-net (and hence this transformation pipeline) can only apply to count-based expression input (UMI-based); accuracy for TPM/RPKM CITE-seq data is unknown
- Skipping SAVER-X denoising retracts cTP-net prediction performance by 0.02 in correlation, more than any other parameter tweak
- Genes in training data absent from testing data must be zero-filled to align feature spaces before normalization-comparable matrices are used

## Evidence

- [discussion] Denoising before LogNormalize: "scRNA data denoising was performed with SAVER-X using default parameters."
- [discussion] LogNormalize of denoised counts: "Denoised counts (Λ) were further transformed with Seurat default LogNormalize function"
- [discussion] ADT relative abundance transform: "For the surface protein counts, we adopted the relative abundance transformation from Stoeckius et al."
- [discussion] Transformed target outperforms raw counts: "The network trained using this transformed relative protein abundance as the response vector yields better prediction accuracy than the network trained using raw protein barcode counts."
- [discussion] UMI-only input requirement: "cTP-net can only apply to count-based expression input (UMI-based)."
- [methods] Package bundles SAVERX denoising step: "This cTPnet package includes two analysis tools: (1) **SAVERX**, we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance"
