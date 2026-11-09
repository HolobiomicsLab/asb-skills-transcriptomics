---
name: single-cell-clustering-across-treatments
description: Use when a single-cell perturbation dataset contains control cells plus
  one or more treatments and you need control-anchored, interpretable cell states
  — especially when standard integration workflows might erase perturbation-unique
  responses.
when_to_use_negative:
- No control/untreated population exists to anchor the semi-supervised clustering
  — ICAT requires control-defined states as input
- The goal is batch correction or integration that removes perturbation effects, rather
  than resolving treatment-induced cell states (ICAT deliberately preserves perturbation-unique
  responses)
- Input is already aggregated (pseudobulk) or is a pre-labeled cell-type annotation
  table rather than per-cell counts
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: icat-sc
  role: Installable ICAT package providing icat.models.icat, icat.optimize.optimize_louvain,
    icat.optimize.optimize_ncfs, and icat.optimize.optimize_sslouvain used to run
    the full clustering workflow
  repo: https://github.com/BradhamLab/icat
- name: scanpy
  role: Handles log1p normalization (sc.pp.log1p), PCA, neighbor graph construction
    (sc.pp.neighbors), Louvain clustering (sc.tl.louvain), and UMAP visualization
    (sc.tl.umap) inside the ICAT workflow
- name: AnnData
  role: Data structure carrying the count matrix, obs treatment labels, and outputs
    (sslouvain column, obsm['X_icat']) through every ICAT step
- name: numpy
  role: Used to set simulated dispersion values (e.g. np.random.choice([1, 2, 3],
    1000)) and array handling in the ICAT example workflow
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:be62ef7d1ff6af800340f49b7710409df61f33e4a4ff56f90471d7fc5df52476
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - AnnData
  - icat-sc
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-clustering-across-treatments

## Summary

Apply ICAT, which combines NCFS self-supervised gene weighting with control-guided semi-supervised Louvain clustering, to resolve cell states in single-cell perturbation data while keeping control labels fixed so treated cells join any state. The paper validates this on a sea urchin PMC SMART-Seq2 dataset where only ICAT recovers a perturbation-unique response that integration workflows miss.

## When to use

Use when a single-cell perturbation dataset contains control cells plus one or more treatments and you need control-anchored, interpretable cell states — especially when standard integration workflows might erase perturbation-unique responses.

## When NOT to use

- No control/untreated population exists to anchor the semi-supervised clustering — ICAT requires control-defined states as input
- The goal is batch correction or integration that removes perturbation effects, rather than resolving treatment-induced cell states (ICAT deliberately preserves perturbation-unique responses)
- Input is already aggregated (pseudobulk) or is a pre-labeled cell-type annotation table rather than per-cell counts

## Inputs

- Cell-by-gene single-cell count matrix (e.g. SMART-Seq2 counts from GSE164240, 435 PMC cells) as an AnnData object
- Per-cell treatment annotation column (control, MK-886, chlorate) in adata.obs, with 'control' as ctrl_value
- Control-cell subset for hyperparameter optimization and control-defined clustering

## Outputs

- Final cell-state labels in the 'sslouvain' column of the returned AnnData (e.g. five clusters for the PMC dataset)
- Control-defined Louvain clusters and control_clusters column
- NCFS gene weight vector and upweighted data matrix stored in adata.obsm['X_icat']
- Cluster-vs-treatment enrichment table (G-tests / Fisher exact tests, Fig 5A-style)
- Optional UMAP embedding computed on adata.obsm['X_icat']

## How to apply

Load the combined cell-by-gene SMART-Seq2 count matrix (e.g. GSE164240, 435 PMC cells) into AnnData with treatment labels (control, MK-886, chlorate) and normalize with sc.pp.log1p. First cluster controls only: grid-search Louvain n_neighbors/resolution with icat.optimize.optimize_louvain (min_neighbors=3, max_neighbors=50, neighbor_step=2, min_res=0.3, max_res=1.2, res_step=0.02, maximizing the Calinski-Harabasz Index), then run sc.pp.neighbors and sc.tl.louvain at the best values. Next weight genes with icat.optimize.optimize_ncfs (sigma grid [0.5, 1, 1.5, 2, 2.5, 3], reg grid [0.25, 0.5, 1, 1.5, 2, 2.5, 3], scored by MCC in 5-NN over 3-fold cross-validation), then fit icat.models.icat with the optimized ncfs_kws (reg, sigma), neighbor_kws (n_neighbors=15), and cluster_kws (resolution=0.75) and run model.cluster on the full AnnData with the treatment column, so control labels remain constant and perturbed cells are free to join any cluster; collect labels from the 'sslouvain' column. Finally map clusters to phenotypes (verifying e.g. sm50+/pks2- vs sm50-/pks2+ marker inversions) and test cluster-vs-treatment enrichment with G-tests and Fisher exact tests.

## Related tools

- **icat-sc** (Installable ICAT package providing icat.models.icat, icat.optimize.optimize_louvain, icat.optimize.optimize_ncfs, and icat.optimize.optimize_sslouvain used to run the full clustering workflow) — https://github.com/BradhamLab/icat
- **scanpy** (Handles log1p normalization (sc.pp.log1p), PCA, neighbor graph construction (sc.pp.neighbors), Louvain clustering (sc.tl.louvain), and UMAP visualization (sc.tl.umap) inside the ICAT workflow)
- **AnnData** (Data structure carrying the count matrix, obs treatment labels, and outputs (sslouvain column, obsm['X_icat']) through every ICAT step)
- **numpy** (Used to set simulated dispersion values (e.g. np.random.choice([1, 2, 3], 1000)) and array handling in the ICAT example workflow)

## Examples

```
from icat import models; model = models.icat(ctrl_value='control', ncfs_kws={'reg': 1, 'sigma': 3}, neighbor_kws={'n_neighbors': 15}, cluster_kws={'resolution': 0.75}); out = model.cluster(adata, adata.obs['treatment'])
```

## Evaluation signals

- The full 435-cell partition resolves exactly five clusters while control-defined states are preserved (control labels fixed during semi-supervised clustering)
- Marker inversion is recovered: cluster 3 is sm50+/pks2- and cluster 4 is sm50-/pks2+ in the PMC data
- Cluster-vs-treatment G-test / Fisher exact enrichment statistics reproduce the Fig 5A associations (cluster 3 enriched in one perturbation, cluster 4 in the other)
- Hyperparameter grids were actually optimized: Louvain grid maximizes the Calinski-Harabasz Index; NCFS grid maximizes MCC in 5-NN, 3-fold cross-validation
- The returned AnnData contains the upweighted matrix in obsm['X_icat'] and unique labels accessible via out.obs['sslouvain']

## Limitations

- ICAT does not automatically compute UMAP/tSNE; you must pass the upweighted matrix (adata.obsm['X_icat']) to sc.tl.umap yourself
- Louatin/NCFS hyperparameters must be tuned on your data before the full workflow; by default semi-supervised clustering reuses the control-clustering n_neighbors and resolution, and separate optimization via optimize_sslouvain is capped at max_cells=750
- Installation is documented for Linux only (pip install icat-sc), and the manuscript analysis was run on a CentOS HPC cluster and 'is not guaranteed to run on your system out-of-the-box'

## Evidence

- [readme] The method works by first identifying a set of conrol-defined cell states by performing unsupervised clustering.: "The method works by first identifying a set of conrol-defined cell states by performing unsupervised clustering."
- [intro] These identified cell states are then fed into a sparse gene weighting algorithm, Neighborhood Component Feature Selection (NCFS), to highly weight the most predictive genes, while also removing: "These identified cell states are then fed into a sparse gene weighting algorithm, Neighborhood Component Feature Selection (NCFS), to highly weight the most predictive genes, while also removing"
- [methods] perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells regardless of: "perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells regardless of"
- [readme] The final clustering is stored in the `sslouvain` column of the returned `AnnData` object.: "The final clustering is stored in the `sslouvain` column of the returned `AnnData` object."
- [readme] While ICAT does not automatically compute UMAP, tsne, or other reduced dimension visualizations during clustering, it is possible to pass the upweighted count matrix: "While ICAT does not automatically compute UMAP, tsne, or other reduced dimension visualizations during clustering, it is possible to pass the upweighted count matrix"
- [readme] find the "best" `sigma` and `reg` NCFS values by measuring the MCC in a weighted KNN over k-fold cross validation: "find the "best" `sigma` and `reg` NCFS values by measuring the MCC in a weighted KNN over k-fold cross validation"
