---
name: single-cell-rna-spatial-data-integration
description: Use when integrating scRNA-seq reference data with spatial transcriptomics
  (e.g., STARmap, MERFISH, 10x Visium) to obtain shared latent embeddings or deconvolve
  spatial spot cell-type proportions via an optimal transport plan.
when_to_use_negative:
- Cells are paired/profiled from the same cells and you only need vertical integration
  without OT-plan deconvolution
- The datasets share no genes and you need diagonal integration with dataset-specific
  modeling only
- Memory cannot hold the dense global OT plan matrix — skip save_OT or use embedding-based
  label transfer instead
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: uniPort
  role: 'Main framework: coupled-VAE + Minibatch-UOT integration and OT-plan-based
    label transfer for spatial deconvolution (up.Run with save_OT=True, uniport label_transfer)'
  repo: https://github.com/caokai1073/uniport
- name: scanpy
  role: 'Preprocessing: normalize_total, log1p with offset 1, HVG selection (k=2000)
    before integration'
- name: scikit-learn
  role: MaxAbsScaler for 0-1 gene scaling and kNN classifier (sklearn.neighbors.KNeighborsClassifier)
    for label-transfer metric evaluation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:e81af0b637b06e865c81c16603d370e62751a571babd57e44bb20efd4ab64d8c
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-rna-spatial-data-integration

## Summary

Integrates heterogeneous single-cell datasets (scRNA-seq with spatial transcriptomics) by combining a coupled variational autoencoder with Minibatch Unbalanced Optimal Transport (Minibatch-UOT) in the uniPort framework, enabling joint embedding and OT-plan-based spatial deconvolution of spot cell-type proportions.

## When to use

Use when integrating scRNA-seq reference data with spatial transcriptomics (e.g., STARmap, MERFISH, 10x Visium) to obtain shared latent embeddings or deconvolve spatial spot cell-type proportions via an optimal transport plan.

## When NOT to use

- Cells are paired/profiled from the same cells and you only need vertical integration without OT-plan deconvolution
- The datasets share no genes and you need diagonal integration with dataset-specific modeling only
- Memory cannot hold the dense global OT plan matrix — skip save_OT or use embedding-based label transfer instead

## Inputs

- scRNA-seq reference AnnData (e.g., synthetic STARmap: 14249 cells x 34041 genes) with cell-type annotations
- Spatial spot/cell AnnData (e.g., STARmap spots: 189 spots x 882 genes)
- AnnData of common highly variable genes across datasets (adata_cm, k=2000)
- Optional ground-truth spot cell-type proportion vectors for benchmarking

## Outputs

- Shared latent cell embeddings (adata.obs['latent']) jointly embedding both datasets
- Global Minibatch-UOT optimal transport plan (when save_OT=True)
- Deconvoluted per-spot cell-type proportion vectors
- Benchmark metrics: PCC, RMSE, JSD (and optionally SSIM) against baselines

## How to apply

Preprocess each AnnData with scanpy: apply normalize_total, log1p with offset 1, scale gene values to 0-1 per dataset with MaxAbsScaler, and select k=2000 highly variable common genes plus kx=ky=2000 dataset-specific HVGs. Train the coupled-VAE and Minibatch-UOT via up.Run(adatas=[scRNA, spots], adata_cm=common_HVGs) with default hyperparameters (lambda_s=0.5, lambda_recon=1.0, lambda_kl=0.5, lambda_ot=1.0, iteration=30000); pass save_OT=True to also output a global OT plan (requires memory for a dense matrix). For spatial deconvolution, map the OT plan rows to scRNA reference cell-type annotations using uniport label_transfer to predict spot-level cell-type proportions. Evaluate deconvolution accuracy by computing PCC, RMSE, and JSD between predicted and ground-truth spot proportion vectors; on the synthetic STARmap benchmark (14249 cells x 34041 genes scRNA; 189 spots x 882 genes spots) uniPort achieved PCC 0.449, RMSE 0.157, JSD 0.569, between SpaOTsc (PCC 0.409) and Tangram (PCC 0.619).

## Related tools

- **uniPort** (Main framework: coupled-VAE + Minibatch-UOT integration and OT-plan-based label transfer for spatial deconvolution (up.Run with save_OT=True, uniport label_transfer)) — https://github.com/caokai1073/uniport
- **scanpy** (Preprocessing: normalize_total, log1p with offset 1, HVG selection (k=2000) before integration)
- **scikit-learn** (MaxAbsScaler for 0-1 gene scaling and kNN classifier (sklearn.neighbors.KNeighborsClassifier) for label-transfer metric evaluation)

## Examples

```
adata, OT = up.Run(adatas=[scRNA, spots], adata_cm=adata_cm, save_OT=True)
```

## Evaluation signals

- Reproduced STARmap deconvolution metrics fall at/near PCC ~0.449, RMSE ~0.157, JSD ~0.569
- Benchmark comparison table contains rows for uniPort, Tangram (PCC 0.619), and SpaOTsc (PCC 0.409)
- Latent embeddings for both datasets co-embed in the shared latent space (adata.obs['latent'] populated for all cells)
- OT plan rows sum consistently against reference cell-type annotations when mapped via label_transfer
- Preprocessed inputs satisfy the pipeline invariants: normalized_total, log1p(offset=1), MaxAbsScaler 0-1 per gene, k=2000 common HVGs

## Limitations

- Storing the dense full OT plan T (save_OT=True) requires substantial memory; verify memory capacity beforehand
- Which hyperparameter combination from the searched grids (lambda in {0.5,1.0,5.0}, gamma in {0.5,1.0}, lambda_s in {0.5,1.0}) was selected per dataset is not stated
- On the synthetic STARmap deconvolution task uniPort underperformed Tangram (PCC 0.449 vs 0.619), so it is competitive but not always top-performing
- scATAC integration relies on gene activity scores rather than directly modeling regulatory signals

## Evidence

- [abstract] coupled-VAE and Minibatch-UOT: "Here we introduce uniPort, a unified single-cell data integration framework that combines a coupled variational autoencoder (coupled-VAE) and minibatch unbalanced optimal transport (Minibatch-UOT)."
- [abstract] OT-plan-based deconvolution: "uniPort provides a flexible label transfer framework to deconvolute heterogeneous spatial transcriptomic data using an optimal transport plan, instead of embedding latent space."
- [discussion] preprocessing steps: "We normalized total counts of each cell using the scanpy.pp.normalize_total function in the scanpy package"
- [discussion] HVG selection: "We identified k = 2000 highly variable common genes across cells of all datasets, and identified kx = ky = 2000 highly variable genes for each dataset"
- [discussion] scaling: "We normalized values of each gene to the range of 0-1 within each dataset by the MaxAbsScaler function"
- [discussion] STARmap dataset dimensions: [evidence span withheld — anchor status: not_found]
- [readme] memory caveat: "save_OT**: if True, output a global OT plan. Need more memory. Default: False"
- [readme] default hyperparameters: "lambda_s**: balanced parameter for common and specific genes. Default: 0.5"
