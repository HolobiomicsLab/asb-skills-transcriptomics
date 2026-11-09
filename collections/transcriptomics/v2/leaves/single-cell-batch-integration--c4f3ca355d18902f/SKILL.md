---
name: single-cell-batch-integration
description: Use when a paired control/perturbed scRNA-seq dataset (e.g. GSE164240
  PMC cells treated with MK-886, or simulated control + simulate.perturb cells) must
  be clustered so treatment-affected states are resolved across conditions.
when_to_use_negative:
- When the dataset is a standard batch-correction problem with no perturbation/control
  design — plain Harmony or Scanorama alone is the intended tool there, and ICAT's
  control-guided clustering is unnecessary
- When control-defined cell states or marker genes are already known a priori and
  the goal is classification rather than discovering perturbation-unique states
- When the input is a feature/signature table rather than a cell-by-gene count matrix
  consumable by scanpy/AnnData
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ICAT
  role: 'Primary method: control-defined unsupervised clustering, NCFS gene weighting,
    and semi-supervised Louvain clustering of control + perturbed cells; results stored
    in the sslouvain column'
  repo: https://github.com/BradhamLab/icat
- name: scanpy
  role: AnnData data structure, log1p normalization (sc.pp.log1p), PCA, neighbors,
    Louvain clustering on each integration embedding, and UMAP on the upweighted X_icat
    matrix
- name: Seurat
  role: Baseline integration workflow run on the same control/perturbed data to compare
    cluster-level treatment enrichment against ICAT
- name: Scanorama
  role: Baseline integration workflow producing an alternative integrated embedding
    for the Fig 5C comparison
- name: Harmony
  role: Baseline integration workflow producing a third integrated embedding for the
    comparison of perturbation-response detection
- name: icat-sc
  role: pip-installable package providing icat.models.icat, icat.simulate, and icat.optimize
    (optimize_louvain, optimize_ncfs, optimize_sslouvain) used in the workflow
  repo: https://github.com/BradhamLab/icat
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:a8f78bdf14321469953855999c9a85774b798b5cddcaa8ccb330a9a5ec4a0e0f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - Harmony
  - ICAT
  - Scanorama
  - Seurat
  - icat-sc
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# single-cell-batch-integration

## Summary

Integrate control and perturbed single-cell transcriptomes with ICAT's control-guided semi-supervised clustering (with Seurat, Scanorama, and Harmony as comparison baselines) to resolve perturbation-unique cell states that standard integration workflows miss.

## When to use

Use when a paired control/perturbed scRNA-seq dataset (e.g. GSE164240 PMC cells treated with MK-886, or simulated control + simulate.perturb cells) must be clustered so treatment-affected states are resolved across conditions.

## When NOT to use

- When the dataset is a standard batch-correction problem with no perturbation/control design — plain Harmony or Scanorama alone is the intended tool there, and ICAT's control-guided clustering is unnecessary
- When control-defined cell states or marker genes are already known a priori and the goal is classification rather than discovering perturbation-unique states
- When the input is a feature/signature table rather than a cell-by-gene count matrix consumable by scanpy/AnnData

## Inputs

- Control and perturbed single-cell count matrices (e.g. processed GSE164240 PMC dataset) as AnnData objects
- Treatment labels column (adata.obs['treatment'], with ctrl_value='control')
- Optional simulated data from icat.simulate.SingleCellDataset + simulate.perturb for validation

## Outputs

- AnnData with sslouvain semi-supervised cluster assignments in obs and NCFS-upweighted features in obsm['X_icat']
- Per-cluster control vs. perturbed cell-count tables with G-test p-values and post hoc Fisher's exact depletion/enrichment calls per integration method (ICAT vs. Seurat/Scanorama/Harmony)
- UMAP visualization of the upweighted space colored by sslouvain and condition

## How to apply

Load control and perturbed count matrices into a single AnnData object, concatenate, and apply log1p normalization (scanpy). First cluster control cells alone: run PCA, then optimize Louvain n_neighbors/resolution by maximizing the Calinski-Harabasz Index over a grid (e.g. neighbors 3–50, resolution 0.3–1.2), and cluster with sc.pp.neighbors/sc.tl.louvain. Feed control labels into NCFS gene weighting, optimizing sigma and reg by measuring MCC in a weighted KNN over k-fold cross-validation (sigma_vals ~0.5–3, reg_vals ~0.25–3); the upweighted matrix is stored in adata.obsm['X_icat']. Run semi-supervised Louvain (sslouvain) with control labels fixed and perturbed cells free to join any cluster, optionally via optimize.optimize_sslouvain. As a benchmark, run Seurat, Scanorama, and Harmony integrations on the same data, cluster each with sc.tl.louvain, tabulate control vs. perturbed counts per cluster, and test per-cluster treatment enrichment with a G-test followed by post hoc Fisher's exact tests to determine depletion vs. enrichment direction.

## Related tools

- **ICAT** (Primary method: control-defined unsupervised clustering, NCFS gene weighting, and semi-supervised Louvain clustering of control + perturbed cells; results stored in the sslouvain column) — https://github.com/BradhamLab/icat
- **scanpy** (AnnData data structure, log1p normalization (sc.pp.log1p), PCA, neighbors, Louvain clustering on each integration embedding, and UMAP on the upweighted X_icat matrix)
- **Seurat** (Baseline integration workflow run on the same control/perturbed data to compare cluster-level treatment enrichment against ICAT)
- **Scanorama** (Baseline integration workflow producing an alternative integrated embedding for the Fig 5C comparison)
- **Harmony** (Baseline integration workflow producing a third integrated embedding for the comparison of perturbation-response detection)
- **icat-sc** (pip-installable package providing icat.models.icat, icat.simulate, and icat.optimize (optimize_louvain, optimize_ncfs, optimize_sslouvain) used in the workflow) — https://github.com/BradhamLab/icat

## Examples

```
from icat import models; model = models.icat(ctrl_value="control", ncfs_kws={'reg': 1, 'sigma': 3}); out = model.cluster(adata, adata.obs['treatment'])
```

## Evaluation signals

- On the PMC/MK-886 dataset, the ICAT sslouvain assignments must resolve both a depleted and an enriched perturbation-response cluster, whereas Seurat, Scanorama, and Harmony Louvain clusterings each fail to resolve both (Fig 5C pattern)
- Control cells retain their initial Louvain labels during semi-supervised clustering — only experimental-condition cells are free to move between clusters
- NCFS hyperparameters sigma and reg must maximize MCC in weighted KNN k-fold cross-validation (grid ~sigma 0.5–3, reg 0.25–3); Louvain parameters must maximize the Calinski-Harabasz Index
- Observed clusterings should be reproducible on simulated data: 2-population, 1000-gene SingleCellDataset with per-gene dispersion drawn from [1, 2, 3], perturbed via simulate.perturb
- Each perturbed-vs-control cluster count table must show statistically significant G-test results with post hoc Fisher's exact tests indicating enrichment direction

## Limitations

- ICAT makes no automatic UMAP/tSNE during clustering — the upweighted matrix in adata.obsm['X_icat'] must be passed to dimensionality-reduction methods manually
- The ICAT_manuscript pipeline was run on a CentOS HPC and 'is not guaranteed to run on your system out-of-the-box'; reproduce via conda environment + snakemake
- Only Linux installation via pip is documented; default ICAT reuses control-clustering n_neighbors/resolution for semi-supervised clustering, which may require separate optimization (optimize_sslouvain with max_cells=750) for best results

## Evidence

- [abstract] only ICAT identifies a perturbation-unique cellular response: "We empirically validate ICAT in a developmental model and find that only ICAT identifies a perturbation-unique cellular response."
- [intro] ICAT scope statement: "ICAT is a tool developed to better identify cell states in scRNAseq experiments where perturbations or some other biologic heterogeneity is present"
- [intro] semi-supervised clustering design: "perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells"
- [readme] Calinski-Harabasz Louvain optimization: "# optimizing the Calinski-Harabasz Index over a grid of `n` and `r` values"
- [readme] NCFS MCC cross-validation optimization: "# find "best" `sigma` and `reg` NCFS values by measuring the MCC in a weighted KNN over k-fold cross validation"
