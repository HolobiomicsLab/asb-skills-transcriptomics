---
name: neighborhood-component-feature-selection-gene-weighting
description: Use when clustering single-cell perturbation data where control-defined
  cell states are already labeled, and non-explanatory gene variance must be down-weighted
  before assigning perturbed cells to states.
when_to_use_negative:
- No control-defined cell-state labels exist yet — NCFS is supervised by control Louvain
  clusters, so run optimize_louvain/clustering first
- Data are not in a scanpy AnnData structure with log1p normalization applied — ICAT
  makes heavy use of the scanpy library and AnnData data structure
- The goal is integration across treatments rather than identifying perturbation-unique
  cell states — the paper reports integration workflows behave differently from ICAT
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: icat-sc
  role: Python package providing models.icat (NCFS weighting via ncfs_kws) and optimize.optimize_ncfs
    grid search; installed with pip install icat-sc
  repo: https://github.com/BradhamLab/icat
- name: ICAT
  role: Algorithm embedding NCFS sparse gene weighting between control clustering
    and semi-supervised Louvain clustering
  repo: https://github.com/BradhamLab/icat
- name: scanpy
  role: Provides log1p normalization, neighbors, Louvain, and UMAP on the NCFS-transformed
    matrix
- name: AnnData
  role: Data structure carrying obs treatment labels, louvain/control_clusters labels,
    and obsm['X_icat'] output
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:1aecadaf82ff1b4df0d12aea1fe2a445924735c383b5c9606a862869dbbd6e5b
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
yes_rate: 0.7272727272727273
n_claims: 11
---

# neighborhood-component-feature-selection-gene-weighting

## Summary

Sparse gene-weighting step of ICAT that uses Neighborhood Component Feature Selection (NCFS) to highly weight genes predictive of control-defined cell states while removing variance from non-explanatory genes, producing an upweighted matrix for semi-supervised clustering.

## When to use

Use when clustering single-cell perturbation data where control-defined cell states are already labeled, and non-explanatory gene variance must be down-weighted before assigning perturbed cells to states.

## When NOT to use

- No control-defined cell-state labels exist yet — NCFS is supervised by control Louvain clusters, so run optimize_louvain/clustering first
- Data are not in a scanpy AnnData structure with log1p normalization applied — ICAT makes heavy use of the scanpy library and AnnData data structure
- The goal is integration across treatments rather than identifying perturbation-unique cell states — the paper reports integration workflows behave differently from ICAT

## Inputs

- log1p-normalized AnnData count matrix (scanpy/sc.pp.log1p) with control and perturbed cells
- Control-cell cluster labels (e.g. adata.obs.louvain or control_clusters column) from unsupervised Louvain clustering
- Treatment/condition labels per cell (e.g. adata.obs['treatment'] with 'control' vs perturbed values)

## Outputs

- Learned NCFS weight vector (sparse gene weights)
- Upweighted/transformed expression matrix in adata.obsm['X_icat']
- Neighbors defined in the upweighted space, enabling sc.tl.umap visualization

## How to apply

First define control cell states by unsupervised Louvain clustering (hyperparameters n_neighbors and resolution optimized with optimize_louvain over the Calinski-Harabasz Index), since these labels supervise NCFS. Then grid-search NCFS's kernel width (sigma over [0.5, 1, 1.5, 2, 2.5, 3]) and regularization (reg over [0.25, 0.5, 1, 1.5, 2, 2.5, 3]) with icat.optimize.optimize_ncfs, scored by MCC from a 5-NN weighted KNN over 3-fold cross-validation on control labels. Fit models.icat with the chosen ncfs_kws (e.g. reg=1, sigma=3) and neighbor_kws (n_neighbors=15); NCFS learns a weight vector that transforms the log1p-normalized count matrix, with the transformed matrix stored in adata.obsm['X_icat']. The rationale is that self-supervised feature weighting concentrates signal in treatment-relevant genes so that downstream semi-supervised Louvain clustering (control labels fixed, perturbed cells free) resolves states robustly across heterogeneous conditions.

## Related tools

- **icat-sc** (Python package providing models.icat (NCFS weighting via ncfs_kws) and optimize.optimize_ncfs grid search; installed with pip install icat-sc) — https://github.com/BradhamLab/icat
- **ICAT** (Algorithm embedding NCFS sparse gene weighting between control clustering and semi-supervised Louvain clustering) — https://github.com/BradhamLab/icat
- **scanpy** (Provides log1p normalization, neighbors, Louvain, and UMAP on the NCFS-transformed matrix)
- **AnnData** (Data structure carrying obs treatment labels, louvain/control_clusters labels, and obsm['X_icat'] output)

## Examples

```
from icat import optimize; sigma, reg = optimize.optimize_ncfs(controls, controls.obs.louvain, n_neighbors=5, n_splits=3, sigma_vals=[0.5,1,1.5,2,2.5,3], reg_vals=[0.25,0.5,1,1.5,2,2.5,3])
```

## Evaluation signals

- optimize_ncfs returns the sigma/reg pair maximizing weighted-KNN MCC over 3-fold cross-validation on control labels (5-NN)
- The learned weights highly weight predictive genes while removing variance from non-explanatory ones; transformed data live in adata.obsm['X_icat']
- Downstream: semi-supervised clustering preserves original control labels while perturbed cells reassign, and final labels appear in the 'sslouvain' column
- Applied to the GSE164240 sea urchin PMC dataset, the workflow resolves exactly five clusters with cluster 3 (sm50+/pks2−) and cluster 4 (sm50−/pks2+) states preserved and Fig 5A enrichment reproduced

## Limitations

- ICAT makes heavy use of the scanpy library and AnnData data structure; inputs outside that ecosystem need conversion
- Hyperparameters (sigma, reg, n_neighbors, resolution) should be optimized prior to the full workflow; defaults from the paper (reg=1, sigma=3) may not transfer
- The ICAT_manuscript analysis ran on a CentOS HPC and 'is not guaranteed to run on your system out-of-the-box'; optimization of sslouvain is capped at max_cells=750
- ICAT does not automatically compute UMAP/tSNE; use the upweighted matrix in obsm['X_icat'] for visualization

## Evidence

- [intro] NCFS sparse weighting rationale: "These identified cell states are then fed into a sparse gene weighting algorithm, Neighborhood Component Feature Selection (NCFS), to highly weight the most predictive genes"
- [readme] NCFS hyperparameter grid search: "sigma, reg =  optimize.optimize_ncfs(
    controls,
    controls.obs.louvain,
    n_neighbors=5,
    n_splits=3,
    sigma_vals=[0.5, 1, 1.5, 2, 2.5, 3]"
- [readme] Model fit with ncfs_kws and transformed output: "model = models.icat(
        ctrl_value="control",
        ncfs_kws={'reg': 1, 'sigma': 3},
        neighbor_kws={'n_neighbors': 15}"
- [readme] X_icat upweighted matrix for visualization: "it is possible to pass the upweighted count matrix
(found in `adata.obsm["X_icat"]`) to these algorithms"
- [readme] Optimize before full workflow: "For working with your own data, we recommend finding appropriate Louvain and NCFS hyper
parameters **prior** to running the complete ICAT workflow."
- [abstract] Robustness claim: "ICAT is robust to low signal strength, high perturbation severity, and disparate cell type proportions"
