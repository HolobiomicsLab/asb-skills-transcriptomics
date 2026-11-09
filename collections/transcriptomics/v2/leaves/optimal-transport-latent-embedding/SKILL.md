---
name: optimal-transport-latent-embedding
description: Use when integrating unpaired single-cell datasets with both common and
  dataset-specific HVGs — e.g., scATAC gene activity matrices vs scRNA counts with
  unmatched cell types — and you need latent embeddings plus robustness metrics.
when_to_use_negative:
- Datasets are already paired cell-by-cell (vertical integration of the same cells
  profiled with multiple modalities is a different mode; default mode='h' assumes
  horizontal integration with common genes)
- The task is spatial transcriptomics deconvolution via the OT plan rather than latent
  embedding — uniPort's label transfer for spatial data uses the optimal transport
  plan 'instead of embedding latent space'
- Only batch-effect correction of a single modality is needed (methods like scMC or
  SCALEX target batch correction, not cross-modal integration)
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: uniPort
  role: 'Core framework: coupled-VAE + Minibatch-UOT via uniport.Run embeds both datasets
    in a shared latent space; uniport.metrics.label_transfer and uniport.metrics.silhouette
    score the integration'
  repo: https://github.com/caokai1073/uniport
- name: scanpy
  role: 'Preprocessing: scanpy.pp.normalize_total and scanpy.pp.log1p (offset 1) normalization,
    and HVG selection of k=2000 common and kx=ky=2000 dataset-specific genes'
- name: scikit-learn
  role: 'Evaluation: sklearn.neighbors.KNeighborsClassifier trained on reference cell-type
    annotations and UMAP coordinates for label-transfer metric computation'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:a04d1dbea6b47359529921296bb5698170768da64da1c9192670ea1b9fd2bad9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniport
  tools:
  - scanpy
  - scikit-learn
  - uniPort
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# optimal-transport-latent-embedding

## Summary

Integrate heterogeneous single-cell multi-omics datasets (e.g., mouse spleen scATAC gene activity and scRNA data) by training uniPort's coupled-VAE with Minibatch Unbalanced Optimal Transport to jointly embed both modalities in a shared latent space, then evaluate integration quality with label-transfer metrics (ARI, NMI, F1) and Silhouette coefficients.

## When to use

Use when integrating unpaired single-cell datasets with both common and dataset-specific HVGs — e.g., scATAC gene activity matrices vs scRNA counts with unmatched cell types — and you need latent embeddings plus robustness metrics.

## When NOT to use

- Datasets are already paired cell-by-cell (vertical integration of the same cells profiled with multiple modalities is a different mode; default mode='h' assumes horizontal integration with common genes)
- The task is spatial transcriptomics deconvolution via the OT plan rather than latent embedding — uniPort's label transfer for spatial data uses the optimal transport plan 'instead of embedding latent space'
- Only batch-effect correction of a single modality is needed (methods like scMC or SCALEX target batch correction, not cross-modal integration)

## Inputs

- AnnData matrices (.h5ad) of preprocessed scATAC gene activity scores (e.g., SnapATAC-processed, 3166 cells x 19,410 genes)
- AnnData matrix of preprocessed scRNA data (e.g., MultiMAP-processed mouse spleen scRNA)
- AnnData matrix containing the k=2000 highly variable common genes (adata_cm)
- Cell-type annotations for the reference dataset

## Outputs

- Shared latent-space cell embeddings stored in adata.obs['latent']
- Label-transfer metrics (ARI, NMI, F1) per dataset
- Silhouette coefficient of the integrated embeddings
- Total ARI+NMI+F1 score (e.g., written to ubm_integration_metrics.csv)
- Optional global OT plan when save_OT=True

## How to apply

Load both AnnData objects (e.g., SnapATAC-processed scATAC gene activity, 3166 cells x 19,410 genes, and MultiMAP-processed scRNA) and construct unbalanced settings (UBM-ATAC/UBM-RNA) by removing DC, Granulocyte, Macrophage, and NK populations from one modality. Preprocess with scanpy.pp.normalize_total, scanpy.pp.log1p (offset 1), select k=2000 highly variable common genes plus kx=ky=2000 dataset-specific HVGs, scale each gene to 0-1 with MaxAbsScaler (TFIDF_LSI available for scATAC). Call uniport.Run on the two datasets with default coupling settings (lambda_s=0.5, lambda_recon=1.0, lambda_kl=0.5, lambda_ot=1.0, iteration=30000) to embed both modalities in the shared latent space; train a kNN classifier (sklearn.neighbors.KNeighborsClassifier) on reference cell-type annotations and UMAP coordinates and apply uniport.metrics.label_transfer to obtain ARI, NMI, and F1, then compute uniport.metrics.silhouette on the integrated embeddings. Sum ARI+NMI+F1 per dataset and compare against reported values (2.225/0.676 for UBM-ATAC; 2.191/0.688 for UBM-RNA).

## Related tools

- **uniPort** (Core framework: coupled-VAE + Minibatch-UOT via uniport.Run embeds both datasets in a shared latent space; uniport.metrics.label_transfer and uniport.metrics.silhouette score the integration) — https://github.com/caokai1073/uniport
- **scanpy** (Preprocessing: scanpy.pp.normalize_total and scanpy.pp.log1p (offset 1) normalization, and HVG selection of k=2000 common and kx=ky=2000 dataset-specific genes)
- **scikit-learn** (Evaluation: sklearn.neighbors.KNeighborsClassifier trained on reference cell-type annotations and UMAP coordinates for label-transfer metric computation)

## Examples

```
import uniport as up; adata = up.Run(adatas=[adata_atac, adata_rna], adata_cm=adata_cm, lambda_s=0.5, lambda_recon=1.0, lambda_kl=0.5, lambda_ot=1.0, iteration=30000)
```

## Evaluation signals

- Reproduced totals: ARI+NMI+F1 ≈ 2.225 and Silhouette ≈ 0.676 for UBM-ATAC; total ≈ 2.191 and Silhouette ≈ 0.688 (third highest) for UBM-RNA
- Latent embeddings are present in adata.obs['latent'] after uniport.Run completes
- Preprocessing invariants: log-normalization used offset 1, k=2000 common HVGs and kx=ky=2000 dataset-specific HVGs selected, each gene scaled to 0-1 with MaxAbsScaler
- All cell types except the removed ones (DC, Granulocyte, Macrophage, NK) are represented in both modalities after UBM construction
- Label-transfer metrics computed via uniport.metrics.label_transfer after training the kNN classifier on reference annotations and UMAP coordinates

## Limitations

- Storing a dense full OT plan (save_OT=True) requires substantial memory; computing capacity with respect to memory should be ascertained before use
- Which hyperparameter combination from the searched grids (λ ∈ {0.5, 1.0, 5.0}, γ ∈ {0.5, 1.0}, λs ∈ {0.5, 1.0}) was selected for each dataset is not stated
- uniPort's scATAC integration relies on gene activity scores rather than directly modeling regulatory signals, unlike GLUE

## Evidence

- [abstract] integrate unpaired multiomics: "Here we introduce uniPort, a unified single-cell data integration framework that combines a coupled variational autoencoder (coupled-VAE) and minibatch unbalanced optimal transport (Minibatch-UOT)."
- [discussion] discussion workflow: "We identified k = 2000 highly variable common genes across cells of all datasets, and identified kx = ky = 2000 highly variable genes for each dataset"
- [discussion] discussion scaling: "We normalized values of each gene to the range of 0-1 within each dataset by the MaxAbsScaler function"
- [discussion] discussion evaluation: "we trained a k-Nearest-Neighbor (kNN) classifier by the sklearn.neighbors.KNeighborsClassifier function based on cell-type annotations and UMAP coordinates"
- [readme] readme invocation: "integration with both common and dataset-specific genes
# latent representation are stored in adata.obs['latent']
adata = up.Run(adatas=[adata1, adata2], adata_cm=adata_cm)"
- [readme] readme defaults: "lambda_s: balanced parameter for common and specific genes. Default: 0.5
+ lambda_recon: balanced parameter for reconstruct term. Default: 1.0"
