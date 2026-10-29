---
name: random-trial-aggregation-across-datasets
description: Use when comparing two sketching/sampling methods across multiple single-cell
  datasets and sketch sizes, where robust Hausdorff distance to the full dataset quantifies
  representativeness and stochastic trials must be aggregated.
when_to_use_negative:
- You only need a single sketch for downstream analysis (e.g., clustering with Louvain
  or trajectory inference with Monocle 2) rather than a method-vs-method accuracy
  benchmark
- You want to evaluate fairness-constrained sketching with prior class labels and
  l_min representatives per class — that is a separate optimization mode of Sphetcher,
  not this comparison protocol
- Your dataset is small enough that sketching is unnecessary; the paper motivates
  this for datasets whose size exceeds the capability of current computational analysis
  methods
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_0091
tools:
- name: Sphetcher
  role: Spherical sketching method under evaluation; run per dataset, sketch size,
    and trial with Pearson correlation distance to produce sketch indicator vectors
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Clone the canzarlab/Sphetcher repository to build the sphetcher binary used
    in every trial
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:2f19eb0bf1038cf3ee3bb0732ddf2c46ef536f6dd06a0ee424331935ee345415
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Sphetcher
  - git
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# random-trial-aggregation-across-datasets

## Summary

Benchmarking protocol that repeats Sphetcher spherical sketching and Hie et al. geometric sketching over 10 random trials per dataset and sketch size (1%, 4%, 10%), computes robust Hausdorff distances of each sketch to the full data, and aggregates results into comparison tables and plots across six single-cell datasets.

## When to use

Use when comparing two sketching/sampling methods across multiple single-cell datasets and sketch sizes, where robust Hausdorff distance to the full dataset quantifies representativeness and stochastic trials must be aggregated.

## When NOT to use

- You only need a single sketch for downstream analysis (e.g., clustering with Louvain or trajectory inference with Monocle 2) rather than a method-vs-method accuracy benchmark
- You want to evaluate fairness-constrained sketching with prior class labels and l_min representatives per class — that is a separate optimization mode of Sphetcher, not this comparison protocol
- Your dataset is small enough that sketching is unnecessary; the paper motivates this for datasets whose size exceeds the capability of current computational analysis methods

## Inputs

- expression_matrix.csv (CSV, rows are cells, columns are genes/transcripts/principal components) for six datasets: muraro, klein, zeisel, chen, zeiselCNS, saunders
- sketch_size values corresponding to 1%, 4%, and 10% of cells
- Sketch indicator output vectors from Sphetcher and sketches from Hie et al. geometric sketching, 10 trials each

## Outputs

- hausdorff_results.csv (robust Hausdorff distances per dataset, sketch size, method, and trial)
- Aggregated comparison plots reproducing Figures 2/S1

## How to apply

Prepare each expression matrix (rows are cells, columns are features/PCs) and run Sphetcher via 'sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv', using Pearson correlation as the distance metric as in the paper's experiments. Run Hie et al. geometric sketching on the same matrices at the same sketch sizes as the baseline. Repeat both procedures over 10 random trials per dataset and sketch size (1%, 4%, 10%). For every trial compute the robust Hausdorff distance between the sketch and the full dataset, then aggregate distances per dataset, sketch size, and method to reproduce the Figures 2/S1 comparison and write hausdorff_results.csv. Rationale: aggregating over 10 trials controls sketching randomness so consistent per-dataset differences reflect method accuracy rather than sampling noise.

## Related tools

- **Sphetcher** (Spherical sketching method under evaluation; run per dataset, sketch size, and trial with Pearson correlation distance to produce sketch indicator vectors) — https://github.com/canzarlab/Sphetcher
- **git** (Clone the canzarlab/Sphetcher repository to build the sphetcher binary used in every trial) — https://github.com/canzarlab/Sphetcher

## Examples

```
cd src; make; sphetcher zeisel_pca.csv 1000 sketch_indicator_output.csv
```

## Evaluation signals

- Each sketch_indicator_output.csv is a binary indicator vector of n samples containing exactly sketch_size 1's
- hausdorff_results.csv contains one robust Hausdorff distance per (dataset, sketch size, method, trial) combination — 10 trials for all six datasets at 1%, 4%, and 10%
- Per-dataset aggregated distances show Sphetcher spherical sketches with smaller robust Hausdorff distances than geometric sketches across all sketch sizes
- The 1% spherical sketch should outperform the 10% geometric sketch, matching the paper's reported finding
- Comparison plots are consistent with the paper's Figures 2/S1 layout of distance versus sketch size per dataset

## Limitations

- The paper used Pearson correlation as the distance metric in most experiments, but the interplay of sketching with respect to a given distance metric and subsequent algorithmic analysis 'was not systematically addressed in this study'
- No software version numbers are stated for Sphetcher or the downstream tools, so exact reproducibility of trial results may vary
- Pre-computed sketches on the Sphetcher site cover only Saunders (1%, 4%, 10%), MOCA (1%, 4%), and Zeisel CNS (1%, 4%, 10%); muraro, klein, and chen sketches must be generated by the practitioner

## Evidence

- [results] Sphetcher's sphere-based sampling consistently produces sketches with smaller Hausdorff distances to the full dataset than geometric sketching: [evidence span withheld — anchor status: not_found]
- [discussion] In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric"
- [methods] sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv: "sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv"
- [readme] sketch_indicator_output.csv : an indicator vector of n samples where 1 indicates the sample is in the sketch: "`sketch_indicator_output.csv` : an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector)."
- [intro] We provide theoretical guarantees for the spherical sketch computed by Sphetcher and demonstrate through experiments on six single-cell datasets: "We provide theoretical guarantees for the spherical sketch computed by Sphetcher and demonstrate through experiments on six single-cell datasets"
