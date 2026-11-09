---
name: distance-metric-benchmarking
description: Use when sketching a scRNA-seq expression matrix (cells x features/PCs)
  with Sphetcher and you need to decide which distance metric for the sketching objective
  preserves cluster structure for downstream clustering.
when_to_use_negative:
- You only need a single production sketch — the paper notes that in most experiments
  Sphetcher used Pearson correlation as its distance metric, so a full four-metric
  benchmark is unnecessary unless metric choice is the question
- You want to systematically evaluate the interplay between sketching metric and internal
  preprocessing routines of downstream algorithms — the article states this interplay
  'was not systematically addressed in this study'
- The downstream algorithm is something other than Louvain or Monocle 2 — Sphetcher's
  underlying model might be less compatible with assumptions made by other algorithms
  and this was not tested
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: Sphetcher
  role: Computes the spherical sketch under each candidate distance metric, emitting
    a sketch indicator vector
  repo: https://github.com/canzarlab/Sphetcher
- name: Louvain
  role: Downstream community-detection clustering applied to each extracted sketch
    so ARI can be compared across metrics
- name: git
  role: Clones the Sphetcher repository for compilation
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:d9e3fc35b50092af8ea708fd7a31aad433bf0bd2b5a52721a422d21ce8022492
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Louvain
  - Sphetcher
  - git
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# distance-metric-benchmarking

## Summary

Benchmark how the distance metric used inside Sphetcher's spherical thresholding objective (Euclidean, Manhattan, cosine, Pearson) affects the accuracy of downstream scRNA-seq clustering, measured by Adjusted Rand Index against reference cell labels. The paper found Euclidean-distance sketches yielded the most accurate clusterings on dataset chen.

## When to use

Use when sketching a scRNA-seq expression matrix (cells x features/PCs) with Sphetcher and you need to decide which distance metric for the sketching objective best preserves cluster structure for downstream clustering.

## When NOT to use

- You only need a single production sketch — the paper notes that in most experiments Sphetcher used Pearson correlation as its distance metric, so a full four-metric benchmark is unnecessary unless metric choice is the question
- You want to systematically evaluate the interplay between sketching metric and internal preprocessing routines of downstream algorithms — the article states this interplay 'was not systematically addressed in this study'
- The downstream algorithm is something other than Louvain or Monocle 2 — Sphetcher's underlying model might be less compatible with assumptions made by other algorithms and this was not tested

## Inputs

- Expression matrix CSV per dataset (rows = cells, columns = features/PCs), e.g. the chen and muraro matrices
- Reference cell-type labels for each dataset (for ARI computation)
- Sketch size (e.g., the Figure 4 sketch size)
- Distance metric choice per run: Euclidean, Manhattan, cosine, or Pearson

## Outputs

- One sketch_indicator_output.csv per dataset/metric combination (indicator vector with sketch_size 1's)
- Extracted sketched-cell matrices (cells with indicator = 1)
- Louvain clustering assignments for each sketch
- ari_scores.csv: tabulated ARI scores across datasets and all four metrics

## How to apply

Clone github.com/canzarlab/Sphetcher and compile with a C++11-compatible compiler (cd src; make). For each distance metric (Euclidean, Manhattan, cosine, Pearson), run sphetcher on the chen (and optionally muraro) expression matrix — rows are cells, columns are features/PCs — at a fixed sketch size (e.g., the Figure 4 size), writing one sketch_indicator_output.csv per dataset/metric combination. Extract the sketched cells (indicator = 1) from each matrix using the indicator vectors. Cluster each extracted sketch with Louvain community detection as the downstream scRNA-seq algorithm, then compute the Adjusted Rand Index of each clustering against the reference cell labels. Tabulate ARI scores across all dataset/metric combinations and verify which metric yields the highest ARI; the paper's finding is that Euclidean distance in the objective yields the most accurate clustering of chen, while Manhattan distance and geometric sketching yield substantially less accurate clusterings.

## Related tools

- **Sphetcher** (Computes the spherical sketch under each candidate distance metric, emitting a sketch indicator vector) — https://github.com/canzarlab/Sphetcher
- **Louvain** (Downstream community-detection clustering applied to each extracted sketch so ARI can be compared across metrics)
- **git** (Clones the Sphetcher repository for compilation) — https://github.com/canzarlab/Sphetcher

## Examples

```
git clone https://github.com/canzarlab/Sphetcher; cd Sphetcher/src; make; sphetcher chen_pca.csv 1000 sketch_indicator_output.csv
```

## Evaluation signals

- Each sketch_indicator_output.csv contains exactly sketch_size entries equal to 1 (indicator vector of n samples)
- Eight indicator files exist (two datasets x four metrics) and each sketched submatrix has exactly sketch_size rows
- ari_scores.csv contains one ARI value per dataset/metric combination with Euclidean on chen having the highest ARI
- Manhattan-distance and geometric-sketching ARI values are substantially lower than the Euclidean value on chen, reproducing the paper's finding
- All runs use the identical sketch size and downstream Louvain parameters so metric is the only varying factor

## Limitations

- The interplay of sketching with respect to a given distance metric and subsequent algorithmic analysis was not systematically addressed in the study
- Sphetcher's underlying model might be less compatible with assumptions made by algorithms other than Louvain and Monocle 2; this was not tested
- No software version numbers are stated for Sphetcher or the downstream tools used
- A C++11-compatible compiler is required to build sphetcher

## Evidence

- [other] Euclidean metric yields most accurate clustering: [evidence span withheld — anchor status: not_found]
- [methods] C++11 build requirement: "A compiler that supports C++11 is needed to build sphetcher. You can download and compile the latest code from github as follows: git clone  https://github.com/canzarlab/Sphetcher"
- [readme] Input matrix format: "First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs)."
- [readme] Usage and output format: "`sketch_indicator_output.csv` : an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector)."
- [discussion] Pearson as default metric in experiments: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric and was combined with a specific algorithm for downstream analysis."
- [discussion] Louvain as widely used downstream clustering: "Louvain community detection and Monocle 2 are widely used for scRNA-seq clustering and the inference of single-cell trajectories"
