---
name: prior-grid-sampling-with-class-labels
description: Use when a single-cell expression matrix is too large (hundreds of thousands
  to millions of cells) for geometric sketching, prior class labels are available,
  and a fast, competitive subsample measured by Hausdorff distance is needed.
when_to_use_negative:
- The dataset is small enough that data-dependent methods like geometric sketching,
  spatial random sampling, k-means++, or dropClust scale efficiently — the grid sampling
  strategy alone is aimed at very large datasets where those methods are too slow.
- No prior categorical knowledge (cell labels, collection time points) exists to supply
  as class_labels.csv with a chosen l_min.
- The task requires systematic evaluation of how the distance-metric-based sketch
  interacts with internal preprocessing routines of downstream algorithms — the article
  explicitly did not address this interplay.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Sphetcher
  role: Provides the prior grid sampling strategy executed with the class-labels/l_min
    CLI option to generate sketch indicator vectors for large single-cell datasets
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Used to clone the Sphetcher source repository before compilation
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:78eab98cb9f28e2a13dde7863e6bd05cd6d28cf0adf5242cb7928c5c78be30b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Sphetcher
  - git
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# prior-grid-sampling-with-class-labels

## Summary

A lightweight preprocessing strategy for massive single-cell RNA-seq datasets that uses Sphetcher's prior grid sampling with class labels (e.g., cell type) and a minimum representatives-per-class threshold to subsample very large datasets (Saunders 665,858 cells; MOCA 2,026,641 cells) quickly before downstream sketch quality assessment.

## When to use

Use when a single-cell expression matrix is too large (hundreds of thousands to millions of cells) for geometric sketching, prior class labels are available, and a fast, competitive subsample measured by Hausdorff distance is needed.

## When NOT to use

- The dataset is small enough that data-dependent methods like geometric sketching, spatial random sampling, k-means++, or dropClust scale efficiently — the grid sampling strategy alone is aimed at very large datasets where those methods are too slow.
- No prior categorical knowledge (cell labels, collection time points) exists to supply as class_labels.csv with a chosen l_min.
- The task requires systematic evaluation of how the distance-metric-based sketch interacts with internal preprocessing routines of downstream algorithms — the article explicitly did not address this interplay.

## Inputs

- expression_matrix.csv (CSV, rows are cells, columns are features/genes/PCs)
- sketch_size (integer number of samples to obtain)
- class_labels.csv (column vector of integer class labels 1..K, e.g., cell label or collection time point)
- l_min (minimum number of representatives per class)

## Outputs

- sketch_indicator_output.csv (indicator vector of n samples, 1 = in sketch, with sketch_size 1's)
- Hausdorff distance values between the prior-grid sketch and the full dataset for comparison against geometric sketching (Figure S3 pattern)

## How to apply

Clone the Sphetcher repository (github.com/canzarlab/Sphetcher) and compile the C++11 source with `cd src; make`. Load the expression matrix as a CSV with rows as cells and columns as features/genes/PCs (e.g., Saunders adult mouse brain, MOCA, or zeiselCNS matrices), plus a class_labels.csv column vector of integers 1..K encoding prior categories such as cell type or collection time point. Run Sphetcher with the prior-labels option at evaluated sketch sizes — Saunders at 1%/4%/10% (6,646 / 26,573 / 66,649 cells), MOCA at 1%/4% (20,296 / 81,093 cells) — supplying l_min as the minimum number of representatives sampled from each class. The output is a sketch indicator vector of n samples with sketch_size 1's. Compute the Hausdorff distance between the prior-grid sketch and the full dataset using Sphetcher's distance metric (Pearson correlation in most experiments) and compare against geometric sketching, as done to reproduce Figure S3; the sketch is accepted if its Hausdorff distances are competitive with (not worse than) geometric sketching across sketch sizes.

## Related tools

- **Sphetcher** (Provides the prior grid sampling strategy executed with the class-labels/l_min CLI option to generate sketch indicator vectors for large single-cell datasets) — https://github.com/canzarlab/Sphetcher
- **git** (Used to clone the Sphetcher source repository before compilation) — https://github.com/canzarlab/Sphetcher

## Examples

```
sphetcher zeisel_pca.csv 1000 zeisel_pca_labels.csv 3 sketch_indicator_output.csv
```

## Evaluation signals

- The output sketch_indicator_output.csv is an indicator vector of length n containing exactly sketch_size 1's.
- Hausdorff distances of the prior-grid sketch are competitive with (not worse than) geometric sketching across the evaluated sketch sizes, matching the Figure S3 pattern.
- Each class in class_labels.csv retains at least l_min representatives in the sketch.
- Runtime is orders of magnitude faster than geometric sketching on the large zeiselCNS/Saunders datasets, especially at small sketch sizes (e.g., 1%).
- Compilation succeeds with a C++11-supporting compiler, confirming the correct build before running.

## Limitations

- The interplay of sketching with respect to a given distance metric and subsequent algorithmic analysis was not systematically addressed in the study.
- Additional experiments are required to fully address the benefits of leveraging prior (partial) knowledge on, e.g., cell types in the selection of representative cells.
- No software version numbers are stated for Sphetcher or downstream tools; in most experiments Sphetcher used Pearson correlation as the distance metric, and results may differ with other metrics.

## Evidence

- [readme] sphetcher expression_matrix.csv sketch_size class_labels.csv l_min sketch_indicator_output.csv: "sphetcher expression_matrix.csv sketch_size class_labels.csv l_min sketch_indicator_output.csv"
- [readme] Additionally you can provide the prior information (e.g, cell label, collection time point) in case you want to preserve certain number of samples from each category.: "Additionally you can provide the prior information (e.g, cell label, collection time point) in case you want to preserve certain number of samples from each category."
- [methods] A compiler that supports C++11 is needed to build sphetcher.: "A compiler that supports C++11 is needed to build sphetcher."
- [intro] Sphetcher requires only 16 minutes to compute a sketch for a mouse embryonic dataset comprising two million cells.: "Sphetcher requires only 16 minutes to compute a sketch for a mouse embryonic dataset comprising two million cells."
- [discussion] In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric and was combined with a specific algorithm for downstream analysis.: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric and was combined with a specific algorithm for downstream analysis."
