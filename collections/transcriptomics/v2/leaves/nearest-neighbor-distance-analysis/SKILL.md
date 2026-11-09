---
name: nearest-neighbor-distance-analysis
description: Use when you must score how hard each cohort sample is to infer from
  representatives — e.g. comparing active-learning vs passive selection at equal representative
  counts, or deciding how many representatives are needed for a bulk cohort.
when_to_use_negative:
- No single-cell reference is available — the deep generative model enables single-cell
  level deconvolution only 'provided there is a single-cell reference'
- You want to extend the analysis to proteomics or other modalities — the article
  states this is future work not yet implemented and would require algorithm adjustments
- You already have real single-cell profiles for every sample, so inference difficulty
  is not a relevant selection criterion
tools:
- name: scSemiProfiler
  role: Runs the passive and type-II active-learning selectors and performs in silico
    inference (activeselect / scinfer) whose outputs are scored by Dsc
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: scikit-learn
  role: Provides the PCA dimensionality reduction and the Euclidean nearest-neighbor
    computation used to derive Dsc
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:5ec1231d414ca27aecf7298ca93bd4bf310175c5a35fd72a5e80e8e63c08e2d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - scSemiProfiler
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# nearest-neighbor-distance-analysis

## Summary

Quantifies single-cell in silico inference difficulty (Dsc) by computing, per sample, the average Euclidean distance between representative-sample cells and their K=1 nearest neighbors in the target sample within a PCA-reduced space, as used in scSemiProfiler to compare active-learning versus passive representative selection.

## When to use

Use when you must score how hard each cohort sample is to infer from representatives — e.g. comparing active-learning vs passive selection at equal representative counts, or deciding how many representatives are needed for a bulk cohort.

## When NOT to use

- No single-cell reference is available — the deep generative model enables single-cell level deconvolution only 'provided there is a single-cell reference'
- You want to extend the analysis to proteomics or other modalities — the article states this is future work not yet implemented and would require algorithm adjustments
- You already have real single-cell profiles for every sample, so inference difficulty is not a relevant selection criterion

## Inputs

- PCA-reduced bulk and single-cell data from the COVID-19 cohort (28-sample cohort, preprocessed with doublet removal, cell filtering, library normalization, log1p transformation; 100 PCs for >100 samples)
- Inferred (semi-profiled) single-cell data per target sample produced by the scSemiProfiler VAE-GAN deep generative model
- Representative sample assignments from the active-learning or passive selector

## Outputs

- Per-sample inference difficulty Dsc values (average Euclidean distance to K=1 nearest neighbors) aggregated across samples for each selector and representative count
- selection_comparison.csv tabulating Dsc for active-learning versus passive selection at each representative count

## How to apply

Reduce bulk and single-cell data with Principal Component Analysis implemented in Scikit-learn (100 PCs when there are more than 100 samples). After running in silico inference with the scSemiProfiler deep generative model (activeselect / scinfer), compute per-sample Dsc as the average Euclidean distance between representative cells and their K=1 nearest neighbors in the target sample within the PCA-reduced space, then aggregate across samples. Use this metric both inside the type-II active-learning selector's heterogeneity term (Db + λsc*Dsc + λpb*Dpb with λsc = λpb = 1) and as an external evaluation of selection quality. Tabulate Dsc for each selector and representative count and save as selection_comparison.csv. The rationale is that lower Dsc means inferred cells sit closer to true target-sample cells, i.e. easier, higher-fidelity inference.

## Related tools

- **scSemiProfiler** (Runs the passive and type-II active-learning selectors and performs in silico inference (activeselect / scinfer) whose outputs are scored by Dsc) — https://github.com/mcgilldinglab/scSemiProfiler
- **scikit-learn** (Provides the PCA dimensionality reduction and the Euclidean nearest-neighbor computation used to derive Dsc)

## Examples

```
from sklearn.neighbors import NearestNeighbors; d, _ = NearestNeighbors(n_neighbors=1).fit(target_pca).kneighbors(repr_pca); Dsc = d.mean()
```

## Evaluation signals

- The type-II active-learning selector yields lower aggregated Dsc than the passive uncertainty-style selector at the same representative count, validated at 28 representatives on the COVID-19 cohort
- Passive selection needs roughly 4 additional representatives (i.e. 32 instead of 28) to reach the same inference difficulty as active learning
- All per-sample Dsc values are non-negative finite Euclidean distances averaged over representative cells with K=1 nearest neighbors
- selection_comparison.csv contains paired Dsc entries for both selectors at each representative count with no missing samples

## Limitations

- Requires a single-cell reference: 'The tool’s deep generative model enables “single-cell level deconvolution” across various studies, provided there is a single-cell reference.'
- Extension to proteomics and other modalities is only future work, so Dsc is grounded in scRNA-seq PCA space only
- Comprehensive benchmarking against established standards is still required for new modalities

## Evidence

- [methods] Compute per-sample inference difficulty Dsc as the average Euclidean distance between representative cells and their K = 1 nearest neighbors in the target sample within the PCA-reduced space: [evidence span withheld — anchor status: not_found]
- [methods] PCA implemented in Scikit-learn: "we employ Principal Component Analysis (PCA)83–85, implemented in the Python package Scikit-learn86, to reduce the dimensionality"
- [results] active learning reduces inference difficulty at 28 representatives: [evidence span withheld — anchor status: not_found]
- [discussion] single-cell reference requirement: "The tool’s deep generative model enables “single-cell level deconvolution” across various studies, provided there is a single-cell reference."
- [discussion] VAE-GAN infers remaining samples: "employs a deep generative learning model, VAE-GAN, to infer the single-cell data of the remainder of the cohort."
