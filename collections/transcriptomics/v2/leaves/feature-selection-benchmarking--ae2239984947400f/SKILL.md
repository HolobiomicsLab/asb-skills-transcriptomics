---
name: feature-selection-benchmarking
description: Use when comparing unsupervised feature selection methods for trajectory
  preservation in scRNA-seq or iterative immunofluorescence (4i) data with known ground
  truth labels (cell types, cell cycle phase, latent pseudotime).
when_to_use_negative:
- When the feature set is already a small, curated panel of informative markers —
  the benchmark is designed for high-dimensional lists containing noisy or irrelevant
  features
- When ground truth annotations are available and the goal is purely supervised predictive
  modeling rather than unsupervised trajectory-preserving feature selection
- When the data have no cell-type or trajectory structure to preserve (e.g., static
  bulk measurements)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: DELVE
  role: Primary unsupervised feature selection method benchmarked; run with k=10 weighted
    kNN affinity graph, 1000 subsampled neighborhoods, dynamic seed feature modules
  repo: https://github.com/jranek/delve
- name: delve_benchmark
  role: Benchmarking package providing the `fs` class with select() and evaluate_select()
    and 12 feature selection functions (run_delve_fs, laplacian_score_fs, hotspot_fs,
    mcfs_fs, scmer_fs, hvg, variance_score, etc.)
  repo: https://github.com/jranek/delve_benchmark
- name: SketchKH
  role: Kernel-herding sketching used to obtain the 1000 subsampled neighborhoods
    consumed by DELVE feature selection
  repo: https://github.com/CompCy-lab/SketchKH
- name: Splatter
  role: R package (v1.18.2) used via the scprep SplatSimulate wrapper to generate
    simulated linear trajectories with ground truth differential expression
- name: SymSim
  role: R package (v0.0.0.9000) used to simulate five tree differentiation trajectories
    with varying mRNA capture efficiency
- name: Scanpy
  role: Provides diffusion pseudotime (DPT) inference on k=10 kNN graphs used to score
    pseudotime preservation
- name: scikit-learn
  role: Provides k-NN (k=3) and SVM classifiers with nested 10-fold CV and KMeans++
    clustering used for evaluation metrics
- name: Hotspot
  role: Similarity-based feature selection baseline (negative binomial null) included
    in the 12-method comparison
provenance:
  source_task_ids:
  - task_002
  - task_008
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:b60d7e3c71b5f76b68af652aa05ec5eba07b2c98fde30cd8715b37d8aabc7751
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - Hotspot
  - Python
  - Scanpy
  - SketchKH
  - Slingshot
  - Splatter
  - SymSim
  - UMAP (uniform manifold approximation and projection)
  - delve_benchmark
  - scikit-learn
  - scprep (SplatSimulate)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# feature-selection-benchmarking

## Summary

Benchmark twelve feature selection strategies (DELVE, Laplacian Score, Hotspot, MCFS, SCMER, neighborhood variance, HVG, max variance, random forest, dynamic seed, all features, random) on their ability to preserve cell types and cell-type transitions in single-cell RNA-seq and proteomic 4i imaging data, using classification, clustering, precision@k, and pseudotime-correlation metrics.

## When to use

Use when comparing unsupervised feature selection methods for trajectory preservation in scRNA-seq or iterative immunofluorescence (4i) data with known ground truth labels (cell types, cell cycle phase, latent pseudotime).

## When NOT to use

- When the feature set is already a small, curated panel of informative markers — the benchmark is designed for high-dimensional lists containing noisy or irrelevant features
- When ground truth annotations are available and the goal is purely supervised predictive modeling rather than unsupervised trajectory-preserving feature selection
- When the data have no cell-type or trajectory structure to preserve (e.g., static bulk measurements)

## Inputs

- Preprocessed single-cell .h5ad files from Zenodo record 10534873 (e.g., adata_RPE.h5ad: 2759 cells × 241 imaging features; PDAC 4i: 9 cell lines × 253 features) with ground truth cell cycle phase/age annotations in adata.obs
- Simulated datasets with known ground truth: Splatter linear paths (1500 cells × 500 genes × 6 clusters, BCV = 0.1/0.25/0.5) and SymSim tree trajectories (10,000 cells × 20,000 genes × 4 cell types, σ = 0.6/0.4)
- PCA embeddings (n_pcs = 50) of simulated scRNA-seq data for methods that operate on low-dimensional representations
- Ground truth labels: cell type/cluster labels, latent progression steps, cell cycle phases (G0, G1, S, G2, M), or cell cycle age in hours

## Outputs

- Ranked feature list per method (top p features, e.g., predicted_features returned by fs.select())
- Summary CSV of median metric scores per method × condition (e.g., symsim_feature_selection_results.csv) containing k-NN/SVM classification accuracy, NMI, precision@k, RMSE in hours, and Kendall pseudotime correlation
- Split proliferative vs. arrested lineage pseudotime correlations for cell cycle datasets

## How to apply

Instantiate the `fs` class from the delve_benchmark package and call `select()` to run any of the 12 feature selection functions (e.g., `run_delve_fs`, `laplacian_score_fs`, `hotspot_fs`, `mcfs_fs`, `scmer_fs`, `hvg`, `variance_score`), then `evaluate_select()` to score each method. Select a fixed number of top-ranked features per method (p=30 for 4i datasets with 241–253 imaging features, p=100 for Splatter simulations with 500 genes, and p=50–2000 for SymSim trajectories with 20,000 genes) and evaluate with k-NN classification (k=3, 3-fold stratified CV), SVM with nested 10-fold CV for cell cycle phase, KMeans++ NMI against ground truth labels, precision@k against random-forest-derived reference features, and diffusion pseudotime (Scanpy DPT on a k=10 kNN graph) scored by Kendall rank correlation to ground truth progression. Run DELVE with a weighted k-NN affinity graph (k=10), 1000 kernel-herding subsampled neighborhoods, and system-appropriate cluster counts (c=3–5 modules; e.g., n_clusters=10 for PDAC cell lines, 5 for RPE). Use R packages Splatter v1.18.2 and SymSim v0.0.0.9000 to generate simulated ground truth, and aggregate median metric scores per method × noise level × p combination into a results CSV. The rationale is that variance-based selection is overwhelmed by noisy features, so the benchmark tests whether similarity-based methods recover trajectory-relevant features at small subset sizes.

## Related tools

- **DELVE** (Primary unsupervised feature selection method benchmarked; run with k=10 weighted kNN affinity graph, 1000 subsampled neighborhoods, dynamic seed feature modules) — https://github.com/jranek/delve
- **delve_benchmark** (Benchmarking package providing the `fs` class with select() and evaluate_select() and 12 feature selection functions (run_delve_fs, laplacian_score_fs, hotspot_fs, mcfs_fs, scmer_fs, hvg, variance_score, etc.)) — https://github.com/jranek/delve_benchmark
- **SketchKH** (Kernel-herding sketching used to obtain the 1000 subsampled neighborhoods consumed by DELVE feature selection) — https://github.com/CompCy-lab/SketchKH
- **Splatter** (R package (v1.18.2) used via the scprep SplatSimulate wrapper to generate simulated linear trajectories with ground truth differential expression)
- **SymSim** (R package (v0.0.0.9000) used to simulate five tree differentiation trajectories with varying mRNA capture efficiency)
- **Scanpy** (Provides diffusion pseudotime (DPT) inference on k=10 kNN graphs used to score pseudotime preservation)
- **scikit-learn** (Provides k-NN (k=3) and SVM classifiers with nested 10-fold CV and KMeans++ clustering used for evaluation metrics)
- **Hotspot** (Similarity-based feature selection baseline (negative binomial null) included in the 12-method comparison)

## Examples

```
from delve_benchmark.tl import run_delve_fs; feats = run_delve_fs(adata, k=10, num_subsamples=1000, n_clusters=10)
```

## Evaluation signals

- Similarity-based methods (DELVE, Laplacian Score, Hotspot) should achieve higher cell-type classification accuracy and pseudotime correlation at small feature subsets (e.g., 100 genes), while variance-based methods need ~2000 genes (Supplementary Fig. 13 expectation)
- On Splatter simulations, DELVE should yield the highest median k-NN classification accuracy across noise levels (0.937 high, 0.882 medium, 0.734 low SNR)
- On the RPE 4i dataset, DELVE should reproduce median SVM accuracy 0.960 (vs. random forest 0.957), precision@30 0.800, NMI 0.599, RMSE 1.806 h, and pseudotime correlation 0.656 (proliferation) / 0.405 (arrest)
- Output CSV must contain one row per method × α_mean × p (or per method × cell line) with all four metric columns populated
- Random/all-feature baselines should underperform DELVE on classification and NMI, providing a sanity floor for the benchmark

## Limitations

- Feature selection can greatly bias the interpretation of the underlying cellular trajectory, so careful consideration is required when applying it prior to trajectory inference
- DELVE leverages all dynamic modules equally when constructing the approximate cellular trajectory; excluding unwanted (e.g., confounding) modules is possible but not implemented
- Similarity-based methods rely on the cell-similarity graph from the full dataset and can fail to identify relevant features when the number of noisy features outweighs the number of informative ones

## Evidence

- [discussion] Benchmarked twelve feature selection methods on identifying biologically relevant features for trajectory analysis: "we benchmarked twelve feature selection methods29,49,53,55–58 on their ability to identify biologically relevant features for trajectory analysis from single-cell RNA sequencing data and proteomic"
- [discussion] Similarity-based methods achieved the highest recovery of DE genes and classification accuracy: "similarity-based feature selection methods (e.g., DELVE, Laplacian score, and Hotspot) achieved the highest recovery of differentially expressed genes within a cell type or along a cellular lineage"
- [readme] fs class provides select() and evaluate_select() methods: [evidence span withheld — anchor status: external_doc]
- [readme] Preprocessed .h5ad benchmark data available on Zenodo: "You can download all of the preprocessed single-cell datasets (`.h5ad` files) from the [Zenodo](https://zenodo.org/records/10534873) repository."
- [discussion] DELVE feature selection improves downstream trajectory/gene program resolution: "performing feature selection with DELVE prior to performing trajectory inference improved the identification and resolution of gene programs"
