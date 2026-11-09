---
name: weighted-cross-entropy-classifier-warmup
description: 'Use when reconstructing scBridge''s heterogeneous transfer loop: annotated
  scRNA-seq (.h5ad, obs[''CellType'']) plus unlabeled scATAC-seq activity matrix must
  first be fitted by a supervised encoder/classifier before GMM reliability estimation
  and prototype alignment.'
when_to_use_negative:
- When no annotated scRNA-seq source data with cell-type labels is available — the
  weighted cross-entropy warm-up is fully supervised and cannot start without obs['CellType']
  labels.
- When source and target share no common genes after restriction — the cross-omics
  classifier warm-up and prototype alignment both operate on the common gene space.
- When scATAC-seq peak data has not been converted to a gene activity matrix (e.g.,
  via Signac) — raw peak-by-cell matrices are not valid input for scBridge.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scBridge
  role: Reference implementation of the warm-up and full heterogeneous transfer loop
  repo: https://github.com/XLearning-SCU/scBridge
- name: PyTorch 1.12.1
  role: Framework in which the encoder f, classifier g, and weighted cross-entropy
    warm-up are implemented
- name: scikit-learn 1.1.1
  role: Provides the Gaussian Mixture Model used in later iterations to model per-cell
    reliability
- name: Signac
  role: Used to transform scATAC-seq peak data into the gene activity matrix required
    as target input
  repo: https://github.com/stuart-lab/signac
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-41795-5
    title: scBridge embraces cell heterogeneity in single-cell RNA-seq and ATAC-seq
      data integration
  source_content_hash: sha256:d16ce3c4047e050fa709f005fda6235525ce233588206526498d7ed68e10c49f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/XLearning-SCU/scBridge
  tools:
  - CUDA
  - PyTorch
  - PyTorch 1.12.1
  - Python
  - Signac
  - scBridge
  - scikit-learn
  - scikit-learn 1.1.1
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# weighted-cross-entropy-classifier-warmup

## Summary

Warm-up stage of scBridge: train the PyTorch encoder f and classifier g on annotated scRNA-seq cells with a weighted cross-entropy loss so that downstream reliability modeling of scATAC-seq cells starts from a stable labeled anchor.

## When to use

Use when reconstructing scBridge's heterogeneous transfer loop: annotated scRNA-seq (.h5ad, obs['CellType']) plus unlabeled scATAC-seq activity matrix must first be fitted by a supervised encoder/classifier before GMM reliability estimation and prototype alignment.

## When NOT to use

- When no annotated scRNA-seq source data with cell-type labels is available — the weighted cross-entropy warm-up is fully supervised and cannot start without obs['CellType'] labels.
- When source and target share no common genes after restriction — the cross-omics classifier warm-up and prototype alignment both operate on the common gene space.
- When scATAC-seq peak data has not been converted to a gene activity matrix (e.g., via Signac) — raw peak-by-cell matrices are not valid input for scBridge.

## Inputs

- Annotated source scRNA-seq (CITE-seq) AnnData .h5ad with cell type annotation in obs['CellType']
- Unlabeled target scATAC-seq (ASAP-seq) AnnData .h5ad with gene activity matrix
- Common-gene restriction across source and target matrices

## Outputs

- Trained PyTorch encoder f and classifier g checkpoints used by the integration loop
- Integrated source and target AnnData with Embedding/X_umap and Prediction/Reliability fields
- UMAP plots colored by cell type, domain, and reliability

## How to apply

Load the annotated source CITE-seq AnnData and unlabeled target ASAP-seq AnnData, restrict both matrices to their common genes, and preprocess with Standard normalization for source (divide by total counts, multiply by 10,000) and TFIDF for target. Train the encoder f and classifier g on the annotated scRNA-seq cells using weighted cross-entropy loss with learning_rate=5e-4, batch_size=512, train_epoch=20, and random_seed=2023, early stopping when classification accuracy on the annotated data reaches 99%. This warm-up is required because the subsequent integration iterations (max_iteration=20) reuse f and g to compute scATAC-seq embedding discriminability and classification confidence for the Gaussian Mixture reliability model. After warm-up, iterate: fit the scikit-learn GMM, perform reliability-weighted cross-omics prototype alignment (align_loss_epoch=1, prototype_momentum=0.9), select scATAC-seq cells with reliability > 0.95, merge them into the annotated data, and repeat until no new cells are selected.

## Related tools

- **scBridge** (Reference implementation of the warm-up and full heterogeneous transfer loop) — https://github.com/XLearning-SCU/scBridge
- **PyTorch 1.12.1** (Framework in which the encoder f, classifier g, and weighted cross-entropy warm-up are implemented)
- **scikit-learn 1.1.1** (Provides the Gaussian Mixture Model used in later iterations to model per-cell reliability)
- **Signac** (Used to transform scATAC-seq peak data into the gene activity matrix required as target input) — https://github.com/stuart-lab/signac

## Examples

```
git clone https://github.com/XLearning-SCU/scBridge && cd scBridge && conda env create -f environment.yml && conda activate scBridge && python main.py --reliability_threshold=0.95 --train_epoch=20 --batch_size=512 --learning_rate=5e-4 --random_seed=2023
```

## Evaluation signals

- Training is early-stopped only when annotated-data classification accuracy reaches 99%.
- Reliability-selected scATAC-seq cells must exceed reliability_threshold=0.95 at each iteration; iteration terminates when no new cells are selected.
- Output AnnData must contain Embedding/X_umap and Prediction/Reliability fields.
- Run completes within max_iteration=20 iterations with fixed random_seed=2023 for reproducibility.
- UMAP plots colored by cell type, domain, and reliability show source and target mixing without over-integration of distinct cell types.

## Limitations

- Novel-type discovery mode relies on a heuristic stopping rule (stop when more than 75% of scATAC-seq cells are selected as annotated) and is deferred to Supplementary Note 2, not covered here.
- Over-integration (merging distinct cell types) and under-integration are failure modes when omics difference is confused with cell-type difference.
- All experiments were conducted on a single Nvidia RTX 3090 GPU with CUDA 11.4; CPU-only runs are not described.

## Evidence

- [discussion] Warm up encoder f and classifier g on annotated scRNA-seq data with weighted cross-entropy loss: [evidence span withheld — anchor status: not_found]
- [discussion] Early stopping at 99% annotated accuracy: "we early stopped the network optimization when the cell classification accuracy of the annotated data reached 99%"
- [discussion] GMM reliability modeling follows warm-up: "we model the reliability of scATAC-seq cells with Gaussian Mixture based on their discriminability and confidence."
- [readme] Input requirements from README: "scBridge accepts AnnData (https://anndata.readthedocs.io/en/latest/index.html) as inputs, including the source (scRNA-seq) and target (scATAC-seq) h5ad files. The source data should include the cell"
- [methods] Preprocessing defaults: "`--source_preprocess=["Standard", "TFIDF"]` (Default="Standard") the preprocessing strategy for source data"
