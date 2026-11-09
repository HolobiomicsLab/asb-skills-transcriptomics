---
name: clustering-performance-metric-visualization
description: Use when comparing clustering outputs (e.g., ICAT sslouvain assignments
  vs. Seurat/Scanorama/Harmony/no-integration Louvain) across simulated perturbation-severity,
  marker-gene-count, or cell-proportion conditions against known ground-truth labels.
when_to_use_negative:
- You have a single clustering result with no ground-truth labels and only one method
  — metric comparison requires reference labels and/or multiple methods to contrast
- Your goal is to run the ICAT clustering itself on new experimental data rather than
  evaluate or benchmark clustering performance across conditions
- Clustering has already been evaluated and you only need to re-tune ICAT hyperparameters
  for a new dataset rather than produce comparison visualizations
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: icat-sc
  role: Provides models.icat clustering (sslouvain assignments to score), optimize.optimize_louvain
    (Calinski-Harabasz grid search), and optimize.optimize_ncfs (MCC-based sigma/reg
    grid search)
  repo: https://github.com/BradhamLab/icat
- name: scanpy
  role: AnnData handling, log1p normalization, PCA, neighbors/Louvain clustering,
    and UMAP visualization of scored clusters
- name: Seurat
  role: Baseline integration + Louvain clustering workflow whose clustering output
    is scored against ground truth
- name: Scanorama
  role: Baseline integration + Louvain clustering workflow whose clustering output
    is scored against ground truth
- name: Harmony
  role: Baseline integration + Louvain clustering workflow whose clustering output
    is scored against ground truth
- name: Snakemake
  role: Pipeline runner for reproducing the manuscript benchmark and regenerating
    the metric panels
  repo: https://github.com/BradhamLab/ICAT_manuscript
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:c2ae2836f330c091ac00f02d8f7acbd0892ff061bdb10e6e7543c0aadd3a259c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - Harmony
  - Scanorama
  - Seurat
  - Snakemake
  - icat-sc
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# clustering-performance-metric-visualization

## Summary

Scores single-cell clustering results against ground-truth population labels and visualizes cross-method, cross-condition performance (e.g., the ICAT vs. Seurat/Scanorama/Harmony/no-integration benchmark panels from the ICAT manuscript, regenerated as fig3F_G_metric_panel.png).

## When to use

Use when comparing clustering outputs (e.g., ICAT sslouvain assignments vs. Seurat/Scanorama/Harmony/no-integration Louvain) across simulated perturbation-severity, marker-gene-count, or cell-proportion conditions against known ground-truth labels.

## When NOT to use

- You have a single clustering result with no ground-truth labels and only one method — metric comparison requires reference labels and/or multiple methods to contrast
- Your goal is to run the ICAT clustering itself on new experimental data rather than evaluate or benchmark clustering performance across conditions
- Clustering has already been evaluated and you only need to re-tune ICAT hyperparameters for a new dataset rather than produce comparison visualizations

## Inputs

- Concatenated control/perturbed scRNA-seq AnnData objects from simulate.SingleCellDataset / simulate.perturb (log1p-normalized)
- Ground-truth population labels (e.g., 'Population' column) for simulated cells
- Treatment labels (adata.obs['treatment'] with 'control'/'perturbed' values)
- Archived datasets from BradhamLab/ICAT_manuscript archived_data.tar.gz

## Outputs

- Per-method, per-condition clustering metric values (e.g., ARI, LISI, F1 vs. ground truth)
- Optimized hyperparameters (louvain_n, louvain_r, sigma, reg, sslouvain_n, sslouvain_r)
- Comparison figure panel (fig3F_G_metric_panel.png) reproducing Fig 3F/G of the manuscript

## How to apply

Load the archived simulated control/perturbed AnnData objects with ground-truth population labels from the BradhamLab/ICAT_manuscript repository and run each clustering pipeline over the condition grid (perturbation severity 1–25%, 10–105 marker genes, varied cell proportions). Before benchmarking, tune each method's parameters on control cells: optimize_louvain grid-searches n_neighbors (3–50) and resolution (0.3–1.2) by maximizing the Calinski-Harabasz Index, and optimize_ncfs grid-searches sigma (0.5–3) and reg (0.25–3) by maximizing the MCC of a weighted 5-NN under k-fold cross-validation. Score each method's cluster assignments (e.g., the sslouvain column of the returned AnnData) against ground-truth labels for every condition. Aggregate per-method, per-condition metrics and regenerate the Fig 3F/G comparison panel, validating that only ICAT resolves the perturbation-unique cellular response while remaining stable as perturbation intensity and cell-state resolution vary.

## Related tools

- **icat-sc** (Provides models.icat clustering (sslouvain assignments to score), optimize.optimize_louvain (Calinski-Harabasz grid search), and optimize.optimize_ncfs (MCC-based sigma/reg grid search)) — https://github.com/BradhamLab/icat
- **scanpy** (AnnData handling, log1p normalization, PCA, neighbors/Louvain clustering, and UMAP visualization of scored clusters)
- **Seurat** (Baseline integration + Louvain clustering workflow whose clustering output is scored against ground truth)
- **Scanorama** (Baseline integration + Louvain clustering workflow whose clustering output is scored against ground truth)
- **Harmony** (Baseline integration + Louvain clustering workflow whose clustering output is scored against ground truth)
- **Snakemake** (Pipeline runner for reproducing the manuscript benchmark and regenerating the metric panels) — https://github.com/BradhamLab/ICAT_manuscript

## Examples

```
from icat import optimize; louvain_n, louvain_r = optimize.optimize_louvain(controls, min_neighbors=3, max_neighbors=50, min_res=0.3, max_res=1.2)
```

## Evaluation signals

- optimize_louvain returns parameters that maximize the Calinski-Harabasz Index over the n_neighbors=3–50, resolution=0.3–1.2 grid
- optimize_ncfs returns sigma/reg values that maximize MCC of a weighted 5-NN under k-fold cross-validation (n_splits=3)
- Clustered AnnData contains a populated 'sslouvain' column of final assignments
- Regenerated fig3F_G_metric_panel.png reproduces the reported pattern that only ICAT resolves the perturbation-unique cellular response across severity and cell-proportion conditions
- ICAT metrics remain highly stable as perturbation intensity (1–25%) and marker-gene count (10–105) vary, and outperform Scanorama and Seurat on all metrics

## Limitations

- ICAT does not automatically compute UMAP/t-SNE during clustering; visualizations must be built from the upweighted matrix in adata.obsm['X_icat']
- Hyperparameters are dataset-specific — the README recommends finding Louvain and NCFS parameters prior to the full workflow; all original values are in supplemental tables
- optimize_sslouvain caps semi-supervised optimization at max_cells=750, and defaults reuse control n_neighbors/resolution for sslouvain unless separately optimized
- The ICAT_manuscript pipeline was run on a CentOS HPC and 'is not guaranteed to run on your system out-of-the-box'; benchmark conclusions rest on simulated data plus one developmental model

## Evidence

- [readme] The final clustering is stored in the `sslouvain` column of the returned `AnnData` object.: "The final clustering is stored in the `sslouvain` column of the returned `AnnData` object."
- [readme] optimizing the Calinski-Harabasz Index over a grid of `n` and `r` values: "optimizing the Calinski-Harabasz Index over a grid of `n` and `r` values"
- [readme] find the "best" `sigma` and `reg` NCFS values by measuring the MCC in a weighted KNN over k-fold cross validation: "find the "best" `sigma` and `reg` NCFS values by measuring the MCC in a weighted KNN over k-fold cross validation"
- [abstract] While requiring no a priori knowledge of extant cell states or discriminatory marker genes, ICAT is robust to low signal strength, high perturbation severity, and disparate cell type proportions.: "While requiring no a priori knowledge of extant cell states or discriminatory marker genes, ICAT is robust to low signal strength, high perturbation severity, and disparate cell type proportions."
- [abstract] We empirically validate ICAT in a developmental model and find that only ICAT identifies a perturbation-unique cellular response.: "We empirically validate ICAT in a developmental model and find that only ICAT identifies a perturbation-unique cellular response."
