---
name: inference-difficulty-benchmarking
description: Use when a cohort bulk dataset has candidate representative-selection
  strategies and you must quantify which strategy yields lower single-cell inference
  difficulty at a fixed representative budget.
when_to_use_negative:
- The task is cell-type-level bulk deconvolution rather than comparing representative-selection
  strategies — conventional deconvolution methods (CIBERSORTx, Bisque, DWLS, MuSiC)
  are constrained to cell-type-level resolution and cannot assess single-cell inference
  difficulty.
- Only one selection strategy is available, so no active-vs-passive comparison curve
  can be produced at matched representative counts.
- No single-cell reference or representative data exist at all — the tool requires
  a single-cell reference to perform single-cell-level decomposition.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3474
tools:
- name: scSemiProfiler
  role: Runs the passive and type-II active-learning representative selectors (activeselect)
    and performs in silico single-cell inference (scinfer) with the VAE-GAN deep generative
    model used to compute Dsc.
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: scikit-learn
  role: Provides the PCA implementation used to reduce the data dimensionality before
    distance computations and clustering (K-Means into B distinct clusters) for representative
    selection.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:7f0078b2c9cde2ed66f2fe1b72419a9299f62c5f1cb6cd9e88ed56285e07caf9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - scSemiProfiler
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# inference-difficulty-benchmarking

## Summary

Benchmarks representative-selection strategies (type-II active learning vs passive uncertainty-style selection) by measuring in silico single-cell inference difficulty — the average Euclidean distance between representative cells and their nearest neighbors in target samples — across representative counts using scSemiProfiler.

## When to use

Use when a cohort bulk dataset has candidate representative-selection strategies and you must quantify which strategy yields lower single-cell inference difficulty at a fixed representative budget.

## When NOT to use

- The task is cell-type-level bulk deconvolution rather than comparing representative-selection strategies — conventional deconvolution methods (CIBERSORTx, Bisque, DWLS, MuSiC) are constrained to cell-type-level resolution and cannot assess single-cell inference difficulty.
- Only one selection strategy is available, so no active-vs-passive comparison curve can be produced at matched representative counts.
- No single-cell reference or representative data exist at all — the tool requires a single-cell reference to perform single-cell-level decomposition.

## Inputs

- Preprocessed cohort bulk RNA-seq data (e.g., the 28-sample COVID-19 cohort, library-normalized and log1p-transformed)
- Preprocessed single-cell data for representative samples (doublet removal, cell filtering, library normalization, log1p)
- Selector configuration (number of representatives B, λsc = λpb = 1, K = 1 nearest neighbors)
- scSemiProfiler VAE-GAN deep generative model

## Outputs

- selection_comparison.csv tabulating Dsc (average single-cell distance from target samples to their representatives) for active-learning vs passive selection at each representative count
- Per-sample inference-difficulty scores Dsc aggregated across samples per selector and representative count
- Equivalence statement of how many additional representatives passive selection needs to match active-learning difficulty (≈4 for the COVID-19 cohort at 28 representatives)

## How to apply

Load the preprocessed cohort bulk and single-cell data (e.g., the 28-sample COVID-19 cohort with doublet removal, cell filtering, library normalization, and log1p transformation), then reduce dimensionality with PCA from Scikit-learn (100 PCs when there are more than 100 samples). Run the passive selector, which picks the sample with the largest Euclidean bulk distance Db to its representative in each of B rounds, and the type-II active-learning selector, which identifies the B clusters with highest in-cluster heterogeneity (Db + λsc*Dsc + λpb*Dpb with λsc = λpb = 1), chooses the representative minimizing in-cluster heterogeneity (Rnew = argmin over j of the sum of min(Db(j,i), Db(Rc,i))), and reassigns membership to the closest representative. For each selector and each representative count, perform in silico inference with the scSemiProfiler deep generative model (activeselect / scinfer). Compute per-sample Dsc as the average Euclidean distance between representative cells and their K = 1 nearest neighbors in the target sample within the PCA-reduced space, aggregate across samples, and tabulate active vs passive Dsc at each count into selection_comparison.csv.

## Related tools

- **scSemiProfiler** (Runs the passive and type-II active-learning representative selectors (activeselect) and performs in silico single-cell inference (scinfer) with the VAE-GAN deep generative model used to compute Dsc.) — https://github.com/mcgilldinglab/scSemiProfiler
- **scikit-learn** (Provides the PCA implementation used to reduce the data dimensionality before distance computations and clustering (K-Means into B distinct clusters) for representative selection.)

## Evaluation signals

- At the 28 representatives used for analysis, the active-learning selector yields lower aggregated Dsc (inference difficulty) than the passive selector on the COVID-19 cohort.
- Passive selection requires roughly 4 additional representatives to reach the same inference difficulty as active learning.
- selection_comparison.csv contains paired Dsc values for both selectors at every evaluated representative count.
- Per-sample Dsc is computed as the average Euclidean distance between representative cells and their K = 1 nearest neighbors in the target sample within the PCA-reduced space.
- PCA uses 100 components when the cohort has more than 100 samples, and active-learning heterogeneity weights default to λsc = λpb = 1.

## Limitations

- The finding is demonstrated on the COVID-19 cohort; generalization of the 28-representative comparison to other cohorts is not established here.
- Adapting the methodology to other modalities (e.g., proteomics) would require algorithm adjustments and comprehensive performance evaluations benchmarked against established standards; extension is future work only.
- The tool requires a single-cell reference to perform single-cell level deconvolution.

## Evidence

- [methods] we employ Principal Component Analysis (PCA)83–85, implemented in the Python package Scikit-learn86, to reduce the dimensionality: "we employ Principal Component Analysis (PCA)83–85, implemented in the Python package Scikit-learn86, to reduce the dimensionality"
- [methods] The dimensionality-reduced data is clustered into B distinct clusters using the K-Means algorithm87: "The dimensionality-reduced data is clustered into B distinct clusters using the K-Means algorithm87"
- [discussion] employs a deep generative learning model, VAE-GAN, to infer the single-cell data of the remainder of the cohort.: "employs a deep generative learning model, VAE-GAN, to infer the single-cell data of the remainder of the cohort."
- [readme] New global mode functions "inspect_data" and "global_stop_checking" have been introduced.: "New global mode functions "inspect_data" and "global_stop_checking" have been introduced."
