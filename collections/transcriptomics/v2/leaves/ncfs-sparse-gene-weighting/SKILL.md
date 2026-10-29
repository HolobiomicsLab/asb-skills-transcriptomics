---
name: ncfs-sparse-gene-weighting
description: Use when clustering single-cell perturbation data where control-defined
  cell states are available and differential signal must be isolated from noisy genes
  prior to semi-supervised clustering.
when_to_use_negative:
- When no control-defined cell state labels exist — NCFS requires fixed labels from
  unsupervised clustering of the control arm
- When the goal is to co-cluster treated and control cells without preserving control
  labels, e.g. standard integration workflows rather than control-guided clustering
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ICAT
  role: Framework whose NCFS sparse gene weighting step transforms the expression
    matrix into the upweighted space (adata.obsm['X_icat']) for control-guided clustering
  repo: https://github.com/BradhamLab/icat
- name: icat-sc
  role: pip-installable package providing models.icat (NCFS weighting) and optimize.optimize_ncfs
    for grid-searching sigma and reg
  repo: https://github.com/BradhamLab/icat
- name: scanpy
  role: Provides log1p normalization, PCA, neighbors, and Louvain clustering of the
    control arm whose labels drive NCFS feature weighting
- name: AnnData
  role: Data structure holding the expression matrix, treatment labels, control_clusters,
    and the X_icat upweighted matrix
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:b7e1ba5dede2ef2b1f6fbfe5e953742528841ce271c890b4443273361a42853d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - AnnData
  - ICAT
  - icat-sc
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# ncfs-sparse-gene-weighting

## Summary

Applies Neighborhood Component Feature Selection (NCFS) within ICAT to learn a sparse per-gene weight vector that highly weights genes predictive of control-defined cell states while down-weighting non-explanatory genes, then transforms the expression matrix into the upweighted space used for semi-supervised clustering.

## When to use

Use when clustering single-cell perturbation data where control-defined cell states are available and differential signal must be isolated from noisy genes prior to semi-supervised clustering.

## When NOT to use

- When no control-defined cell state labels exist — NCFS requires fixed labels from unsupervised clustering of the control arm
- When the goal is to co-cluster treated and control cells without preserving control labels, e.g. standard integration workflows rather than control-guided clustering

## Inputs

- AnnData object with control-arm cells log1p-normalized (scanpy sc.pp.log1p)
- Control cell state labels (e.g. controls.obs.louvain or control_clusters) from unsupervised Louvain clustering
- NCFS hyperparameters sigma (kernel width) and reg (regularization), optionally from optimize_ncfs grid search

## Outputs

- Learned per-gene weight vector (sparse, down-weighting non-explanatory genes)
- Upweighted data matrix stored in adata.obsm['X_icat']
- Neighbors computed in the upweighted space, consumed by the sslouvain step

## How to apply

First define control cell states by unsupervised Louvain clustering of the control arm (e.g. sc.pp.neighbors + sc.tl.louvain on the control AnnData after sc.pp.log1p normalization) and record these labels, e.g. in a 'control_clusters' column. Feed these fixed control labels into NCFS gradient-ascent feature weighting via icat's models.icat with ncfs_kws such as {'reg': 1, 'sigma': 3}; the regularization (reg) and kernel width (sigma) can be grid-searched with icat.optimize.optimize_ncfs, which measures the MCC in a weighted KNN over k-fold cross-validation (e.g. sigma in [0.5–3], reg in [0.25–3], n_neighbors=5, n_splits=3). NCFS highly weights the most predictive genes and removes variance from non-explanatory genes, and the learned weight vector transforms the data matrix into the upweighted space stored in adata.obsm['X_icat']. This upweighted matrix then serves as the input space for the semi-supervised Louvain (sslouvain) step in which control labels remain constant while perturbed cells cluster freely. Hyperparameter optimization for Louvain and NCFS is recommended on the control data alone before running the full ICAT workflow.

## Related tools

- **ICAT** (Framework whose NCFS sparse gene weighting step transforms the expression matrix into the upweighted space (adata.obsm['X_icat']) for control-guided clustering) — https://github.com/BradhamLab/icat
- **icat-sc** (pip-installable package providing models.icat (NCFS weighting) and optimize.optimize_ncfs for grid-searching sigma and reg) — https://github.com/BradhamLab/icat
- **scanpy** (Provides log1p normalization, PCA, neighbors, and Louvain clustering of the control arm whose labels drive NCFS feature weighting)
- **AnnData** (Data structure holding the expression matrix, treatment labels, control_clusters, and the X_icat upweighted matrix)

## Examples

```
model = models.icat(ctrl_value='control', ncfs_kws={'reg': 1, 'sigma': 3}); out = model.cluster(adata, adata.obs['treatment'])
```

## Evaluation signals

- adata.obsm['X_icat'] exists and differs from the raw log1p matrix, reflecting the learned NCFS weight vector
- optimize_ncfs returns sigma and reg values whose weighted-KNN MCC over k-fold cross-validation is maximized on the control labels
- Downstream sslouvain clustering in the upweighted space recovers simulated true populations with near-perfect agreement (ARI approximately 1) on the equal-population simulated dataset
- Non-explanatory genes receive near-zero weights, concentrating weight mass on genes predictive of the control-defined cell states
- Control cell labels remain immutable after the subsequent sslouvain step, confirming the upweighted space was used with fixed control labels

## Limitations

- NCFS hyperparameters (sigma, reg) are dataset-dependent; the README recommends grid-searching them on control data prior to the full ICAT workflow rather than using defaults
- The weighting is driven entirely by control-defined Louvain labels, so poor control clustering propagates into the gene weights
- The article's discussion section content was not extractable, so discussed caveats beyond abstract-level claims (robustness to low signal strength, high perturbation severity, disparate cell type proportions) are not verifiable here

## Evidence

- [intro] These identified cell states are then fed into a sparse gene weighting algorithm, Neighborhood Component Feature Selection (NCFS), to highly weight the most predictive genes, while also removing: "These identified cell states are then fed into a sparse gene weighting algorithm, Neighborhood Component Feature Selection (NCFS), to highly weight the most predictive genes, while also removing"
- [intro] We then transform the data matrix using this weight vector: "We then transform the data matrix using this weight vector"
- [readme] find "best" `sigma` and `reg` NCFS values by measuring the MCC in a weighted KNN over k-fold cross validation: "find "best" `sigma` and `reg` NCFS values by measuring the MCC in a
weighted KNN over k-fold cross validation"
- [readme] model = models.icat(ctrl_value="control", ncfs_kws={'reg': 1, 'sigma': 3}, ...): "model = models.icat(
        ctrl_value="control",
        ncfs_kws={'reg': 1, 'sigma': 3},"
- [readme] the upweighted count matrix (found in adata.obsm["X_icat"]): "pass the upweighted count matrix
(found in `adata.obsm["X_icat"]`) to these algorithms"
