---
name: sphere-cover-thresholding-algorithm-execution
description: Use when reproducing or benchmarking the scalability of Sphetcher-H versus
  geometric sketching on a million-scale single-cell expression matrix (e.g., 2,026,641-cell
  MOCA), recording per-stage and total runtimes at a fixed sketch size such as 20296
  (1% of cells).
when_to_use_negative:
- The goal is sketch quality (rare cell type recovery, trajectory support) rather
  than scalability/runtime comparison — use the accuracy evaluation workflow instead
- The dataset is small enough that data-dependent sampling runtime is not a bottleneck;
  the 16-vs-30-minute comparison targets datasets on the order of two million cells
- Prior categorical labels (cell type, collection time point) must drive sketch composition
  — that requires the fairness variant of Sphetcher, not the plain runtime benchmark
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Sphetcher
  role: Hybrid grid sampling + spherical thresholding (set cover) sketching tool whose
    stages are timed and compared against geometric sketching
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Used to clone the Sphetcher repository before compilation with a C++11 compiler
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:2e4146839f3bb5b6192ee49f7f0275daefbf796f85eb5cb6a72d162e357cdf96
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# sphere-cover-thresholding-algorithm-execution

## Summary

Execute Sphetcher's hybrid strategy (Sphetcher-H: grid sampling followed by spherical-thresholding set cover) on very large scRNA-seq expression matrices and benchmark its runtime against geometric box-based sketching, e.g. on the 2,026,641-cell MOCA dataset where Sphetcher-H takes ~16 min versus ~30 min for geometric sketching.

## When to use

Use when reproducing or benchmarking the scalability of Sphetcher-H versus geometric sketching on a million-scale single-cell expression matrix (e.g., 2,026,641-cell MOCA), recording per-stage and total runtimes at a fixed sketch size such as 20296 (1% of cells).

## When NOT to use

- The goal is sketch quality (rare cell type recovery, trajectory support) rather than scalability/runtime comparison — use the accuracy evaluation workflow instead
- The dataset is small enough that data-dependent sampling runtime is not a bottleneck; the 16-vs-30-minute comparison targets datasets on the order of two million cells
- Prior categorical labels (cell type, collection time point) must drive sketch composition — that requires the fairness variant of Sphetcher, not the plain runtime benchmark

## Inputs

- CSV expression matrix (rows = cells, columns = features/genes/PCs), e.g. the MOCA matrix from Cao et al. (2019) with 2,026,641 cells
- Integer sketch_size (e.g. 20296 for 1% of the MOCA dataset)
- Sphetcher source code from github.com/canzarlab/Sphetcher compiled with a C++11-supporting compiler
- Geometric sketching baseline executable run on the same matrix and sketch size

## Outputs

- sketch_indicator_output.csv: indicator vector of n samples with 1's marking the sketch_size selected cells
- runtime_comparison.csv: per-stage (grid sampling, distances, set cover) and total timings for Sphetcher-H and geometric sketching

## How to apply

Clone github.com/canzarlab/Sphetcher and compile in src with a C++11-supporting compiler (make). Prepare the dataset — for the Cao et al. (2019) Mouse organogenesis cell atlas (MOCA, 2,026,641 cells) — as a CSV expression matrix with rows as cells and columns as features/PCs. Run Sphetcher-H, which first applies a grid sampling strategy then the spherical thresholding (set cover) step, at sketch size 20296 (1% of cells), timing the grid-sampling, distance-computation, and set-cover stages separately. Run the geometric box-based sketching baseline on the same matrix and sketch size, recording its total runtime. Aggregate per-stage and total timings into runtime_comparison.csv; the reference result is ~16 minutes for Sphetcher-H (grid sampling 10 s, distances 600 s, set cover 400 s) versus ~30 minutes for geometric sketching.

## Related tools

- **Sphetcher** (Hybrid grid sampling + spherical thresholding (set cover) sketching tool whose stages are timed and compared against geometric sketching) — https://github.com/canzarlab/Sphetcher
- **git** (Used to clone the Sphetcher repository before compilation with a C++11 compiler)

## Examples

```
sphetcher moca_pca.csv 20296 sketch_indicator_output.csv
```

## Evaluation signals

- Sphetcher-H total runtime on the 2M-cell MOCA dataset is on the order of the reported 16 minutes (grid sampling ~10 s, distance computation ~600 s, set cover ~400 s)
- runtime_comparison.csv contains timing entries for both Sphetcher-H and geometric sketching and all timed stages
- Geometric sketching total runtime is in the same order as the reported ~30 minutes on the same matrix and sketch size
- sketch_indicator_output.csv has exactly sketch_size (20296) entries set to 1 out of 2,026,641 samples

## Limitations

- No software version numbers are stated for Sphetcher or the baselines, so exact runtime reproduction may vary across builds and hardware
- Sphetcher used Pearson correlation as the distance metric in most experiments; runtime is tied to that choice of metric and distance-computation stage
- The reported 16-minute figure assumes Sphetcher combined with a prior grid sampling strategy (Sphetcher-H); plain Sphetcher alone does not scale to two million cells this way

## Evidence

- [intro] Sphetcher requires only 16 minutes to compute a sketch for a mouse embryonic dataset comprising two million cells.: "Sphetcher requires only 16 minutes to compute a sketch for a mouse embryonic dataset comprising two million cells."
- [methods] Mouse organogenesis cell atlas (MOCA) from Cao et al. (2019) (2,026,641 cells): "Mouse organogenesis cell atlas (MOCA) from [Cao et al. (2019)](https://www.nature.com/articles/s41586-019-0969-x) (2,026,641 cells)"
- [readme] A compiler that supports C++11 is needed to build sphetcher.: "A compiler that supports C++11 is needed to build sphetcher. You can download and compile the latest code from github as follows"
- [readme] sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv: "sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv"
- [readme] Output sketch_indicator_output.csv: an indicator vector of n samples where 1 indicates the sample is in the sketch: "`sketch_indicator_output.csv` : an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise"
