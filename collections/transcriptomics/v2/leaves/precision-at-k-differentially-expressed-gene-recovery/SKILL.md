---
name: precision-at-k-differentially-expressed-gene-recovery
description: Use when benchmarking feature selection on simulated single-cell data
  with known ground-truth DE genes and you need an objective overlap score between
  the top-k ranked features and the truth set.
when_to_use_negative:
- The dataset is real single-cell data without known ground-truth DE genes — precision@k
  cannot be computed; use task-based evaluation instead (e.g., cell type classification
  accuracy or pseudotime correlation)
- Evaluating only one feature selection method without a comparison baseline — the
  metric is designed for benchmarking 12 strategies against each other
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: delve_benchmark
  role: Provides the fs class with select() and evaluate_select() methods to run the
    12 feature selection strategies and score them on trajectory/DE-recovery tasks
  repo: https://github.com/jranek/delve_benchmark
- name: Splatter
  role: R package used (via scprep's SplatSimulate wrapper) to simulate linear trajectories
    with ground-truth differentially expressed genes per cluster/path
- name: DELVE
  role: Unsupervised feature selection method being benchmarked; ranks features by
    association with the cell trajectory graph and supplies the top-100 feature list
    scored by precision@k
  repo: https://github.com/jranek/delve
- name: SketchKH
  role: Kernel-herding subsampling used within DELVE to generate the 1000 subsampled
    neighborhoods
  repo: https://github.com/CompCy-lab/SketchKH
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:45774600e40649cb0bc48a57aa18b7f50bd7a6cfaf117e8c0d88ed8097493b62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve_benchmark
  tools:
  - DELVE
  - Python
  - Scanpy
  - SketchKH
  - Splatter
  - delve_benchmark
  - scikit-learn
  - scprep (SplatSimulate)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# Precision-at-k differentially expressed gene recovery

## Summary

A benchmark metric that scores a feature selection method by the fraction of its top-k selected features (here k=100) that overlap the known ground-truth differentially expressed genes in simulated scRNA-seq data. Used in the DELVE paper to quantify which of 12 strategies best recovers biologically relevant genes from Splatter simulations.

## When to use

Use when benchmarking feature selection on simulated single-cell data with known ground-truth DE genes and you need an objective overlap score between the top-k ranked features and the truth set.

## When NOT to use

- The dataset is real single-cell data without known ground-truth DE genes — precision@k cannot be computed; use task-based evaluation instead (e.g., cell type classification accuracy or pseudotime correlation)
- Evaluating only one feature selection method without a comparison baseline — the metric is designed for benchmarking 12 strategies against each other

## Inputs

- Preprocessed simulated single-cell data as an anndata (.h5ad) object (cells x features) with ground-truth cluster/path labels in adata.obs
- Ground-truth differentially expressed gene list per cluster or path from the Splatter simulation parameters
- Ranked feature list from each feature selection method (top 100 selected features)
- Feature selection method configuration (e.g., DELVE hyperparameters: k=10 affinity graph, c=3 modules, 1000 KH subsampled neighborhoods)

## Outputs

- Precision@k score (overlap of top-100 selected features with ground-truth DE genes) per method per fold/replicate
- Aggregated median precision@k table across simulation replicates and noise conditions (BCV 0.1/0.25/0.5), written to CSV
- Ranked feature lists returned by model.select() or evaluate_select()

## How to apply

First simulate scRNA-seq data with known ground truth, e.g., Splatter path simulations via scprep's SplatSimulate wrapper (1500 cells, 500 genes, 6 clusters, BCV varied across 0.1, 0.25, 0.5), and extract the ground-truth differentially expressed genes per cluster/path. Within each 3-fold cross-validation split, run the feature selection method of interest (e.g., DELVE with k=10 weighted k-NN affinity graph, 1000 kernel-herding subsampled neighborhoods, c=3 modules, dynamic seeds from 10 random KMeans initializations) and retain the top 100 ranked features. Compute precision@k as the overlap of the selected features with the Splatter ground-truth DE gene set, then aggregate median scores across simulation replicates and noise conditions. Precision@k can be computed alongside downstream task metrics (k-NN cell type classification accuracy, diffusion pseudotime Kendall rank correlation) for a fuller comparison; in the DELVE study, similarity-based methods (DELVE, Laplacian score, Hotspot) achieved the highest DE gene recovery. The delve_benchmark package's fs class wraps this selection-plus-evaluation loop.

## Related tools

- **delve_benchmark** (Provides the fs class with select() and evaluate_select() methods to run the 12 feature selection strategies and score them on trajectory/DE-recovery tasks) — https://github.com/jranek/delve_benchmark
- **Splatter** (R package used (via scprep's SplatSimulate wrapper) to simulate linear trajectories with ground-truth differentially expressed genes per cluster/path)
- **DELVE** (Unsupervised feature selection method being benchmarked; ranks features by association with the cell trajectory graph and supplies the top-100 feature list scored by precision@k) — https://github.com/jranek/delve
- **SketchKH** (Kernel-herding subsampling used within DELVE to generate the 1000 subsampled neighborhoods) — https://github.com/CompCy-lab/SketchKH

## Examples

```
from delve_benchmark import fs; model = fs(adata=adata, fs_method=delve_benchmark.tl.run_delve_fs, feature_threshold=100, labels_key='labels'); predicted_features, score_df = model.evaluate_select()
```

## Evaluation signals

- Precision@k values fall in [0, 1]; with k=100 and 500 genes, a random baseline should be clearly outperformed by similarity-based methods
- Median precision@k across the 10 simulation datasets per BCV condition (0.1, 0.25, 0.5) is reported; scores should degrade monotonically with increasing noise (lower signal-to-noise ratio)
- DELVE/Laplacian/Hotspot (similarity-based methods) should show the highest DE gene recovery, consistent with the published benchmark
- Output CSV contains one row per feature selection method per noise condition with aggregated median metric scores
- Selected feature count matches feature_threshold=100 for every method and fold

## Limitations

- Feature selection can greatly bias interpretation of the underlying cellular trajectory; careful consideration should be made when performing feature selection for trajectory analysis
- Simulated-data precision@k depends on Splatter's simulation parameters (mean_rate=0.0173, mean_shape=0.54, lib_loc=12.6, lib_scale=0.423, bcv_df=90.2); results may not transfer to real data
- Evaluation at a fixed k=100 means methods are compared only on their top-ranked features, not the full ranking

## Evidence

- [discussion] similarity-based feature selection achieved highest DE recovery: "similarity-based feature selection methods (e.g., DELVE, Laplacian score, and Hotspot) achieved the highest recovery of differentially expressed genes within a cell type"
- [discussion] benchmark of twelve methods on trajectory-relevant features: "we benchmarked twelve feature selection methods29,49,53,55–58 on their ability to identify biologically relevant features for trajectory analysis from single-cell RNA sequencing data"
- [readme] fs class evaluation methods: [evidence span withheld — anchor status: external_doc]
- [discussion] feature selection biases trajectory interpretation: "feature selection can greatly bias the interpretation of the underlying cellular trajectory45, thus careful consideration should be made when performing feature selection for trajectory analysis"
