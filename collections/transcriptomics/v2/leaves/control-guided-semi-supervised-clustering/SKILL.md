---
name: control-guided-semi-supervised-clustering
description: Use when analyzing scRNA-seq perturbation data (e.g. treatment vs control,
  gene knock-outs) where the goal is to find perturbation-unique cell states without
  a priori marker knowledge, and integration approaches would erase treatment-specific
  signal.
when_to_use_negative:
- When the goal is batch correction or cross-dataset integration of similar conditions
  — ICAT is designed to preserve perturbation-specific signal that integration workflows
  like Seurat, Scanorama, or Harmony would remove
- When treatment labels are unavailable or the dataset has no defined control population
  to anchor the unsupervised control clustering step
- When a priori cell states or marker genes are already known and a fully supervised
  classification approach is preferred over ICAT's unsupervised control-defined states
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ICAT
  role: Core algorithm implementing control-defined clustering, NCFS gene weighting,
    and semi-supervised Louvain clustering
  repo: https://github.com/BradhamLab/icat
- name: icat-sc
  role: pip-installable package providing icat.models.icat, icat.optimize, and icat.simulate
    modules
  repo: https://github.com/BradhamLab/icat
- name: scanpy
  role: Provides log1p normalization (sc.pp.log1p), PCA, neighbor graph construction,
    Louvain clustering, and UMAP visualization on the ICAT output
- name: AnnData
  role: Data structure holding the expression matrix, treatment labels, sslouvain
    cluster assignments, and the X_icat upweighted matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:7f5d9416ceb51814aedba33cbedf95d288eb6f90dcdc8a32453bcef32bd338e9
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# control-guided-semi-supervised-clustering

## Summary

Runs ICAT's control-guided semi-supervised clustering, which first defines cell states by unsupervised Louvain clustering of control cells, weights genes with NCFS, then semi-supervised clusters all cells so control labels stay fixed while perturbed cells join any cluster. It resolves perturbation-unique cell states that integration workflows (Seurat, Scanorama, Harmony) do not.

## When to use

Use when analyzing scRNA-seq perturbation data (e.g. treatment vs control, gene knock-outs) where the goal is to find perturbation-unique cell states without a priori marker knowledge, and integration approaches would erase treatment-specific signal.

## When NOT to use

- When the goal is batch correction or cross-dataset integration of similar conditions — ICAT is designed to preserve perturbation-specific signal that integration workflows like Seurat, Scanorama, or Harmony would remove
- When treatment labels are unavailable or the dataset has no defined control population to anchor the unsupervised control clustering step
- When a priori cell states or marker genes are already known and a fully supervised classification approach is preferred over ICAT's unsupervised control-defined states

## Inputs

- AnnData object of concatenated control and perturbed single-cell expression data with a treatment column (e.g. adata.obs['treatment'] with 'control'/'perturbed' labels)
- Log1p-normalized expression matrix (sc.pp.log1p applied before clustering)
- Optional PCA representation of control cells for hyperparameter optimization

## Outputs

- AnnData with semi-supervised cluster assignments in obs['sslouvain']
- Upweighted (NCFS-weighted) matrix stored in adata.obsm['X_icat']
- Neighbor graph defined in the ICAT-upweighted space, ready for sc.tl.umap visualization

## How to apply

Log1p-normalize the concatenated control and perturbed AnnData, then optionally PCA the controls. Optimize Louvain hyperparameters on control cells with optimize.optimize_louvain (Calinski-Harabasz over grids of n_neighbors 3–50 and resolution 0.3–1.2), then optimize NCFS parameters with optimize.optimize_ncfs (MCC in weighted KNN over k-fold CV, sigma ~0.5–3, reg ~0.25–3). Instantiate models.icat with ctrl_value='control', ncfs_kws, neighbor_kws, and cluster_kws, then call model.cluster(adata, adata.obs['treatment']); ICAT identifies control-defined states by unsupervised clustering, applies Neighborhood Component Feature Selection to weight predictive genes and down-weight non-explanatory ones, transforms the matrix with the learned weights, and performs semi-supervised Louvain clustering with control labels fixed. Read the final labels from out.obs['sslouvain'] and the upweighted matrix from adata.obsm['X_icat'], which can be passed directly to sc.tl.umap since neighbors are already defined in the ICAT space. If desired, separately optimize sslouvain_n and sslouvain_r with optimize_sslouvain on the full dataset with control_clusters annotated.

## Related tools

- **ICAT** (Core algorithm implementing control-defined clustering, NCFS gene weighting, and semi-supervised Louvain clustering) — https://github.com/BradhamLab/icat
- **icat-sc** (pip-installable package providing icat.models.icat, icat.optimize, and icat.simulate modules) — https://github.com/BradhamLab/icat
- **scanpy** (Provides log1p normalization (sc.pp.log1p), PCA, neighbor graph construction, Louvain clustering, and UMAP visualization on the ICAT output)
- **AnnData** (Data structure holding the expression matrix, treatment labels, sslouvain cluster assignments, and the X_icat upweighted matrix)

## Examples

```
model = models.icat(ctrl_value='control', ncfs_kws={'reg': 1, 'sigma': 3}, neighbor_kws={'n_neighbors': 15}, cluster_kws={'resolution': 0.75}); out = model.cluster(adata, adata.obs['treatment'])
```

## Evaluation signals

- out.obs['sslouvain'] exists and contains non-trivial cluster labels covering both control and perturbed cells
- adata.obsm['X_icat'] is populated with the upweighted matrix and sc.pp.neighbors/sc.tl.umap can be computed directly on it without redefining neighbors
- Control-cell cluster assignments match the previously computed control Louvain labels (control labels remain constant during semi-supervised clustering)
- On benchmark datasets (Kagohara, Kang), ICAT's ARI against ground-truth labels exceeds that of Seurat, Scanorama, and Harmony, with ARI values within [0, 1] and LISI/DB metrics consistent with Figure 4 trends
- Optimized hyperparameters fall within the recommended grid ranges (n_neighbors 3–50, resolution 0.3–1.2, sigma 0.5–3, reg 0.25–3)

## Limitations

- ICAT can only be installed on Linux machines via pip (pip install icat-sc)
- Default behavior reuses the same n_neighbors and resolution for semi-supervised clustering as for control clustering; separate optimization via optimize_sslouvain is recommended when working with your own data
- The ICAT_manuscript analysis was run on a CentOS HPC and is not guaranteed to run out-of-the-box on other systems
- ICAT does not automatically compute UMAP, t-SNE, or other reduced-dimension visualizations during clustering; these must be computed afterwards from adata.obsm['X_icat']

## Evidence

- [intro] perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells regardless of: "perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells regardless of"
- [methods] ICAT can be installed on linux machines using `pip` with the following command: `pip install icat-sc`: "ICAT can be installed on linux machines using `pip` with the following command: `pip install icat-sc`"
- [intro] These identified cell states are then fed into a sparse gene weighting algorithm, Neighborhood Component Feature Selection (NCFS), to highly weight the most predictive genes, while also removing: "These identified cell states are then fed into a sparse gene weighting algorithm, Neighborhood Component Feature Selection (NCFS), to highly weight the most predictive genes, while also removing"
- [readme] While ICAT does not automatically compute UMAP, tsne, or other reduced dimension visualizations during clustering, it is possible to pass the upweighted count matrix: "While ICAT does not automatically compute UMAP, tsne, or other reduced dimension visualizations during clustering, it is possible to pass the upweighted count matrix"
- [readme] We recommend finding appropriate Louvain and NCFS hyper parameters **prior** to running the complete ICAT workflow: "For working with your own data, we recommend finding appropriate Louvain and NCFS hyper
parameters **prior** to running the complete ICAT workflow"
