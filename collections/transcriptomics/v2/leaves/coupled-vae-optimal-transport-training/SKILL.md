---
name: coupled-vae-optimal-transport-training
description: Use when integrating heterogeneous single-cell datasets (e.g. paired
  10x PBMC scATAC/scRNA, MERFISH + scRNA) with nonlinear cross-modality deformations,
  or when common genes alone are insufficient and dataset-specific HVGs must be retained.
when_to_use_negative:
- Input datasets are unimodal and batch-matched where a linear method such as Harmony
  suffices
- You need to model regulatory signals of scATAC-seq directly — uniPort's scATAC integration
  relies on gene activity scores rather than directly modeling regulatory signals
  (as GLUE does)
- No highly variable common genes exist between datasets and vertical/paired structure
  is absent, requiring careful mode='d' justification rather than default integration
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: uniPort
  role: Provides the coupled-VAE + Minibatch-UOT trainer (uniport.Run), TFIDF_LSI
    scATAC preprocessing, filter_data/batch_scale utilities, and gene-imputation decoding
    via out='predict'
  repo: https://github.com/caokai1073/uniport
- name: scanpy
  role: Normalizes total counts per cell (scanpy.pp.normalize_total) and performs
    log-normalization with offset 1 (scanpy.pp.log1p) before VAE training
- name: scikit-learn
  role: MaxAbsScaler scales gene values to the 0-1 range within each dataset; sklearn.neighbors.KNeighborsClassifier
    supports downstream label transfer on the learned embeddings
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:2af9edb715cdcd6fc6dfbd34be7ecbac892e351ef38ccd547fa1a4d1f4f4fc08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniport
  tools:
  - Tangram
  - gimVI
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
yes_rate: 0.9230769230769231
n_claims: 13
---

# coupled-vae-optimal-transport-training

## Summary

Trains uniPort's coupled variational autoencoder (coupled-VAE) with minibatch unbalanced optimal transport (Minibatch-UOT) to jointly embed heterogeneous single-cell multi-omics and/or spatial datasets into a shared latent space, leveraging both highly variable common genes and dataset-specific genes.

## When to use

Use when integrating heterogeneous single-cell datasets (e.g. paired 10x PBMC scATAC/scRNA, MERFISH + scRNA) with nonlinear cross-modality deformations, or when common genes alone are insufficient and dataset-specific HVGs must be retained.

## When NOT to use

- Input datasets are unimodal and batch-matched where a linear method such as Harmony suffices
- You need to model regulatory signals of scATAC-seq directly — uniPort's scATAC integration relies on gene activity scores rather than directly modeling regulatory signals (as GLUE does)
- No highly variable common genes exist between datasets and vertical/paired structure is absent, requiring careful mode='d' justification rather than default integration

## Inputs

- Per-dataset preprocessed AnnData matrices with dataset-specific HVGs (e.g. adata1.h5ad, adata2.h5ad)
- Common-gene AnnData matrix (adata_cm.h5ad) containing the shared highly variable genes
- Hyperparameter grids: lambda_ot in {0.5, 1.0, 5.0}, lambda_kl in {0.5, 1.0}, lambda_s in {0.5, 1.0}
- Mode selection flag: 'h', 'v', or 'd'

## Outputs

- AnnData with joint latent embeddings stored in adata.obs['latent']
- Optional global OT plan matrix when save_OT=True (adata, OT tuple)
- Imputed/predicted gene expression in spatial data when out='predict'
- Latent embeddings for downstream metrics (Silhouette, FOSCTTM, ARI/NMI/F1, imputation correlations)

## How to apply

Preprocess each dataset with scanpy (normalize_total, log1p with offset 1); for scATAC-seq apply uniport's TFIDF_LSI (TF-IDF weighting plus latent semantic indexing). Identify k=2000 highly variable common genes across datasets plus kx=ky=2000 dataset-specific HVGs, and scale each gene to 0-1 with MaxAbsScaler. Load the per-dataset AnnData objects and the common-gene AnnData (adata1.h5ad, adata2.h5ad, adata_cm.h5ad) and call uniport.Run(adatas=[adata1, adata2], adata_cm=adata_cm), choosing mode ('h' horizontal with common genes, 'v' vertical for the same cells, 'd' diagonal with no common genes) and sweeping hyperparameters lambda_ot from {0.5, 1.0, 5.0}, lambda_kl (gamma) from {0.5, 1.0}, and lambda_s from {0.5, 1.0} with the default 30000 iterations. The rationale is that the coupled-VAE learns a shared latent space while Minibatch-UOT handles unpaired, heterogeneous cell populations without requiring global alignment. Set out='predict' with a specified decoder to impute gene expression (e.g. held-out MERFISH genes) and ref_id to designate the reference dataset.

## Related tools

- **uniPort** (Provides the coupled-VAE + Minibatch-UOT trainer (uniport.Run), TFIDF_LSI scATAC preprocessing, filter_data/batch_scale utilities, and gene-imputation decoding via out='predict') — https://github.com/caokai1073/uniport
- **scanpy** (Normalizes total counts per cell (scanpy.pp.normalize_total) and performs log-normalization with offset 1 (scanpy.pp.log1p) before VAE training)
- **scikit-learn** (MaxAbsScaler scales gene values to the 0-1 range within each dataset; sklearn.neighbors.KNeighborsClassifier supports downstream label transfer on the learned embeddings)

## Examples

```
import uniport as up; adata = up.Run(adatas=[adata1, adata2], adata_cm=adata_cm, mode='h', lambda_ot=1.0, lambda_kl=0.5, lambda_s=0.5, iteration=30000)
```

## Evaluation signals

- Reproduced paired PBMC metrics fall within a small tolerance of published values: Silhouette 0.64, average FOSCTTM 0.0694, ARI+NMI+F1 total 2.321
- MERFISH imputation correlations on 12 random 80/20 splits (122 training / 31 held-out of 153 genes) approximate mSCC 0.259, aSCC 0.26, mPCC 0.249, aPCC 0.294
- Latent embeddings computed with uniport.metrics.silhouette and label_transfer produce coherent cell-type clusters in UMAP space
- Training converged within the default iteration budget of 30000 with the selected hyperparameters from the searched grids

## Limitations

- Storing the dense full OT plan T (save_OT=True) requires substantial memory; computing capacity should be ascertained before use
- Which hyperparameter combination from the searched grids (lambda, gamma, lambda_s) was selected for each dataset is not stated in the paper
- uniPort's scATAC integration relies on gene activity scores rather than directly modeling regulatory signals, unlike GLUE
- Manifold/OT alignment quality can be constrained by scDART-style global-alignment assumptions if mode is mis-set for the data structure

## Evidence

- [abstract] coupled-VAE + Minibatch-UOT training: "Here we introduce uniPort, a unified single-cell data integration framework that combines a coupled variational autoencoder (coupled-VAE) and minibatch unbalanced optimal transport (Minibatch-UOT)."
- [discussion] preprocessing thresholds: "We identified k = 2000 highly variable common genes across cells of all datasets, and identified kx = ky = 2000 highly variable genes for each dataset"
- [readme] README run example: "# integration with both common and dataset-specific genes
# latent representation are stored in adata.obs['latent']
adata = up.Run(adatas=[adata1, adata2], adata_cm=adata_cm)"
- [readme] mode semantics: "If 'mode=h', integrate data with common genes (Horizontal integration). If 'mode=v', integrate data profiled from the same cells (Vertical integration). If 'mode=d', inetrgate data without common"
- [discussion] hyperparameter grids: "We chose parameters for training coupled-VAE and Minibatch-UOT from λ ∈ {0.5, 1.0, 5.0}, γ ∈ {0.5, 1.0}, and λs ∈ {0.5, 1.0} for all datasets"
