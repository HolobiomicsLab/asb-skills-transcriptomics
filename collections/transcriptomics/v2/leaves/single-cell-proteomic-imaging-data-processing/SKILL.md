---
name: single-cell-proteomic-imaging-data-processing
description: Use when processing preprocessed single-cell proteomic imaging (4i) .h5ad
  datasets (e.g., PDAC cell lines or RPE with 241–253 imaging features) to select
  features that preserve cell cycle phases or cellular trajectories prior to trajectory
  inference.
when_to_use_negative:
- Input is already a curated feature set or the analysis goal is supervised feature
  selection with available manual annotations rather than unsupervised trajectory
  preservation
- Data are not single-cell profiles (e.g., bulk omics or imaging data without per-cell
  feature matrices), since the method relies on a cell-similarity graph over single-cell
  measurements
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: DELVE
  role: Unsupervised feature selection method identifying dynamically-expressed molecular
    features that recapitulate cellular trajectories; installed via pip as delve-fs
  repo: https://github.com/jranek/delve
- name: delve_benchmark
  role: Python package with the fs class (select() and evaluate_select()) and 12 feature-selection
    functions to benchmark trajectory preservation on 4i data
  repo: https://github.com/jranek/delve_benchmark
- name: SketchKH
  role: Distribution-aware kernel-herding sketching of cells per sample-set used within
    the DELVE pipeline for subsampling
  repo: https://github.com/CompCy-lab/SketchKH
- name: Python
  role: Implementation language; requires anndata, scanpy, scikit-learn, scipy, umap-learn==0.5.1,
    numpy >=1.26.4,<2.0
- name: UMAP (uniform manifold approximation and projection)
  role: Embedding/visualization of cells on the selected features within the trajectory
    analysis workflow
provenance:
  source_task_ids:
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:bf23df92b6fee4bb1f7615a862cd92cefb3d7d04f3ac62a0fd555d9362bb8c31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - Python
  - SketchKH
  - UMAP (uniform manifold approximation and projection)
  - delve_benchmark
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# single-cell-proteomic-imaging-data-processing

## Summary

Load and preprocess single-cell protein iterative immunofluorescence (4i) datasets as .h5ad AnnData objects and perform trajectory-preserving unsupervised feature selection (DELVE and benchmarked alternatives) so that downstream cell-type and cell-cycle trajectory analyses are not distorted by noisy or irrelevant imaging-derived features.

## When to use

Use when processing preprocessed single-cell proteomic imaging (4i) .h5ad datasets (e.g., PDAC cell lines or RPE with 241–253 imaging features) to select features that preserve cell cycle phases or cellular trajectories prior to trajectory inference.

## When NOT to use

- Input is already a curated feature set or the analysis goal is supervised feature selection with available manual annotations rather than unsupervised trajectory preservation
- Data are not single-cell profiles (e.g., bulk omics or imaging data without per-cell feature matrices), since the method relies on a cell-similarity graph over single-cell measurements

## Inputs

- Preprocessed single-cell proteomic imaging .h5ad file (cells x imaging-derived features, e.g. 2759 x 241 for RPE 4i or 253 features per PDAC cell line) from Zenodo record 10534873
- Dynamic seed feature list from DELVE Step 1 (e.g., 13 seed features)
- Optional ground-truth labels in adata.obs (e.g., cell cycle phase G0/G1/S/G2/M from manual gating of core cell cycle markers)

## Outputs

- Ranked feature table with Laplacian Scores for all features (ranked_features.csv)
- Top-30 selected trajectory-preserving features per method
- Benchmark score dataframe (classification accuracy, NMI, precision@30, pseudotime correlation) across cell lines and methods

## How to apply

Read the preprocessed .h5ad object (e.g., 2759 cells x 241 features for RPE, or 253 features for each of nine PDAC cell lines — BxPC3, CFPAC, HPAC, MiaPaCa, Pa01C, Pa02C, Pa16C, PANC1, UM53 — from Zenodo record 10534873) with anndata. In Step 1, subsample cells per sample-set via kernel-herding sketching (sketchKH) and identify dynamic seed feature modules built from core regulatory complexes to mitigate confounding variation. In Step 2, subset the expression matrix to the ~13 dynamic seed features, build a k=10 kNN affinity graph with Gaussian weights (bandwidth sigma_i set to the distance to the 3rd nearest neighbor), form the unnormalized graph Laplacian L = D - W, and rank all features by Laplacian Score, retaining globally smooth features. Select the top p = 30 ranked features (DELVE with k=10, num_subsamples=1000, n_clusters=10 for PDAC), then optionally benchmark against eleven alternatives (Laplacian Score, Hotspot with negative binomial null, neighborhood variance, HVG Seurat flavor with 20 bins, max variance, MCFS, SCMER, random forest, all features, random features, dynamic seed) using the delve_benchmark fs class.

## Related tools

- **DELVE** (Unsupervised feature selection method identifying dynamically-expressed molecular features that recapitulate cellular trajectories; installed via pip as delve-fs) — https://github.com/jranek/delve
- **delve_benchmark** (Python package with the fs class (select() and evaluate_select()) and 12 feature-selection functions to benchmark trajectory preservation on 4i data) — https://github.com/jranek/delve_benchmark
- **SketchKH** (Distribution-aware kernel-herding sketching of cells per sample-set used within the DELVE pipeline for subsampling) — https://github.com/CompCy-lab/SketchKH
- **Python** (Implementation language; requires anndata, scanpy, scikit-learn, scipy, umap-learn==0.5.1, numpy >=1.26.4,<2.0)
- **UMAP (uniform manifold approximation and projection)** (Embedding/visualization of cells on the selected features within the trajectory analysis workflow)

## Examples

```
import anndata; adata = anndata.read_h5ad('adata_RPE.h5ad'); import delve; model = delve.Module(fs_method='delve'); predicted_features = model.select(adata, num_subsamples=1000, n_clusters=10, k=10, feature_threshold=30)
```

## Evaluation signals

- Top-30 DELVE-ranked features recover phase-specific features at precision@30 of about 0.800 (median reported for the RPE dataset)
- DELVE and Hotspot achieve the highest median classification accuracy, NMI, precision@30, and Kendall pseudotime correlation across all nine PDAC cell lines versus the other ten methods
- KMeans++ (c = 5 centroids) clustering on selected features yields high normalized mutual information against ground truth cell cycle phase annotations
- Diffusion pseudotime (20 diffusion map components, k=10) on selected features shows high Kendall rank correlation with ground truth, correctly separating proliferative (high pRB) from arrested (low pRB) lineages
- SVM with nested 10-fold cross-validation on the 30-feature set accurately predicts cell cycle phase labels (G0, G1, S, G2, M)

## Limitations

- Feature selection can greatly bias the interpretation of the underlying cellular trajectory; careful consideration is needed before applying it
- DELVE could be improved by incorporating a set of previously established regulators
- All dynamic modules were leveraged equally when constructing the approximate cellular trajectory; excluding unwanted dynamic modules is possible but not implemented
- Similarity-based methods rely on the cell-similarity graph from the full dataset and can fail to identify relevant features when the number of noisy features outweighs the number of informative ones

## Evidence

- [readme] DELVE is an unsupervised feature selection method for identifying a representative subset of dynamically-expressed molecular features that recapitulate cellular trajectories from single-cell data (e.g. single-cell RNA sequencing, protein iterative immunofluorescence imaging): "DELVE is an unsupervised feature selection method for identifying a representative subset of dynamically-expressed molecular features that recapitulate cellular trajectories"
- [methods] construct affinity graph and Laplacian Score ranking: [evidence span withheld — anchor status: not_found]
- [results] validation precision@30: [evidence span withheld — anchor status: not_found]
- [readme] data access via Zenodo: "You can download all of the preprocessed single-cell datasets (`.h5ad` files) from the [Zenodo](https://zenodo.org/records/10534873) repository."
- [intro] noise overwhelms similarity-based selection: "they rely on the cell-similarity graph from the full dataset and can fail to identify relevant features when the number of noisy features outweighs the number of informative ones"
