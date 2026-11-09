---
name: kernel-herding-distribution-sketching
description: Use when a downstream step (e.g., DELVE Step 1 module discovery) needs
  a distribution-preserving subsample of a large .h5ad single-cell dataset, or when
  per-sample sketches must retain cell-type frequencies.
when_to_use_negative:
- The analysis goal is exhaustive per-cell inference (e.g., diffusion pseudotime regression
  over all 500 selected genes) rather than distribution-level subsampling — sketching
  discards cells
- The dataset is a small .h5ad where the full distribution is already computationally
  tractable and subsampling would add no benefit
- No meaningful sample-set structure or feature space exists on which the Gaussian
  kernel and random Fourier features can be computed (e.g., raw unnormalized counts
  lacking preprocessing)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: sketchKH
  role: Python package providing the Kernel Herding sketch() function that selects
    a small, representative set of cells per sample-set from a preprocessed .h5ad
    object
  repo: https://github.com/CompCy-lab/SketchKH
- name: anndata
  role: Reads the preprocessed .h5ad input object and returns the downsampled adata_subsample
    annotated data object
- name: DELVE
  role: 'Downstream consumer: uses the sketchKH-sampled 1000 representative neighborhoods
    to compute delta_mean and cluster features into dynamic modules'
  repo: https://github.com/jranek/delve
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46773-z
    title: 'DELVE: feature selection for preserving biological trajectories in single-cell
      data'
  source_content_hash: sha256:c4d118b7f63de3304f686d2627f150257bcbd317efa76a9576170301667f68d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CompCy-lab/SketchKH
  tools:
  - DELVE
  - EnrichR
  - Python
  - ScVelo
  - Scanpy
  - anndata
  - gseapy
  - scikit-learn
  - sketchKH
  - statsmodels
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# Kernel Herding distribution sketching

## Summary

Subsamples a small, representative set of cells (or prototypical cellular neighborhoods) from a preprocessed single-cell dataset via Kernel Herding, preserving cell-type frequencies and the spectral properties of the original data. In DELVE, it supplies the ~1000-neighborhood subsample on which dynamic feature-module discovery is performed.

## When to use

Use when a downstream step (e.g., DELVE Step 1 module discovery) needs a distribution-preserving subsample of a large .h5ad single-cell dataset, or when per-sample sketches must retain cell-type frequencies.

## When NOT to use

- The analysis goal is exhaustive per-cell inference (e.g., diffusion pseudotime regression over all 500 selected genes) rather than distribution-level subsampling — sketching discards cells
- The dataset is a small .h5ad where the full distribution is already computationally tractable and subsampling would add no benefit
- No meaningful sample-set structure or feature space exists on which the Gaussian kernel and random Fourier features can be computed (e.g., raw unnormalized counts lacking preprocessing)

## Inputs

- Preprocessed annotated single-cell data object (.h5ad) with cells x features matrix (e.g., 2759 cells x 241 imaging features for RPE 4i; 5397 cells x 2000 genes for the DE differentiation scRNA-seq dataset)
- Key in adata.obs identifying sample-sets (e.g., 'FCS_File') when sketching multiple profiled samples
- sketching parameters: num_subsamples (e.g., 500 per sample-set or 1000 neighborhoods in DELVE), gamma, frequency_seed, n_jobs

## Outputs

- kh_indices: list of indices referencing the subsampled cells per sample-set
- adata_subsample: downsampled annotated data object (dimensions = num_subsamples x sample-sets x features)
- In DELVE context: the m=1000 prototypical cellular neighborhoods used to compute delta_mean for feature-module clustering

## How to apply

Read the preprocessed .h5ad annotated data object with anndata, then run Kernel Herding sketching from the sketchKH package, specifying the sample-set key in adata.obs (e.g., 'FCS_File' for multi-sample data), the number of cells to subsample per sample-set (DELVE uses num_subsamples=1000 representative cellular neighborhoods), and a frequency_seed for reproducibility. The Gaussian-kernel bandwidth can be set via gamma (DELVE uses a k-NN affinity graph with k=10 and bandwidth equal to the distance to the 3rd nearest neighbor before sketching the neighbor-averaged feature matrix Z). The function returns kh_indices (per-sample-set subsampled cell indices) and a downsampled adata_subsample; feed these indices into subsequent DELVE steps (delta_mean computation, KMeans++ clustering into c=5 modules). The rationale is that Kernel Herding preserves cell-type frequencies and the spectral properties of the original dataset, so downstream variance-based permutation tests and module assignments reflect the full distribution rather than a biased subsample.

## Related tools

- **sketchKH** (Python package providing the Kernel Herding sketch() function that selects a small, representative set of cells per sample-set from a preprocessed .h5ad object) — https://github.com/CompCy-lab/SketchKH
- **anndata** (Reads the preprocessed .h5ad input object and returns the downsampled adata_subsample annotated data object)
- **DELVE** (Downstream consumer: uses the sketchKH-sampled 1000 representative neighborhoods to compute delta_mean and cluster features into dynamic modules) — https://github.com/jranek/delve

## Examples

```
from sketchKH import *; kh_indices, adata_subsample = sketch(adata, sample_set_key='FCS_File', gamma=1, num_subsamples=500, frequency_seed=0, n_jobs=-1)
```

## Evaluation signals

- Sketched data should preserve cell-type frequencies: all major cell-types from the original sample-set remain present at proportional frequencies in adata_subsample
- Spectral properties of the between-cell affinity graph should be maintained by the subsample, which underpins DELVE's k-NN (k=10) graph reconstruction
- Determinism check: rerunning sketch() with the same frequency_seed and num_subsamples yields the same kh_indices
- Downstream DELVE Step 1 validation: the dynamic seed defined as the intersection across 10 KMeans++ initializations should recover 13 seed features (out of 241 imaging-derived features) for the RPE dataset
- Schema check: sketch() returns exactly two objects — kh_indices (list of index arrays, one per sample-set) and adata_subsample (cells x features annotated data object)

## Limitations

- Sketch size trades representativeness against information content: too few subsampled cells per sample-set may drop rare cell populations
- The gamma scale parameter for random Fourier frequency feature computation must be chosen appropriately; a poor bandwidth can distort the kernel and hence the sketch quality
- Kernel Herding assumes the feature representation captures the biologically relevant structure — sketching noisy or unpreprocessed features will faithfully preserve the wrong distribution

## Evidence

- [readme] SketchKH README: sketches preserve cell-type frequencies: [evidence span withheld — anchor status: external_doc]
- [readme] SketchKH README: sketch() returns kh_indices and adata_subsample: [evidence span withheld — anchor status: not_found]
- [methods] DELVE Step 1 workflow: Kernel Herding subsampling of 1000 neighborhoods: [evidence span withheld — anchor status: not_found]
- [readme] DELVE README: DELVE mitigates confounding variation via dynamic feature modules: "DELVE uses a bottom-up approach to mitigate the effect of unwanted sources of feature variation confounding inference, and instead models cell states from dynamic feature modules"
