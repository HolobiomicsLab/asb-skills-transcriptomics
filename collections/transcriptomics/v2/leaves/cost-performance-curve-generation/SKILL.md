---
name: cost-performance-curve-generation
description: 'Use when evaluating a semi-profiling or representative-selection pipeline:
  you have real-profiled single-cell data for a cohort, inferred or copied data for
  non-representatives at multiple representative counts, and you need a normalized
  error vs. cost curve.'
when_to_use_negative:
- Input is only bulk data with no real-profiled single-cell ground truth for any samples,
  since UB/LB normalization and error validation require paired real-profiled reference
  data
- The goal is deconvolution from bulk data with an external single-cell reference
  (the single-cell-level deconvolution scenario), not cohort-level semi-profiling
  cost-performance evaluation
- Cell-type proportion comparison or UMAP similarity is the target metric instead
  of single-cell-level normalized error curves
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: scSemiProfiler
  role: Runs the semi-profiling pipeline (K-Means initial setup, scprocess, scinfer
    VAE-GAN inference) to generate inferred single-cell data at each representative
    count
  repo: https://github.com/mcgilldinglab/scSemiProfiler
- name: Scikit-learn
  role: Provides the PCA implementation used to reduce real-profiled and inferred
    single-cell datasets to 100 principal components for the distance metric
- name: SCANPY
  role: Used in the single-cell processing step (scprocess) supporting the analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-50150-1
    title: 'scSemiProfiler: Advancing large-scale single-cell studies through semi-profiling
      with deep generative models and active learning'
  source_content_hash: sha256:246b8753f2bcc8bb6d8a219e57f29e558dbea1cb4596f92db62a471c59a83f84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scSemiProfiler
  tools:
  - SCANPY
  - Scikit-learn
  - scSemiProfiler
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cost-performance-curve-generation

## Summary

Generates normalized-error-vs-number-of-representatives curves that quantify how well scSemiProfiler's inferred single-cell data approximates real-profiled data as sequencing cost scales, benchmarked against a selection-only baseline that copies representative cells.

## When to use

Use when evaluating a semi-profiling or representative-selection pipeline: you have real-profiled single-cell data for a cohort, inferred or copied data for non-representatives at multiple representative counts, and you need a normalized error vs. cost curve.

## When NOT to use

- Input is only bulk data with no real-profiled single-cell ground truth for any samples, since UB/LB normalization and error validation require paired real-profiled reference data
- The goal is deconvolution from bulk data with an external single-cell reference (the single-cell-level deconvolution scenario), not cohort-level semi-profiling cost-performance evaluation
- Cell-type proportion comparison or UMAP similarity is the target metric instead of single-cell-level normalized error curves

## Inputs

- Bulk sequencing data for the full cohort (e.g., COVID-19 cohort with 124 samples)
- Real single-cell profiled data for representative samples at each representative count (4 to 28)
- Semi-profiled single-cell data generated via the VAE-GAN inference (scinfer) for non-representative samples
- Selection-only baseline single-cell data (representatives' cells copied to target samples)

## Outputs

- error_curves.csv containing normalized semi-profiling error (EN) values for scSemiProfiler and the selection-only baseline across representative counts 4 to 28
- Cost-performance curve comparing normalized error vs. number of sequenced representatives (Fig. 2h analogue)

## How to apply

For each representative count n (e.g., 4 to 28 on the COVID-19 cohort with 124 samples), run the scSemiProfiler pipeline — initial setup with K-Means clustering on 100 PCs, single-cell processing via scprocess, and VAE-GAN-based inference via scinfer — to generate semi-profiled data for non-representative samples. For the selection-only baseline, assign each target sample its representative's single-cell data without the deep generative model transformation. Reduce both real-profiled and inferred datasets to 100 principal components using Scikit-learn PCA and compute the single-cell difference as the average Euclidean distance of each cell to its K=1 nearest neighbor in the paired dataset. Normalize each error as EN = (mean_difference - LB) / (UB - LB), where UB is the average single-cell difference of randomly paired samples from the PCA-reduced real-profiled data and LB is the difference between two random halves of the comprehensive PCA matrix. Save the normalized errors for both methods across all representative counts into error_curves.csv for plotting (Fig. 2h-style).

## Related tools

- **scSemiProfiler** (Runs the semi-profiling pipeline (K-Means initial setup, scprocess, scinfer VAE-GAN inference) to generate inferred single-cell data at each representative count) — https://github.com/mcgilldinglab/scSemiProfiler
- **Scikit-learn** (Provides the PCA implementation used to reduce real-profiled and inferred single-cell datasets to 100 principal components for the distance metric)
- **SCANPY** (Used in the single-cell processing step (scprocess) supporting the analysis)

## Examples

```
from sklearn.decomposition import PCA; X100 = PCA(n_components=100).fit_transform(adata.X)
```

## Evaluation signals

- The semi-profiling normalized error at 28 representatives is strictly below the selection-only normalized error at 28 representatives
- Normalized error decreases monotonically (or near-monotonically) as the number of representatives increases from 4 to 28
- The computed output matches the Fig. 2h Source Data file of the paper
- LB (difference between two random halves of the comprehensive PCA matrix) is smaller than any method's mean_difference, and UB (randomly paired samples) is larger, keeping EN within [0, 1]
- error_curves.csv contains both methods' error values for every representative count in the scanned range

## Limitations

- Requires real single-cell ground-truth data for the entire cohort to compute UB, LB, and validate the curve, which is only feasible in retrospective evaluations
- The error metric depends on PCA dimensionality (100 components) and K=1 nearest-neighbor distance choices; different settings are not directly comparable to the published curve
- The tool requires a single-cell reference to perform single-cell-level decomposition, and adaptation to modalities beyond scRNA-seq (e.g., proteomics) is not yet implemented

## Evidence

- [methods] PCA to 100 components: "we employ Principal Component Analysis (PCA)83–85, implemented in the Python package Scikit-learn86, to reduce the dimensionality"
- [readme] semi-profiling scenario with representative selection: [evidence span withheld — anchor status: external_doc]
- [discussion] pipeline structure: active learning + VAE-GAN inference: "The pipeline ingests bulk data, leverages an active learning module to judiciously select representatives for authentic single-cell sequencing"
- [readme] cohort scale for the COVID-19 experiment: [evidence span withheld — anchor status: external_doc]
