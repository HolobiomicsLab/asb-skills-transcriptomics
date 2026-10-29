---
name: semi-supervised-gene-weighting-clustering
description: Use when analyzing paired control/perturbed scRNA-seq data (e.g. gene
  knock-outs, treatments) where the goal is to identify cell states unique to the
  perturbed condition without a priori knowledge of marker genes or cell states.
when_to_use_negative:
- When there is no paired control condition and no perturbation/biologic heterogeneity
  — ICAT is specifically designed to identify cell states across treatments and needs
  treatment labels
- When the goal is to force all cells from different conditions into shared integrated
  clusters (batch-correction) — ICAT deliberately leaves experimental cells free to
  form perturbation-unique clusters rather than integrating them away
- When a priori cell-state or marker-gene knowledge should drive supervised cell-type
  annotation instead of unsupervised control-guided clustering
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ICAT
  role: Core algorithm providing self-supervised feature weighting (NCFS) and control-guided
    semi-supervised Louvain clustering; used via models.icat.cluster()
  repo: https://github.com/BradhamLab/icat
- name: icat-sc
  role: pip-installable Python package providing the icat.models, icat.optimize, and
    icat.simulate modules used to run and tune the workflow
  repo: https://github.com/BradhamLab/icat
- name: scanpy
  role: Provides AnnData structure, log1p normalization (sc.pp.log1p), PCA, neighbor
    graph, Louvain clustering, and UMAP on the ICAT-weighted matrix
- name: AnnData
  role: Data structure holding the concatenated control/perturbed expression matrix,
    obs treatment labels, and the X_icat weighted matrix in obsm
- name: numpy
  role: Used to supply per-gene dispersion values when constructing the simulated
    SingleCellDataset (e.g. np.random.choice([1, 2, 3], 1000))
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:83d2499b6963740bcf64108709cc3999fb851b6593db5a2ddc00d63518e81fcb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - AnnData
  - Harmony
  - ICAT
  - Scanorama
  - Seurat
  - icat-sc
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# semi-supervised-gene-weighting-clustering

## Summary

Apply ICAT's self-supervised feature weighting (NCFS) and control-guided semi-supervised Louvain clustering to resolve perturbation-unique cell states in scRNA-seq, where standard integration workflows (Seurat, Scanorama, Harmony) fail to preserve treatment-specific responses.

## When to use

Use when analyzing paired control/perturbed scRNA-seq data (e.g. gene knock-outs, treatments) where the goal is to identify cell states unique to the perturbed condition without a priori knowledge of marker genes or cell states.

## When NOT to use

- When there is no paired control condition and no perturbation/biologic heterogeneity — ICAT is specifically designed to identify cell states across treatments and needs treatment labels
- When the goal is to force all cells from different conditions into shared integrated clusters (batch-correction) — ICAT deliberately leaves experimental cells free to form perturbation-unique clusters rather than integrating them away
- When a priori cell-state or marker-gene knowledge should drive supervised cell-type annotation instead of unsupervised control-guided clustering

## Inputs

- Concatenated control + perturbed AnnData object (scanpy) with a 'treatment' obs column labeling 'control' vs 'perturbed' cells
- Control-only AnnData with PCA computed (sc.pp.pca) for hyperparameter grid search
- Optionally, archived simulated datasets with ground-truth population labels (from BradhamLab/ICAT_manuscript archived_data.tar.gz)

## Outputs

- AnnData with 'sslouvain' column containing semi-supervised cluster assignments for all cells
- Control-defined Louvain cluster labels on control cells
- NCFS weight vector and upweighted count matrix stored in adata.obsm['X_icat']
- Optional UMAP embedding computed in the ICAT-weighted space

## How to apply

First identify control-defined cell states by unsupervised clustering: log1p-normalize the concatenated AnnData, then grid-search Louvain parameters (n_neighbors 3-50, resolution 0.3-1.2) on control cells by maximizing the Calinski-Harabasz Index via optimize.optimize_louvain. Next, weight predictive genes with Neighborhood Component Feature Selection (NCFS), choosing sigma (grid 0.5-3) and reg (grid 0.25-3) by optimizing MCC of a weighted 5-NN under k-fold cross-validation via optimize_ncfs; transform the data matrix with the learned weight vector. Finally, perform semi-supervised clustering (models.icat with ctrl_value='control', e.g. ncfs_kws={'reg': 1, 'sigma': 3}, cluster_kws={'resolution': 0.75}) so that originally identified control labels remain constant while cells from experimental conditions are free to cluster with any other cells regardless of treatment status; read the resulting assignments from the 'sslouvain' column of the returned AnnData. Optionally optimize sslouvain parameters separately with optimize_sslouvain in NCFS space, and visualize by passing the upweighted matrix in adata.obsm['X_icat'] to sc.tl.umap.

## Related tools

- **ICAT** (Core algorithm providing self-supervised feature weighting (NCFS) and control-guided semi-supervised Louvain clustering; used via models.icat.cluster()) — https://github.com/BradhamLab/icat
- **icat-sc** (pip-installable Python package providing the icat.models, icat.optimize, and icat.simulate modules used to run and tune the workflow) — https://github.com/BradhamLab/icat
- **scanpy** (Provides AnnData structure, log1p normalization (sc.pp.log1p), PCA, neighbor graph, Louvain clustering, and UMAP on the ICAT-weighted matrix)
- **AnnData** (Data structure holding the concatenated control/perturbed expression matrix, obs treatment labels, and the X_icat weighted matrix in obsm)
- **numpy** (Used to supply per-gene dispersion values when constructing the simulated SingleCellDataset (e.g. np.random.choice([1, 2, 3], 1000)))

## Examples

```
model = models.icat(ctrl_value='control', ncfs_kws={'reg': 1, 'sigma': 3}, cluster_kws={'resolution': 0.75}); out = model.cluster(adata, adata.obs['treatment'])
```

## Evaluation signals

- The returned AnnData contains an 'sslouvain' column whose unique values can be printed via out.obs['sslouvain'].unique(); control cells must retain their original control-cluster labels
- optimize_louvain returns an (n_neighbors, resolution) pair that maximizes the Calinski-Harabasz Index over the searched grid (n 3-50, res 0.3-1.2)
- optimize_ncfs returns (sigma, reg) that maximize MCC of a weighted 5-NN over k-fold cross-validation on control cells
- In benchmarks against Seurat, Scanorama, Harmony, and no-integration baselines, ICAT should outperform all baselines on clustering metrics and remain stable across perturbation severity (1-25%), marker-gene count (10-105), and cell-proportion conditions
- Only ICAT should resolve the perturbation-unique cellular response (e.g. a population absent in control data) when clusters are visualized by UMAP on adata.obsm['X_icat']

## Limitations

- ICAT is implemented for Linux and installed via pip (`pip install icat-sc`); the manuscript analysis ran on a CentOS high-performance cluster and the Snakemake pipeline 'is not guaranteed to run on your system out-of-the-box'
- Hyperparameters (Louvain n_neighbors/resolution, NCFS sigma/reg) must be tuned per-dataset; README recommends running grid searches on your own data prior to the full workflow, and the manuscript supplemental tables list parameters used
- The discussion section content was not extractable, so additional caveats from the authors' discussion are unavailable
- By default the same n_neighbors and resolution are reused for semi-supervised clustering as for control clustering; separate optimization (optimize_sslouvain, max_cells=750) is optional and only 'in practice... leads to good results'

## Evidence

- [abstract] employs self-supervised feature weighting and control-guided clustering to accurately resolve cell states across heterogeneous conditions: "employs self-supervised feature weighting and control-guided clustering to accurately resolve cell states across heterogeneous conditions"
- [intro] perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells regardless of: "perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells regardless of"
- [readme] Find the "best" `n_neighbor` and `resolution` parameter for clustering control cells by optimizing the Calinski-Harabasz Index: "Find the "best" `n_neighbor` and `resolution` parameter for clustering control cells by optimizing the Calinski-Harabasz Index over a grid of `n` and `r` values"
- [readme] find "best" `sigma` and `reg` NCFS values by measuring the MCC in a weighted KNN over k-fold cross validation: "find "best" `sigma` and `reg` NCFS values by measuring the MCC in a
# weighted KNN over k-fold cross validation"
- [methods] out = model.cluster(adata, adata.obs['treatment'])
    print(out.obs['sslouvain'].unique()): "out = model.cluster(adata, adata.obs['treatment'])
    print(out.obs['sslouvain'].unique())"
