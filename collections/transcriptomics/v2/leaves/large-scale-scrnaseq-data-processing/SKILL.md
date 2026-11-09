---
name: large-scale-scrnaseq-data-processing
description: Use when a single-cell dataset is too large for data-dependent sampling
  methods (e.g., millions of cells) and you need a fast preprocessing sketch whose
  representativeness can be compared to geometric sketching via Hausdorff distance.
when_to_use_negative:
- The dataset is small enough that data-dependent sampling methods (geometric sketching,
  spatial random sampling, k-means++, dropClust) scale efficiently — the grid-sampling-alone
  benchmark targets very large datasets where speed matters
- The goal is a quality-optimal sketch regardless of runtime, since prior grid sampling
  alone is a lightweight preprocessing approximation rather than the full spherical
  sketching optimization
- Downstream analysis requires systematically evaluated interplay between the sketching
  distance metric and the preprocessing routines of the downstream algorithm — the
  article states this interplay 'was not systematically addressed in this study'
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Sphetcher
  role: Compiled C++11 tool run with the prior-labels option to generate sketch indicator
    vectors from large expression matrices, and used with its Pearson-correlation
    distance metric to compute Hausdorff distances between sketches and full datasets
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Used to clone the Sphetcher repository prior to compilation
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:a214266103404389f06e7c980ca0f67e23feb25956f7228c8ce1ff5787a65c1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Sphetcher
  - git
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# large-scale-scrnaseq-data-processing

## Summary

A benchmarking procedure for very large single-cell RNA-seq datasets (e.g., Saunders 665,858 cells; MOCA 2,026,641 cells) that applies Sphetcher's prior grid sampling strategy alone as a lightweight preprocessing step and evaluates sketch quality via Hausdorff distance against geometric sketching.

## When to use

Use when a single-cell dataset is too large for data-dependent sampling methods (e.g., millions of cells) and you need a fast preprocessing sketch whose representativeness can be compared to geometric sketching via Hausdorff distance.

## When NOT to use

- The dataset is small enough that data-dependent sampling methods (geometric sketching, spatial random sampling, k-means++, dropClust) scale efficiently — the grid-sampling-alone benchmark targets very large datasets where speed matters
- The goal is a quality-optimal sketch regardless of runtime, since prior grid sampling alone is a lightweight preprocessing approximation rather than the full spherical sketching optimization
- Downstream analysis requires systematically evaluated interplay between the sketching distance metric and the preprocessing routines of the downstream algorithm — the article states this interplay 'was not systematically addressed in this study'

## Inputs

- Expression matrix in CSV format with rows as cells and columns as features/genes/PCs (e.g., Saunders adult mouse brain, 665,858 cells; MOCA, 2,026,641 cells)
- Class labels CSV (column vector of integers between 1 and K encoding prior categories such as cell type or collection time point)
- Sketch size (number of samples to obtain, e.g., 6,646 / 26,573 / 66,649 for Saunders; 20,296 / 81,093 for MOCA)
- l_min (minimum number of representatives to sample from each class)

## Outputs

- Sketch indicator vector of n samples (CSV) where 1 indicates the cell is in the sketch and 0 otherwise, containing exactly sketch_size 1's
- Hausdorff distances between each prior-grid sketch and the full dataset, computed with Pearson correlation as distance metric
- Comparison of prior-grid versus geometric-sketching Hausdorff distances (Figure S3 pattern)

## How to apply

Clone and compile Sphetcher from github.com/canzarlab/Sphetcher with a C++11-supporting compiler (cd src; make). Load the Saunders adult mouse brain expression matrix (665,858 cells) and/or the MOCA expression matrix (2,026,641 cells) as CSV with rows as cells and columns as features/PCs. Run Sphetcher with the prior-labels option ('sphetcher expression_matrix.csv sketch_size class_labels.csv l_min sketch_indicator_output.csv') on Saunders at 1%, 4%, and 10% sketch sizes (6,646 / 26,573 / 66,649 cells) and on MOCA at 1% and 4% (20,296 / 81,093 cells), supplying class labels and l_min so a minimum number of representatives is preserved per class. Compute the Hausdorff distance between each prior-grid sketch and the full dataset using Sphetcher's distance metric (Pearson correlation in most experiments). The rationale is that naive grid sampling alone is orders of magnitude faster than geometric sketching while remaining competitive on very large datasets, especially at small sketch sizes; Sphetcher combined with prior grid sampling computed a sketch for a two-million-cell mouse embryonic dataset in 16 minutes.

## Related tools

- **Sphetcher** (Compiled C++11 tool run with the prior-labels option to generate sketch indicator vectors from large expression matrices, and used with its Pearson-correlation distance metric to compute Hausdorff distances between sketches and full datasets) — https://github.com/canzarlab/Sphetcher
- **git** (Used to clone the Sphetcher repository prior to compilation) — https://github.com/canzarlab/Sphetcher

## Examples

```
sphetcher saunders_expression_matrix.csv 6646 saunders_class_labels.csv 3 sketch_indicator_output.csv
```

## Evaluation signals

- The sketch indicator output CSV contains exactly sketch_size 1's among n entries (e.g., 6,646 ones for Saunders 1%)
- Hausdorff distances of prior-grid sketches are competitive with (not worse than) geometric sketching across the evaluated sketch sizes (1%, 4%, 10% for Saunders; 1%, 4% for MOCA), matching the Figure S3 pattern
- Runtime of grid sampling alone is orders of magnitude faster than geometric sketching (e.g., a two-million-cell MOCA-scale sketch in ~16 minutes)
- The class_labels.csv is a column vector with integers between 1 and K, and each class receives at least l_min representatives in the sketch
- Expression matrix CSV has rows as cells and columns as features/PCs, matching the README's input format

## Limitations

- The interplay of sketching with respect to a given distance metric and subsequent algorithmic analysis was not systematically addressed in the study
- Benefits of leveraging prior categorical knowledge (e.g., cell types) in sketch selection are only demonstrated at proof-of-principle level; additional experiments are required
- No software version numbers are stated for Sphetcher or the downstream tools used; in most experiments Sphetcher used Pearson correlation as distance metric, which may not fit all downstream algorithms

## Evidence

- [readme] Clone/compile step: "A compiler that supports C++11 is needed to build sphetcher. You can download and compile the latest code from github as follows"
- [readme] Prior-labels usage and example invocation: "sphetcher expression_matrix.csv sketch_size class_labels.csv l_min sketch_indicator_output.csv"
- [methods] Saunders dataset scale: "Adult mouse brain cells from [Saunders et al. (2018)](https://www.cell.com/cell/fulltext/S0092-8674(18)30955-3)  (665,858 cells)"
- [methods] MOCA dataset scale: "Mouse organogenesis cell atlas (MOCA) from [Cao et al. (2019)](https://www.nature.com/articles/s41586-019-0969-x) (2,026,641 cells)"
- [discussion] Distance metric for Hausdorff evaluation: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric"
- [intro] Scalability of grid sampling combination: "Sphetcher requires only 16 minutes to compute a sketch for a mouse embryonic dataset comprising two million cells."
