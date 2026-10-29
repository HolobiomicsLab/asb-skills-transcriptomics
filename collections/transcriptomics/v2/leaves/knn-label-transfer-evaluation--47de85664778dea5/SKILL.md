---
name: knn-label-transfer-evaluation
description: Use when you have uniPort latent embeddings from two heterogeneous single-cell
  datasets (e.g. scATAC gene activity vs scRNA) and need quantitative ARI+NMI+F1 and
  Silhouette metrics to judge integration quality, including unbalanced-matching cases
  where cell types are missing from one modality.
when_to_use_negative:
- Input data are already paired/vertical integration of the same cells, where label
  transfer via UOT mapping is unnecessary
- The goal is deconvolution of spatial transcriptomic data using the OT plan itself
  rather than embedding-space label transfer metrics
- Reference cell-type annotations are absent, since the kNN classifier requires labeled
  reference cells
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: uniPort
  role: Provides uniport.Run for joint embedding and uniport.metrics.label_transfer
    / uniport.metrics.silhouette for ARI, NMI, F1, and Silhouette evaluation
  repo: https://github.com/caokai1073/uniport
- name: scikit-learn
  role: sklearn.neighbors.KNeighborsClassifier trained on reference cell-type annotations
    and UMAP coordinates for label transfer
- name: scanpy
  role: Preprocessing of AnnData inputs (scanpy.pp.normalize_total, scanpy.pp.log1p)
    before integration and evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:0244e505aa6eaa0591f2154ab7672e30503866dc544a8fc186646528ae2ee9d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniport
  tools:
  - scanpy
  - scikit-learn
  - uniPort
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# knn-label-transfer-evaluation

## Summary

Evaluate uniPort-integrated single-cell multi-omics embeddings by training a k-Nearest-Neighbor classifier (sklearn.neighbors.KNeighborsClassifier) on reference cell-type annotations and UMAP coordinates, then computing ARI, NMI, and F1 label-transfer scores plus Silhouette coefficients on the shared latent space.

## When to use

Use when you have uniPort latent embeddings from two heterogeneous single-cell datasets (e.g. scATAC gene activity vs scRNA) and need quantitative ARI+NMI+F1 and Silhouette metrics to judge integration quality, including unbalanced-matching cases where cell types are missing from one modality.

## When NOT to use

- Input data are already paired/vertical integration of the same cells, where label transfer via UOT mapping is unnecessary
- The goal is deconvolution of spatial transcriptomic data using the OT plan itself rather than embedding-space label transfer metrics
- Reference cell-type annotations are absent, since the kNN classifier requires labeled reference cells

## Inputs

- uniPort latent embeddings (adata.obs['latent']) for query and reference datasets from uniport.Run
- Reference dataset cell-type annotations
- UMAP coordinates of the reference embeddings
- AnnData (.h5ad) preprocessed matrices: k=2000 highly variable common genes plus kx=ky=2000 dataset-specific HVGs, MaxAbsScaler-normalized to 0-1 per gene

## Outputs

- ARI, NMI, and F1 scores from uniport.metrics.label_transfer for each dataset/condition
- Silhouette coefficient from uniport.metrics.silhouette for each integrated embedding
- Summary table (e.g. ubm_integration_metrics.csv) with total ARI+NMI+F1 and Silhouette per condition

## How to apply

After running uniport.Run with default coupling settings (lambda_s=0.5, lambda_recon=1.0, lambda_kl=0.5, lambda_ot=1.0, iteration=30000) to embed both modalities in the shared latent space, train a kNN classifier using sklearn.neighbors.KNeighborsClassifier based on cell-type annotations and UMAP coordinates of the reference dataset. Apply the classifier to the query modality's embeddings via uniport.metrics.label_transfer to obtain ARI, NMI, and F1 per dataset. Then compute uniport.metrics.silhouette on the integrated embeddings for each condition. Sum ARI+NMI+F1 per dataset (e.g. mouse spleen UBM-ATAC and UBM-RNA, where DC, Granulocyte, Macrophage and NK cells are removed from scATAC or scRNA respectively) and write all metrics to a CSV such as ubm_integration_metrics.csv. Validate against expected totals of approximately 2.225 (UBM-ATAC) and 2.191 (UBM-RNA) and Silhouette coefficients of approximately 0.676 and 0.688.

## Related tools

- **uniPort** (Provides uniport.Run for joint embedding and uniport.metrics.label_transfer / uniport.metrics.silhouette for ARI, NMI, F1, and Silhouette evaluation) — https://github.com/caokai1073/uniport
- **scikit-learn** (sklearn.neighbors.KNeighborsClassifier trained on reference cell-type annotations and UMAP coordinates for label transfer)
- **scanpy** (Preprocessing of AnnData inputs (scanpy.pp.normalize_total, scanpy.pp.log1p) before integration and evaluation)

## Examples

```
adata = up.Run(adatas=[adata1, adata2], adata_cm=adata_cm); from sklearn.neighbors import KNeighborsClassifier; knn = KNeighborsClassifier().fit(ref_umap, ref_labels); up.metrics.label_transfer(knn, query_umap)
```

## Evaluation signals

- Total ARI+NMI+F1 for UBM-ATAC reproduces approximately 2.225 and for UBM-RNA approximately 2.191
- Silhouette coefficient on integrated embeddings is approximately 0.676 (UBM-ATAC) and 0.688 (UBM-RNA)
- Metrics for all conditions are written to ubm_integration_metrics.csv with per-dataset ARI, NMI, F1, Silhouette, and total
- Latent embeddings exist in adata.obs['latent'] after uniport.Run completes with default settings (lambda_s=0.5, lambda_ot=1.0, iteration=30000)
- Label transfer uses the kNN classifier trained on the reference domain (ref_id defaults to the domain_id of the last dataset)

## Limitations

- uniPort's scATAC integration relies on gene activity scores rather than directly modeling regulatory signals, which may affect label transfer accuracy compared to methods like GLUE
- Which hyperparameter combination from the searched grids (lambda in {0.5, 1.0, 5.0}, gamma in {0.5, 1.0}, lambdas in {0.5, 1.0}) was selected for each dataset is not stated
- Storing a dense full OT plan (save_OT=True) requires substantial memory; computing capacity with respect to memory should be ascertained before using this option

## Evidence

- [discussion] we trained a k-Nearest-Neighbor (kNN) classifier by the sklearn.neighbors.KNeighborsClassifier function based on cell-type annotations and UMAP coordinates: "we trained a k-Nearest-Neighbor (kNN) classifier by the sklearn.neighbors.KNeighborsClassifier function based on cell-type annotations and UMAP coordinates"
- [abstract] uniPort provides a flexible label transfer framework to deconvolute heterogeneous spatial transcriptomic data using an optimal transport plan: "uniPort provides a flexible label transfer framework to deconvolute heterogeneous spatial transcriptomic data using an optimal transport plan"
- [readme] latent representation are stored in adata.obs['latent']: "integration with both common and dataset-specific genes
# latent representation are stored in adata.obs['latent']
adata = up.Run(adatas=[adata1, adata2], adata_cm=adata_cm)"
- [readme] lambda_s: balanced parameter for common and specific genes. Default: 0.5: "lambda_s: balanced parameter for common and specific genes. Default: 0.5
+ **lambda_recon**: balanced parameter for reconstruct term. Default: 1.0"
