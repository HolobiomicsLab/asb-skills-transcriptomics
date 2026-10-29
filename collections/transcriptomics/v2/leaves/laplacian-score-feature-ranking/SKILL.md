---
name: laplacian-score-feature-ranking
description: Use when a preprocessed single-cell dataset (e.g. .h5ad from scRNA-seq
  or 4i imaging) already has dynamic seed features and you need to rank all features
  by trajectory smoothness before trajectory inference.
when_to_use_negative:
- No dynamic seed features exist yet — DELVE Step 1 must first identify dynamic modules
  from core regulatory complexes before the seed graph can be built
- Data are single-cell images or raw counts that have not been preprocessed/normalized
  into a cells x features matrix suitable for graph construction
- The goal is clustering or differential expression rather than selecting features
  that preserve cellular trajectories — variance-based or other selection may be more
  appropriate
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: DELVE
  role: 'Implements Step 2: builds the seed-feature affinity graph and ranks all features
    by Laplacian Score (delve_fs)'
  repo: https://github.com/jranek/delve
- name: Python
  role: Runtime environment for the DELVE package; executes the Laplacian score computation
    and CSV export
  repo: https://github.com/jranek/delve
- name: anndata
  role: Loads the preprocessed .h5ad objects (e.g. adata_RPE.h5ad) used as input to
    the ranking step
- name: delve_benchmark
  role: Provides laplacian_score_fs and seed_features functions to run and benchmark
    the Laplacian-score ranking against other feature selection methods
  repo: https://github.com/jranek/delve_benchmark
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:3ba4d255022bd4713a9f39a1c6efd6310c8fedeb5040fb024b73bd5998f21a7b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jranek/delve
  tools:
  - DELVE
  - EnrichR
  - Python
  - ScVelo
  - Scanpy
  - anndata
  - delve_benchmark
  - gseapy
  - scikit-learn
  - statsmodels
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# laplacian-score-feature-ranking

## Summary

DELVE Step 2: rank all measured features by Laplacian Score on an affinity graph built from the Step-1 dynamic seed modules, retaining globally smooth features that vary slowly among neighboring cells and excluding noisy, rapidly oscillating features.

## When to use

Use when a preprocessed single-cell dataset (e.g. .h5ad from scRNA-seq or 4i imaging) already has dynamic seed features and you need to rank all features by trajectory smoothness before trajectory inference.

## When NOT to use

- No dynamic seed features exist yet — DELVE Step 1 must first identify dynamic modules from core regulatory complexes before the seed graph can be built
- Data are single-cell images or raw counts that have not been preprocessed/normalized into a cells x features matrix suitable for graph construction
- The goal is clustering or differential expression rather than selecting features that preserve cellular trajectories — variance-based or other selection may be more appropriate

## Inputs

- Preprocessed AnnData .h5ad object (cells x features), e.g. adata_RPE.h5ad (2759 cells x 241 imaging-derived features) from Zenodo record 10534873
- List of dynamic seed features from DELVE Step 1 (e.g. 13 dynamic seed features for the RPE dataset)
- kNN graph parameters: k = 10 neighbors, kernel bandwidth sigma_i = distance to the 3rd nearest neighbor

## Outputs

- ranked_features.csv: feature table sorted in ascending Laplacian Score, including scores for all features
- Top-ranked feature subset (e.g. top 30 for RPE validation, or top p = 500 genes for the DE differentiation dataset) for downstream trajectory inference

## How to apply

Subset the expression matrix to the dynamic seed features identified in Step 1 (e.g. the 13 dynamic seed features of the RPE 4i dataset) and construct a weighted k-nearest-neighbor affinity graph on them (k = 10) with Gaussian edge weights whose kernel bandwidth sigma_i is set to the distance to the 3rd nearest neighbor (equation 6). Form the unnormalized graph Laplacian L = D - W_tilde with diagonal degree matrix D. For each of the measured features (e.g. all 241 imaging-derived features of adata_RPE.h5ad, or the top p = 500 genes for the DE differentiation dataset), mean-center by D and compute the Laplacian Score Lf = f_tilde^T L f_tilde / f_tilde^T D f_tilde (equation 9). Sort features in ascending order of Laplacian Score — low scores indicate expression that is similar among neighboring cells on the approximate trajectory graph — and retain the top-ranked features for downstream trajectory inference.

## Related tools

- **DELVE** (Implements Step 2: builds the seed-feature affinity graph and ranks all features by Laplacian Score (delve_fs)) — https://github.com/jranek/delve
- **Python** (Runtime environment for the DELVE package; executes the Laplacian score computation and CSV export) — https://github.com/jranek/delve
- **anndata** (Loads the preprocessed .h5ad objects (e.g. adata_RPE.h5ad) used as input to the ranking step)
- **delve_benchmark** (Provides laplacian_score_fs and seed_features functions to run and benchmark the Laplacian-score ranking against other feature selection methods) — https://github.com/jranek/delve_benchmark

## Examples

```
import anndata; from delve_fs import delve_fs; adata = anndata.read_h5ad('adata_RPE.h5ad'); ranked = delve_fs(adata, num_pc=50, k=10, num_ss=1000, num_c=5, seed_set=seed_features)
```

## Evaluation signals

- Top-30 ranked features on the RPE 4i dataset recover phase-specific features at precision@30 of about 0.800, the median reported for DELVE
- Laplacian scores are computed for all features and the ranked table is written to ranked_features.csv in ascending score order
- Retained features show expression that is similar amongst neighboring cells on the seed graph, while rapidly oscillating, noisy features receive high scores and are excluded
- On the DE differentiation dataset, the top p = 500 DELVE-ranked genes yield more lineage-specific transcription factors than HVG selection and RNA velocity by approximately 30% (Fig. 6g)

## Limitations

- Feature selection can greatly bias the interpretation of the underlying cellular trajectory; careful consideration is needed when performing feature selection for trajectory analysis
- All dynamic modules are leveraged equally when constructing the approximate cellular trajectory graph; excluding unwanted dynamic modules is possible but not implemented

## Evidence

- [results] Step-2 seed graph and Laplacian score: [evidence span withheld — anchor status: not_found]
- [results] Sorting and export: [evidence span withheld — anchor status: not_found]
- [results] Score interpretation: [evidence span withheld — anchor status: not_found]
- [readme] Package description: "DELVE is an unsupervised feature selection method for identifying a representative subset of dynamically-expressed molecular features that recapitulate cellular trajectories"
- [readme] Benchmark Laplacian implementation: [evidence span withheld — anchor status: not_found]
