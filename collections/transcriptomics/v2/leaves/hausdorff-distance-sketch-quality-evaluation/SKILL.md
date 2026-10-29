---
name: hausdorff-distance-sketch-quality-evaluation
description: Use when you have a sketch (subset of cells) and the full single-cell
  expression/PC matrix and need to quantify how well the sketch covers the original
  transcriptomic space, or to compare competing sketching methods.
when_to_use_negative:
- When the goal is to evaluate sketches against downstream task performance (clustering,
  trajectory inference) rather than geometric coverage of the transcriptomic space
- When no full dataset is available to serve as the reference for the Hausdorff computation
- When the interplay between the sketching distance metric and a downstream algorithm's
  internal preprocessing is the object of study — the paper explicitly did not systematically
  address this
edam_operation: http://edamontology.org/operation_3359
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Sphetcher
  role: Produces spherical sketches (with optional prior grid sampling) to be evaluated;
    its Pearson correlation distance metric is used for the Hausdorff computation
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Used to clone the Sphetcher repository before compiling the evaluation pipeline
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:5cd7eea7e743d32eb238d8482d21a912d2ee7841ef9680a6aa0d58d29928a697
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Sphetcher
  - git
  edam_operation: http://edamontology.org/operation_3359
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# hausdorff-distance-sketch-quality-evaluation

## Summary

Evaluates the quality of single-cell transcriptomic sketches (e.g., Sphetcher spherical sketches vs. Hie et al. geometric sketches vs. grid sampling) by computing robust Hausdorff distances between each sketch and the full dataset, aggregated over sketch sizes (1%–10%) and random trials.

## When to use

Use when you have a sketch (subset of cells) and the full single-cell expression/PC matrix and need to quantify how well the sketch covers the original transcriptomic space, or to compare competing sketching methods.

## When NOT to use

- When the goal is to evaluate sketches against downstream task performance (clustering, trajectory inference) rather than geometric coverage of the transcriptomic space
- When no full dataset is available to serve as the reference for the Hausdorff computation
- When the interplay between the sketching distance metric and a downstream algorithm's internal preprocessing is the object of study — the paper explicitly did not systematically address this

## Inputs

- Expression/PCA matrix in CSV format with rows as cells and columns as features (genes, transcripts, PCs), e.g. zeisel_pca.csv
- Sketch indicator vectors (n samples, 1 = in sketch, 0 otherwise) from each sketching method and trial
- Sketch sizes to evaluate (1%, 4%, 10% of cells)
- Optional: class labels CSV and l_min for prior/grid sampling sketches (e.g., Saunders at 6,646/26,573/66,649 cells; MOCA at 20,296/81,093 cells)

## Outputs

- Robust Hausdorff distance values per dataset, method, sketch size, and random trial
- Aggregated results table, e.g. hausdorff_results.csv
- Comparison plots reproducing Figures 2/S1 (spherical vs. geometric) and Figure S3 (prior grid sampling vs. geometric)

## How to apply

Prepare each dataset as a CSV expression matrix with rows as cells and columns as features/PCs. Sketch the data with the methods under comparison — Sphetcher (spherical thresholding), Hie et al. geometric sketching, and optionally Sphetcher's prior grid sampling for very large datasets like Saunders (665,858 cells) and MOCA (2,026,641 cells) — at sketch sizes of 1%, 4%, and 10%, repeating over 10 random trials per dataset and size. Use Pearson correlation as the distance metric, as in most of the paper's experiments, when computing the robust Hausdorff distance between each sketch and the full dataset. Aggregate distances per dataset, sketch size, and method to reproduce the comparison of Figures 2/S1 (six datasets: muraro, klein, zeisel, chen, zeiselCNS, saunders) and Figure S3 (grid-sampling-alone on zeiselCNS and saunders), writing results to a table such as hausdorff_results.csv. Decision rule: a method is superior if its Hausdorff distances are consistently smaller across datasets and sizes; note the paper's finding that the 1% spherical sketch even outperformed the 10% geometric sketch.

## Related tools

- **Sphetcher** (Produces spherical sketches (with optional prior grid sampling) to be evaluated; its Pearson correlation distance metric is used for the Hausdorff computation) — https://github.com/canzarlab/Sphetcher
- **git** (Used to clone the Sphetcher repository before compiling the evaluation pipeline) — https://github.com/canzarlab/Sphetcher

## Examples

```
git clone https://github.com/canzarlab/Sphetcher && cd src && make && ./sphetcher expression_matrix.csv 6646 sketch_indicator_output.csv
```

## Evaluation signals

- Each sketch indicator output vector has exactly sketch_size entries equal to 1 among n samples
- Sphetcher sketches should show smaller robust Hausdorff distances than geometric sketching on all six datasets (muraro, klein, zeisel, chen, zeiselCNS, saunders) at all sizes 1%–10% across 10 random trials
- The 1% spherical sketch should achieve a smaller Hausdorff distance than the 10% geometric sketch (per the paper's finding)
- Prior-grid sampling distances on zeiselCNS and saunders should be competitive with (not worse than) geometric sketching, matching the Figure S3 pattern
- Aggregated results should be reproducible across the 10 random trials per dataset/sketch size combination

## Limitations

- The interplay of sketching with respect to a given distance metric and subsequent algorithmic analysis was not systematically addressed in the study
- No software version numbers are stated for Sphetcher or downstream tools, so exact reproducibility may vary
- Results depend on the chosen distance metric (Pearson correlation in most experiments); sketches optimal under one metric may not transfer to others
- Building Sphetcher requires a compiler supporting C++11

## Evidence

- [abstract] We show that the spherical sketch computed by Sphetcher constitutes a more accurate representation of the original transcriptomic landscape: "We show that the spherical sketch computed by Sphetcher constitutes a more accurate representation of the original transcriptomic landscape"
- [discussion] In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric"
- [methods] A compiler that supports C++11 is needed to build sphetcher.: "A compiler that supports C++11 is needed to build sphetcher."
- [readme] sketch_indicator_output.csv : an indicator vector of n samples where 1 indicates the sample is in the sketch, 0 otherwise: "an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector)."
- [readme] First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs).: "First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs)."
