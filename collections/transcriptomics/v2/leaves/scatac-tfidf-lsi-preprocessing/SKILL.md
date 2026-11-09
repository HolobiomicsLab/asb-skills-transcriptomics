---
name: scatac-tfidf-lsi-preprocessing
description: Use when preparing scATAC-seq data (binary cell-by-bin accessibility
  matrix) to feed into cross-modality integration (e.g., with scRNA-seq via uniPort's
  coupled-VAE) alongside log-normalized gene expression.
when_to_use_negative:
- When the ATAC data will be summarized as gene activity scores computed by an external
  tool (e.g., SnapATAC) instead of processed from the peak matrix — the paper notes
  uniPort's scATAC integration is based on gene activity scores and tested alternatives
- When data lacks a peak/bin-level accessibility matrix (e.g., only pre-LSI embeddings
  or chromatin summary stats are available)
- When memory is severely constrained and save_OT=True dense OT plans are requested
  downstream — the paper warns dense T storage needs substantial memory
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3301
- http://edamontology.org/topic_3308
tools:
- name: uniPort
  role: Provides the TFIDF_LSI() function for scATAC preprocessing and the Run() function
    for downstream coupled-VAE + Minibatch-UOT integration
  repo: https://github.com/caokai1073/uniport
- name: scanpy
  role: Preprocesses the paired scRNA-seq side (normalize_total, log1p with offset
    1) so both modalities enter uniport.Run on comparable scales
- name: scikit-learn
  role: Supplies MaxAbsScaler for scaling gene values to the 0-1 range within each
    dataset and KNeighborsClassifier for downstream label transfer/metric evaluation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:542a7cafc3c6ce650711625ae57642679852808e8dc8aea6781ff18f72bd8cf7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniport
  tools:
  - scanpy
  - scikit-learn
  - uniPort
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3301
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# scatac-tfidf-lsi-preprocessing

## Summary

Preprocess single-cell ATAC-seq data for multi-omics integration by applying TF-IDF weighting to a binary cell-by-peak/bin accessibility matrix followed by latent semantic indexing (LSI) dimensionality reduction, as implemented in uniPort's TFIDF_LSI function.

## When to use

Use when preparing scATAC-seq data (binary cell-by-bin accessibility matrix) to feed into cross-modality integration (e.g., with scRNA-seq via uniPort's coupled-VAE) alongside log-normalized gene expression.

## When NOT to use

- When the ATAC data will be summarized as gene activity scores computed by an external tool (e.g., SnapATAC) instead of processed from the peak matrix — the paper notes uniPort's scATAC integration is based on gene activity scores and tested alternatives
- When data lacks a peak/bin-level accessibility matrix (e.g., only pre-LSI embeddings or chromatin summary stats are available)
- When memory is severely constrained and save_OT=True dense OT plans are requested downstream — the paper warns dense T storage needs substantial memory

## Inputs

- scATAC-seq count/accessibility matrix as an AnnData object (h5ad), binary cell-by-bin or cell-by-peak form
- Optionally paired scRNA-seq AnnData preprocessed with scanpy.pp.normalize_total and scanpy.pp.log1p (offset 1)

## Outputs

- TF-IDF weighted, LSI-reduced scATAC representation (AnnData with reduced latent features) ready for uniport.Run integration
- Integrated latent embedding with per-cell coordinates in adata.obs['latent'] when combined with scRNA data via uniport.Run

## How to apply

Start from a filtered scATAC count matrix (for PBMC data the paper filters out cells with fewer than 200 genes and genes observed in fewer than 3 cells), representing accessibility as a binary cell-by-bin matrix. Apply uniPort's TFIDF_LSI function, which first performs TF-IDF weighting on the binary accessibility matrix and then latent semantic indexing (LSI) to reduce dimensionality, producing an LSI-reduced representation suitable for joint integration with scanpy-preprocessed scRNA data (normalize_total, then log1p with offset of 1). Downstream, select k = 2000 highly variable common genes across datasets plus kx = ky = 2000 dataset-specific HVGs, and scale each gene to the 0-1 range within each dataset using MaxAbsScaler. The rationale is that TF-IDF plus LSI captures the dominant accessibility structure of sparse ATAC data in a form comparable to gene expression, enabling horizontal integration via uniport.Run(adatas=[...], adata_cm=...) with lambda_ot, lambda_kl (gamma), and lambda_s tuned from {0.5, 1.0, 5.0}, {0.5, 1.0}, {0.5, 1.0} respectively.

## Related tools

- **uniPort** (Provides the TFIDF_LSI() function for scATAC preprocessing and the Run() function for downstream coupled-VAE + Minibatch-UOT integration) — https://github.com/caokai1073/uniport
- **scanpy** (Preprocesses the paired scRNA-seq side (normalize_total, log1p with offset 1) so both modalities enter uniport.Run on comparable scales)
- **scikit-learn** (Supplies MaxAbsScaler for scaling gene values to the 0-1 range within each dataset and KNeighborsClassifier for downstream label transfer/metric evaluation)

## Examples

```
import uniport as up; adata_atac = up.TFIDF_LSI(adata_atac)  # then: adata = up.Run(adatas=[adata_rna, adata_atac], adata_cm=adata_cm)
```

## Evaluation signals

- Accessibility matrix must be binarized cell-by-bin before TF-IDF weighting, matching the paper's input formulation
- After LSI, downstream feature selection should yield k = 2000 highly variable common genes plus kx = ky = 2000 dataset-specific HVGs
- Gene values in each dataset must fall within the 0-1 range after MaxAbsScaler scaling
- On the paired 10x PBMC benchmark, the downstream integration should reach Silhouette ≈ 0.64, average FOSCTTM ≈ 0.0694, and ARI+NMI+F1 total ≈ 2.321
- Default training setup uses iteration = 30000; verify the Run call completed within the iteration budget before scoring metrics

## Limitations

- uniPort's scATAC integration is based on gene activity scores rather than directly modeling regulatory signals, unlike GLUE; results depend on how gene activity scores are calculated
- The hyperparameter grid (λ_ot ∈ {0.5, 1.0, 5.0}, γ ∈ {0.5, 1.0}, λ_s ∈ {0.5, 1.0}) does not document which combination was selected per dataset
- Storing a dense OT plan matrix T (save_OT=True) requires substantial memory and should be checked against computing capacity

## Evidence

- [methods] TFIDF_LSI for scATAC preprocessing: "add *TFIDF_LSI()* function for scATAC preprecess."
- [discussion] LSI on binary cell-by-bin accessibility with TF-IDF weighting: "latent semantic indexing (LSI) was used for dimensionality reduction, resulting in the binary cell-by-bin accessibility as input for TF-IDF weighting"
- [discussion] HVG counts for integration: "We identified k = 2000 highly variable common genes across cells of all datasets, and identified kx = ky = 2000 highly variable genes for each dataset"
- [readme] Install uniPort: "The **uniport** package can be installed via pip3:"
