---
name: perturbation-clustering-benchmarking
description: Use when comparing cell-state clustering methods on single-cell perturbation
  data (e.g. drug-treated vs control, IFN-beta stimulated PBMCs) where cluster quality
  must be quantified against ground-truth labels and mixing metrics.
when_to_use_negative:
- The analysis goal is batch integration for visualization rather than quantifying
  whether perturbation-unique cell states are recovered — integration workflows deliberately
  align treatments and can erase the perturbation signal ICAT is designed to find.
- The input lacks treatment/control labels, since ICAT's control-guided clustering
  and semi-supervised Louvain step require a `control` annotation to fix control labels.
- There are no ground-truth labels or accepted reference clusters for the dataset,
  making ARI and the Davies-Bouldin comparison across methods uninterpretable.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: ICAT
  role: 'Method under test: control-defined unsupervised clustering, NCFS gene weighting,
    and semi-supervised Louvain clustering of perturbed cells'
  repo: https://github.com/BradhamLab/icat
- name: icat-sc
  role: Pip-installable ICAT package providing models.icat and optimize grid-search
    utilities used to run and tune the clustering
  repo: https://github.com/BradhamLab/icat
- name: scanpy
  role: Preprocessing (log1p normalization, PCA, neighbors), Louvain clustering for
    baselines, and UMAP on the ICAT upweighted matrix
- name: AnnData
  role: Data structure holding counts, obs treatment labels, and the X_icat upweighted
    matrix in obsm
- name: numpy
  role: Array operations supporting preprocessing and the simulated-data benchmark
    setup
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:337feca994177d472800e4919a5e510712e0d3e441599f0fb47af18e837ce478
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# perturbation-clustering-benchmarking

## Summary

Benchmark ICAT's control-guided semi-supervised clustering against integration + Louvain workflows (Seurat, Scanorama, Harmony) on real scRNA-seq perturbation datasets, scoring cluster quality with ARI, LISI, and the Davies-Bouldin index.

## When to use

Use when comparing cell-state clustering methods on single-cell perturbation data (e.g. drug-treated vs control, IFN-beta stimulated PBMCs) where cluster quality must be quantified against ground-truth labels and mixing metrics.

## When NOT to use

- The analysis goal is batch integration for visualization rather than quantifying whether perturbation-unique cell states are recovered — integration workflows deliberately align treatments and can erase the perturbation signal ICAT is designed to find.
- The input lacks treatment/control labels, since ICAT's control-guided clustering and semi-supervised Louvain step require a `control` annotation to fix control labels.
- There are no ground-truth labels or accepted reference clusters for the dataset, making ARI and the Davies-Bouldin comparison across methods uninterpretable.

## Inputs

- Raw scRNA-seq count matrices for perturbation datasets (Tian CellMix H2228/HCC827/H1975/Mixed, Kagohara cetuximab vs PBS, Kang control vs IFN-beta) from archived_data.tar.gz in BradhamLab/ICAT_manuscript
- Ground-truth cell type/state labels for ARI computation
- Treatment/batch labels (e.g. control vs perturbed) as AnnData obs columns

## Outputs

- fig4_benchmark_metrics.csv with ARI, LISI, and Davies-Bouldin values per method-dataset pair
- fig4_benchmark.png method-comparison plot reproducing Figure 4
- Per-cell cluster assignments in AnnData obs columns (sslouvain for ICAT; Louvain for baselines)

## How to apply

Retrieve the Tian CellMix (H2228/HCC827/H1975/Mixed), Kagohara (cetuximab vs PBS), and Kang (control vs IFN-beta) datasets from the BradhamLab/ICAT_manuscript archive and preprocess each in Scanpy with log1p normalization, PCA, and a neighbor graph. Run ICAT via `models.icat` with treatment labels so control cells define Louvain clusters and semi-supervised Louvain assigns all cells, reading the upweighted matrix from `adata.obsm["X_icat"]`; before clustering, grid-search hyperparameters with `optimize.optimize_louvain` (Calinski-Harabasz over n_neighbors and resolution) and `optimize.optimize_ncfs` (sigma and reg via MCC in weighted KNN cross-validation). Run the three baseline integration + Louvain workflows and the extended ICATSeurat, ICATScan, and ICATHarm workflows on the same preprocessed objects. For every method-dataset pair, compute ARI against ground-truth cell labels, LISI for treatment-batch mixing, and the Davies-Bouldin index for cluster separation. Assemble metrics into fig4_benchmark_metrics.csv and render fig4_benchmark.png reproducing Figure 4.

## Related tools

- **ICAT** (Method under test: control-defined unsupervised clustering, NCFS gene weighting, and semi-supervised Louvain clustering of perturbed cells) — https://github.com/BradhamLab/icat
- **icat-sc** (Pip-installable ICAT package providing models.icat and optimize grid-search utilities used to run and tune the clustering) — https://github.com/BradhamLab/icat
- **scanpy** (Preprocessing (log1p normalization, PCA, neighbors), Louvain clustering for baselines, and UMAP on the ICAT upweighted matrix)
- **AnnData** (Data structure holding counts, obs treatment labels, and the X_icat upweighted matrix in obsm)
- **numpy** (Array operations supporting preprocessing and the simulated-data benchmark setup)

## Examples

```
from icat import models; model = models.icat(ctrl_value="control", ncfs_kws={'reg': 1, 'sigma': 3}, neighbor_kws={'n_neighbors': 15}, cluster_kws={'resolution': 0.75}); out = model.cluster(adata, adata.obs['treatment'])
```

## Evaluation signals

- ARI values fall within [0, 1] for every method-dataset pair.
- The relative ICAT-vs-integration (Seurat, Scanorama, Harmony) comparison matches the trends reported in Figure 4 of the paper.
- ICAT outperforms the integration workflows on Kagohara and Kang datasets by ARI, LISI, and DB metrics.
- ICAT output contains a populated obs['sslouvain'] column and an upweighted matrix in adata.obsm['X_icat'] before metric computation.
- Louvain and NCFS hyperparameters are selected via the provided grid-search functions (optimize_louvain on Calinski-Harabasz; optimize_ncfs on weighted-KNN MCC) rather than arbitrary defaults.

## Limitations

- The ICAT_manuscript pipeline was run on a high-performance cluster with CentOS; 'It is not guaranteed to run on your system out-of-the-box.'
- ICAT requires Linux for pip installation ('ICAT can be installed on linux machines using pip').
- No discussion-section content was extractable, so caveats about metric limitations (e.g. LISI's reward of treatment mixing) could not be verified from the paper text.

## Evidence

- [readme] ICAT is a tool developed to better identify cell states in scRNAseq experiments where perturbations or some other biologic heterogeneity is present, such as gene knock-outs.: "ICAT is a tool developed to better identify cell states in scRNAseq experiments where perturbations or some other biologic heterogeneity is present, such as gene knock-outs."
- [intro] Semi-supervised clustering behavior: "perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells"
- [readme] Grid-search hyperparameters before full workflow: "we've provided grid search functions to find the "best" `n_neighbor` and `resolution`
parameters for Louvain and Semi-supervised Louvain clustering steps"
- [readme] Upweighted matrix access for downstream metrics/visualization: "it is possible to pass the upweighted count matrix
(found in `adata.obsm["X_icat"]`) to these algorithms"
- [other] Reproduction environment caveat: "This analysis was run on a high performance cluster with a CentOS operating system.

It is not guaranteed to run on your system out-of-the-box."
