---
name: adt-relative-abundance-log-ratio-transformation
description: Use when CITE-seq or REAP-seq ADT/protein count matrices must be converted
  to relative-abundance training targets for cTP-net, or whenever per-cell protein
  log-ratios (vs. geometric mean) are needed instead of raw barcode counts.
when_to_use_negative:
- Input RNA is TPM/RPKM rather than UMI-based counts — cTP-net can only apply to count-based
  (UMI) expression input, so paired imputation is out of scope
- The goal is normalizing the RNA side — that uses SAVER-X denoising followed by Seurat
  LogNormalize, not the ADT log-ratio transform
- Imputing proteins for cells outside the hematopoietic lineage — the authors state
  this was not tested
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: cTP-net
  role: R/Python package whose preprocessing functions (preprocess_seurat2, preprocess_seurat3,
    preprocess_matrix) apply the Stoeckius et al. relative abundance log-ratio transformation
    to ADT counts
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:dd427c3992cf8ef4dc20076b65679b6d420b50b5501e09da8154087b75011998
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - R
  - SAVER-X
  - cTP-net
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# adt-relative-abundance-log-ratio-transformation

## Summary

Transforms raw CITE-seq/REAP-seq surface protein (ADT) barcode counts per cell into relative abundances via the Stoeckius et al. log-ratio transformation, ln(p_ic / geometric mean), producing the response vectors cTP-net trains against. This transformed target yields better prediction accuracy than raw barcode counts.

## When to use

Use when CITE-seq or REAP-seq ADT/protein count matrices must be converted to relative-abundance training targets for cTP-net, or whenever per-cell protein log-ratios (vs. geometric mean) are needed instead of raw barcode counts.

## When NOT to use

- Input RNA is TPM/RPKM rather than UMI-based counts — cTP-net can only apply to count-based (UMI) expression input, so paired imputation is out of scope
- The goal is normalizing the RNA side — that uses SAVER-X denoising followed by Seurat LogNormalize, not the ADT log-ratio transform
- Imputing proteins for cells outside the hematopoietic lineage — the authors state this was not tested

## Inputs

- Raw ADT/protein barcode count matrix (cells x proteins) from CITE-seq or REAP-seq, e.g., the 24 immunophenotype markers
- Optionally a Seurat v2/v3 object or plain matrix holding the paired scRNA-seq counts (UMI-based) that the protein vectors must align with
- Protein panel definition (dprotein) as used by cTPnet preprocessing functions

## Outputs

- Transformed relative protein abundance matrix (per-cell log-ratio values ln(p_ic / geometric mean)) used as cTP-net training response vectors
- Optionally, a combined preprocessed_input.rds holding both the normalized denoised expression matrix and the transformed protein abundance matrix

## How to apply

For each cell j, take the raw ADT count vector p_ic across the surface protein panel (e.g., the 24 immunophenotype markers) and compute the relative abundance as ln(p_ic divided by the geometric mean of that cell's protein counts), following the Stoeckius et al. transformation. In the cTPnet R package this is handled by the preprocessing utilities preprocess_seurat2, preprocess_seurat3, or preprocess_matrix (the dprotein argument selects the protein panel). Run this on paired multi-omic data so the transformed protein vectors align cell-by-cell with the SAVER-X-denoised, Seurat LogNormalize RNA matrix (X_ij = log(Lambda_ij * 10000 / m_j)) that serves as the network input. The rationale, shown in the paper, is that the network trained using this transformed relative protein abundance as the response vector yields better prediction accuracy than training on raw protein barcode counts.

## Related tools

- **cTP-net** (R/Python package whose preprocessing functions (preprocess_seurat2, preprocess_seurat3, preprocess_matrix) apply the Stoeckius et al. relative abundance log-ratio transformation to ADT counts) — https://github.com/zhouzilu/cTPnet

## Examples

```
adt_norm <- preprocess_matrix(adt_count_matrix, dprotein)
```

## Evaluation signals

- Each transformed protein value equals ln of the raw count divided by that cell's geometric-mean protein count, so transformed vectors are centered per cell
- The transformed protein matrix must contain the same number of cells as the input counts (e.g., the 4000-cell cTPnet_demo dataset) with no missing proteins in the response vectors
- If combined with RNA preprocessing, preprocessed_input.rds must be a valid RDS containing both matrices with matching cell counts
- Downstream check: a cTP-net model trained on the transformed relative abundances should outperform one trained on raw protein barcode counts

## Limitations

- cTP-net only applies to UMI-based count expression input; accuracy for TPM/RPKM metric data is unknown since such CITE-seq data was not available for testing
- The shipped models impute 24 immunophenotype markers; a model extending prediction to 100 surface proteins was still in training
- Generalization was benchmarked on PBMC/CBMC immune cells and AML (hematopoietic lineage); imputation on non-hematopoietic cells is untested

## Evidence

- [discussion] For the surface protein counts, we adopted the relative abundance transformation from Stoeckius et al.: "For the surface protein counts, we adopted the relative abundance transformation from Stoeckius et al."
- [discussion] The network trained using this transformed relative protein abundance as the response vector yields better prediction accuracy than the network trained using raw protein barcode counts.: "The network trained using this transformed relative protein abundance as the response vector yields better prediction accuracy than the network trained using raw protein barcode counts."
- [readme] we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance: "we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance"
