---
name: perturbation-simulated-data-evaluation
description: Use when you need to quantitatively compare cell-state clustering methods
  on single-cell perturbation data where ground-truth population labels exist (real
  or simulated), especially under varying perturbation severity, marker-gene signal,
  or cell type proportions.
when_to_use_negative:
- You have no ground-truth labels (real experimental data without known population
  structure) — simulated benchmarks here depend on known labels to score cluster recovery.
- Your goal is cell-state discovery on a single unperturbed condition rather than
  comparing control-defined states against perturbed cells across treatments.
- Your data are not single-cell transcriptomes with matched control and perturbed
  populations (e.g. bulk RNA-seq or spatial-only assays).
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ICAT
  role: Perturbation-aware clustering method under evaluation; identifies control-defined
    states via unsupervised Louvain, weights genes with NCFS, and assigns semi-supervised
    (sslouvain) clusters
  repo: https://github.com/BradhamLab/icat
- name: icat-sc
  role: pip-installable Python package providing icat.models.icat, icat.optimize (optimize_louvain,
    optimize_ncfs), and icat.simulate utilities used to run the pipeline
  repo: https://github.com/BradhamLab/icat
- name: scanpy
  role: Preprocessing (sc.pp.log1p, sc.pp.pca, sc.pp.neighbors, sc.tl.louvain) and
    visualization (sc.tl.umap) of the concatenated AnnData; backbone of the ICAT workflow
- name: AnnData
  role: Data structure holding the concatenated control/perturbed matrix, treatment
    labels, and returned sslouvain assignments and X_icat upweighted matrix
- name: Seurat
  role: 'Integration baseline: Seurat+Louvain clustering run on the same simulated
    datasets for comparison'
- name: Scanorama
  role: 'Integration baseline: Scanorama+Louvain clustering run on the same simulated
    datasets for comparison'
- name: Harmony
  role: 'Integration baseline: Harmony+Louvain clustering run on the same simulated
    datasets for comparison'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:15141b2fe445fcb2c4809b0ce66b385f9ab0975fa13aae6a99a47c6912b7dcb2
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# perturbation-simulated-data-evaluation

## Summary

Benchmark a perturbation-aware clustering method (ICAT) against integration workflows (Seurat, Scanorama, Harmony, no-integration) on simulated scRNA-seq control/perturbed datasets, scoring cluster recovery against ground-truth labels across perturbation severity, marker-gene count, and cell-proportion conditions.

## When to use

Use when you need to quantitatively compare cell-state clustering methods on single-cell perturbation data where ground-truth population labels exist (real or simulated), especially under varying perturbation severity, marker-gene signal, or cell type proportions.

## When NOT to use

- You have no ground-truth labels (real experimental data without known population structure) — simulated benchmarks here depend on known labels to score cluster recovery.
- Your goal is cell-state discovery on a single unperturbed condition rather than comparing control-defined states against perturbed cells across treatments.
- Your data are not single-cell transcriptomes with matched control and perturbed populations (e.g. bulk RNA-seq or spatial-only assays).

## Inputs

- Simulated single-cell control/perturbed datasets as AnnData objects (perturbation severity 1-25%, marker-gene counts 10-105, varied cell-proportion conditions) from archived_data.tar.gz in BradhamLab/ICAT_manuscript
- Ground-truth population labels for each simulated dataset
- Treatment labels (control/perturbed) in adata.obs['treatment']

## Outputs

- Per-method, per-condition clustering assignments (e.g. adata.obs['sslouvain'] for ICAT)
- Aggregated clustering-accuracy metrics (ARI, LISI, F1) comparing ICAT vs Seurat, Scanorama, Harmony, and no-integration baselines
- Regenerated Fig 3F/G comparison panel saved as fig3F_G_metric_panel.png

## How to apply

Load the archived simulated control/perturbed AnnData datasets (perturbation severity 1-25%, marker-gene counts 10-105, varied cell-proportion conditions) with ground-truth population labels from BradhamLab/ICAT_manuscript. Run the ICAT pipeline: log1p-normalize the concatenated AnnData, grid-search Louvain parameters (n_neighbors 3-50, resolution 0.3-1.2) on control cells via optimize_louvain maximizing the Calinski-Harabasz Index, grid-search NCFS sigma (0.5-3) and reg (0.25-3) via optimize_ncfs using MCC of a weighted 5-NN under k-fold cross-validation, then cluster with models.icat given treatment labels and read the sslouvain assignments. Run baselines on identical data: Seurat+Louvain, Scanorama+Louvain, Harmony+Louvain, and no-integration Louvain on the combined control/perturbed matrix. Score every method against ground-truth labels for every severity, marker-gene, and cell-proportion condition, then aggregate per-method/per-condition metrics to regenerate the Fig 3F/G comparison panel (fig3F_G_metric_panel.png).

## Related tools

- **ICAT** (Perturbation-aware clustering method under evaluation; identifies control-defined states via unsupervised Louvain, weights genes with NCFS, and assigns semi-supervised (sslouvain) clusters) — https://github.com/BradhamLab/icat
- **icat-sc** (pip-installable Python package providing icat.models.icat, icat.optimize (optimize_louvain, optimize_ncfs), and icat.simulate utilities used to run the pipeline) — https://github.com/BradhamLab/icat
- **scanpy** (Preprocessing (sc.pp.log1p, sc.pp.pca, sc.pp.neighbors, sc.tl.louvain) and visualization (sc.tl.umap) of the concatenated AnnData; backbone of the ICAT workflow)
- **AnnData** (Data structure holding the concatenated control/perturbed matrix, treatment labels, and returned sslouvain assignments and X_icat upweighted matrix)
- **Seurat** (Integration baseline: Seurat+Louvain clustering run on the same simulated datasets for comparison)
- **Scanorama** (Integration baseline: Scanorama+Louvain clustering run on the same simulated datasets for comparison)
- **Harmony** (Integration baseline: Harmony+Louvain clustering run on the same simulated datasets for comparison)

## Examples

```
from icat import models; model = models.icat(ctrl_value='control', ncfs_kws={'reg': 1, 'sigma': 3}, neighbor_kws={'n_neighbors': 15}, cluster_kws={'resolution': 0.75}); out = model.cluster(adata, adata.obs['treatment']); print(out.obs['sslouvain'].unique())
```

## Evaluation signals

- The regenerated fig3F_G_metric_panel.png reproduces the reported pattern that only ICAT resolves the perturbation-unique cellular response across perturbation-severity and cell-proportion conditions.
- ICAT outperforms Scanorama and Seurat on all clustering metrics across simulated perturbation-severity and marker-gene-number experiments.
- ICAT's metric values remain highly stable as perturbation intensity (1-25%) and marker-gene counts (10-105) vary, consistent with reported robustness to low signal strength and high perturbation severity.
- Control-cell Louvain clustering was tuned by optimize_louvain maximizing the Calinski-Harabasz Index, and NCFS sigma/reg by MCC of a weighted 5-NN over k-fold cross-validation — verify these grid-search outputs precede the final clustering run.
- The sslouvain column in the returned AnnData contains cluster assignments in which originally identified control labels remain constant while experimental-condition cells are free to join any cluster.

## Limitations

- The manuscript analysis was run on a high-performance cluster with a CentOS operating system; the ICAT_manuscript pipeline 'is not guaranteed to run on your system out-of-the-box'.
- The document contains no extractable discussion-section content, so stated caveats/failure modes from the authors' discussion are unavailable.
- ICAT does not automatically compute UMAP/tSNE during clustering; visualization requires passing the upweighted matrix in adata.obsm['X_icat'] to external algorithms.
- Users are advised to grid-search Louvain and NCFS hyperparameters on their own data prior to running the full ICAT workflow; defaults may not transfer.

## Evidence

- [results] Across simulated perturbation-severity and marker-gene-number experiments, ICAT outperformed both Scanorama and Seurat on all metrics: [evidence span withheld — anchor status: not_found]
- [abstract] ICAT identifies a perturbation-unique cellular response: "We empirically validate ICAT in a developmental model and find that only ICAT identifies a perturbation-unique cellular response."
- [abstract] ICAT robustness scope: "While requiring no a priori knowledge of extant cell states or discriminatory marker genes, ICAT is robust to low signal strength, high perturbation severity, and disparate cell type proportions."
- [intro] Semi-supervised clustering definition: "perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells regardless of"
- [readme] Manuscript reproducibility caveat: [evidence span withheld — anchor status: external_doc]
- [readme] UMAP visualization of upweighted space: "the returned `adata` object already has neighbors defined in this upweighted space, so 
calculating a new UMAP is simple"
