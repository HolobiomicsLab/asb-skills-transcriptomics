---
name: clustering-silhouette-assessment
description: Use when integration quality of uniPort-embedded scATAC/scRNA (or other
  single-cell) datasets must be quantified by Silhouette plus ARI, NMI and F1 to compare
  modality-linking or cell-type-removal scenarios.
when_to_use_negative:
- When cells are paired across modalities and you need vertical integration (mode='v')
  with matched-cell evaluation rather than label-transfer-free clustering metrics
- When only the global OT plan is needed (save_OT=True) and no cell-type/clustering
  assessment is required
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3934
tools:
- name: uniPort
  role: Embeds datasets via coupled-VAE + Minibatch-UOT and provides metrics.silhouette
    and metrics.label_transfer for evaluation
  repo: https://github.com/caokai1073/uniport
- name: scanpy
  role: Preprocessing (scanpy.pp.normalize_total, scanpy.pp.log1p offset 1, HVG selection)
    before integration and metric computation
- name: scikit-learn
  role: KNeighborsClassifier trained on reference annotations and UMAP coordinates
    for label-transfer-based ARI/NMI/F1
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:543da9fc3c06c5e8178049634f07e9340cfab4f1a373480a2b8c888b0421ca7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniport
  tools:
  - scanpy
  - scikit-learn
  - uniPort
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3934
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# clustering-silhouette-assessment

## Summary

Evaluates uniPort multi-omics integration quality by computing the Silhouette coefficient on the shared latent-space embeddings and combining it with ARI+NMI+F1 from kNN label transfer, e.g. reaching 0.676/0.688 Silhouette on UBM-ATAC/UBM-RNA mouse spleen scATAC-scRNA integrations.

## When to use

Use when integration quality of uniPort-embedded scATAC/scRNA (or other single-cell) datasets must be quantified by Silhouette plus ARI, NMI and F1 to compare modality-linking or cell-type-removal scenarios.

## When NOT to use

- When cells are paired across modalities and you need vertical integration (mode='v') with matched-cell evaluation rather than label-transfer-free clustering metrics
- When only the global OT plan is needed (save_OT=True) and no cell-type/clustering assessment is required

## Inputs

- uniPort shared-latent AnnData embeddings (adata.obs['latent']) from uniport.Run over two modalities (e.g. mouse spleen scATAC gene activity 3166x19410 and MultiMAP-processed scRNA)
- Reference cell-type annotations and UMAP coordinates for the reference dataset (e.g. scRNA with DC, Granulocyte, Macrophage, NK removed or retained)
- Preprocessed common-gene AnnData (adata_cm) with k=2000 common HVGs and dataset-specific HVG AnnData objects

## Outputs

- ARI, NMI, F1 per dataset from uniport.metrics.label_transfer (kNN-based)
- Silhouette coefficient per integration scenario from uniport.metrics.silhouette
- Combined ARI+NMI+F1 totals (e.g. written to ubm_integration_metrics.csv)

## How to apply

Preprocess each modality per uniPort conventions (scanpy.pp.normalize_total, scanpy.pp.log1p with offset 1, k=2000 highly variable common genes plus kx=ky=2000 dataset-specific HVGs, MaxAbsScaler 0-1 gene scaling) before running uniport.Run (lambda_s=0.5, lambda_recon=1.0, lambda_kl=0.5, lambda_ot=1.0, iteration=30000) to embed both datasets in the shared latent space. Train a sklearn.neighbors.KNeighborsClassifier on reference cell-type annotations and UMAP coordinates, then apply uniport.metrics.label_transfer to obtain ARI, NMI and F1 for each dataset. Compute uniport.metrics.silhouette on the resulting integrated embeddings for each scenario (e.g. UBM-ATAC and UBM-RNA). Sum ARI+NMI+F1 per dataset and report alongside the Silhouette coefficient; totals of ~2.225 (UBM-ATAC) and ~2.191 (UBM-RNA) with Silhouette ~0.676 and ~0.688 validate the pipeline.

## Related tools

- **uniPort** (Embeds datasets via coupled-VAE + Minibatch-UOT and provides metrics.silhouette and metrics.label_transfer for evaluation) — https://github.com/caokai1073/uniport
- **scanpy** (Preprocessing (scanpy.pp.normalize_total, scanpy.pp.log1p offset 1, HVG selection) before integration and metric computation)
- **scikit-learn** (KNeighborsClassifier trained on reference annotations and UMAP coordinates for label-transfer-based ARI/NMI/F1)

## Examples

```
adata = up.Run(adatas=[adata1, adata2], adata_cm=adata_cm); sil = up.metrics.silhouette(adata)
```

## Evaluation signals

- UBM-ATAC total ARI+NMI+F1 ≈ 2.225 and Silhouette ≈ 0.676
- UBM-RNA total ARI+NMI+F1 ≈ 2.191 and Silhouette ≈ 0.688 (third highest Silhouette among compared methods)
- Latent embeddings are stored in adata.obs['latent'] and are non-empty for both domains after Run
- Metrics are computed on embeddings from the default coupling settings (lambda_s=0.5, lambda_recon=1.0, lambda_kl=0.5, lambda_ot=1.0, iteration=30000)
- All metrics for both scenarios are present in ubm_integration_metrics.csv

## Limitations

- Storing a dense OT matrix T (save_OT=True) needs substantial memory; verify memory capacity before use
- Hyperparameters were chosen from grids λ ∈ {0.5, 1.0, 5.0}, γ ∈ {0.5, 1.0}, λs ∈ {0.5, 1.0}; which combination was selected per dataset is not stated
- uniPort scATAC integration relies on gene activity scores rather than directly modeling regulatory signals

## Evidence

- [abstract] uniPort jointly embeds heterogeneous single-cell multi-omics datasets into a shared latent space.: "uniPort jointly embeds heterogeneous single-cell multi-omics datasets into a shared latent space."
- [discussion] kNN classifier for label transfer: "we trained a k-Nearest-Neighbor (kNN) classifier by the sklearn.neighbors.KNeighborsClassifier function based on cell-type annotations and UMAP coordinates"
- [discussion] Preprocessing thresholds: "We identified k = 2000 highly variable common genes across cells of all datasets, and identified kx = ky = 2000 highly variable genes for each dataset"
- [readme] Latent output location: [evidence span withheld — anchor status: external_doc]
