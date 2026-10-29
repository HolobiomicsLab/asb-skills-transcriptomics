---
name: large-scale-cell-distance-matrix-computation
description: Use when sketching or sampling cells from an expression matrix with >1
  million cells (rows = cells, columns = genes/PCs) and you need cell-cell distances
  or a timing comparison of the distance-computation stage.
when_to_use_negative:
- The dataset is small enough that sketching is unnecessary and a full-dataset analysis
  (e.g., SC3 consensus clustering, Monocle 2 trajectory inference) is tractable directly
- The input is not a cell-by-feature matrix with rows as cells (e.g., already a precomputed
  sketch or a pairwise distance file), since Sphetcher expects CSV expression matrices
- The goal is unbiased uniform downsampling rather than data-dependent coverage of
  transcriptomic space, since uniform subsampling ignores similarity between gene
  expression patterns and risks overlooking rare cell states
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Sphetcher
  role: Computes the cell-cell distance matrix (Pearson correlation) as the ~600 s
    distance stage of the hybrid Sphetcher-H pipeline (grid sampling + spherical thresholding
    set cover) for the 2M-cell MOCA sketch
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Used to clone the Sphetcher repository before compilation (git clone https://github.com/canzarlab/Sphetcher)
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:b75df700f1eeeca0b386c593eb0e4438b7640ac602f3e1ed2efd5a8bd624c524
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Sphetcher
  - git
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# large-scale-cell-distance-matrix-computation

## Summary

Compute all-pairs cell-cell distances (e.g., Pearson correlation) over a multi-million-cell single-cell RNA-seq expression matrix as the expensive distance-computation stage of Sphetcher-H, so that runtime can be benchmarked against geometric sketching on the 2,026,641-cell MOCA dataset.

## When to use

Use when sketching or sampling cells from an expression matrix with >1 million cells (rows = cells, columns = genes/PCs) and you need cell-cell distances or a timing comparison of the distance-computation stage.

## When NOT to use

- The dataset is small enough that sketching is unnecessary and a full-dataset analysis (e.g., SC3 consensus clustering, Monocle 2 trajectory inference) is tractable directly
- The input is not a cell-by-feature matrix with rows as cells (e.g., already a precomputed sketch or a pairwise distance file), since Sphetcher expects CSV expression matrices
- The goal is unbiased uniform downsampling rather than data-dependent coverage of transcriptomic space, since uniform subsampling ignores similarity between gene expression patterns and risks overlooking rare cell states

## Inputs

- expression_matrix.csv: cell-by-feature expression/PC matrix in CSV format (rows = cells, columns = genes, transcripts, or PCs), e.g., MOCA (Cao et al. 2019, 2,026,641 cells)
- sketch_size: integer number of samples to sketch (e.g., 20296 = 1% of MOCA cells)
- optional class_labels.csv: prior cell labels (integer classes 1..K) with l_min minimum representatives per class

## Outputs

- sketch_indicator_output.csv: indicator vector of n samples with exactly sketch_size entries equal to 1
- runtime_comparison.csv: per-stage and total timings (grid sampling, distance computation, set cover) for Sphetcher-H and the geometric sketching baseline

## How to apply

Clone github.com/canzarlab/Sphetcher and compile the C++ source with a C++11-supporting compiler (cd src; make). Provide the expression matrix as a CSV where rows are cells and columns are features (genes, transcripts, or principal components), e.g., the Mouse organogenesis cell atlas (MOCA) from Cao et al. 2019 with 2,026,641 cells. Run Sphetcher-H, which first applies a grid sampling strategy and then spherical thresholding (set cover), recording elapsed time of the grid-sampling, distance-computation, and set-cover stages separately at sketch size 20296 (1% of cells). Use Pearson correlation as the distance metric, as in most experiments in the study. Run the geometric box-based sketching baseline on the same matrix and sketch size, then aggregate per-stage and total timings (e.g., into runtime_comparison.csv) to compare against the reported ~16 minutes for Sphetcher-H (grid sampling 10 s, distances 600 s, set cover 400 s) versus ~30 minutes for geometric sketching.

## Related tools

- **Sphetcher** (Computes the cell-cell distance matrix (Pearson correlation) as the ~600 s distance stage of the hybrid Sphetcher-H pipeline (grid sampling + spherical thresholding set cover) for the 2M-cell MOCA sketch) — https://github.com/canzarlab/Sphetcher
- **git** (Used to clone the Sphetcher repository before compilation (git clone https://github.com/canzarlab/Sphetcher)) — https://github.com/canzarlab/Sphetcher

## Examples

```
git clone https://github.com/canzarlab/Sphetcher && cd src && make && sphetcher moca_expression.csv 20296 sketch_indicator_output.csv
```

## Evaluation signals

- Sphetcher-H total runtime on the 2,026,641-cell MOCA matrix is on the order of the reported 16 minutes (grid sampling 10 s, distances 600 s, set cover 400 s)
- Geometric sketching baseline total runtime is around 30 minutes on the same matrix and sketch size, so Sphetcher-H is roughly 2x faster
- sketch_indicator_output.csv has exactly sketch_size (e.g., 20296) entries equal to 1 out of n cells
- runtime_comparison.csv contains timing entries for both methods and all timed stages (grid sampling, distances, set cover)
- Compilation succeeds with a C++11-supporting compiler via make in the src directory

## Limitations

- No software version numbers are stated for Sphetcher or the geometric sketching baseline, so exact runtime reproduction may vary across builds
- The 16-minute figure is specific to the two-million-cell MOCA dataset; timings will differ for other datasets and sketch sizes
- The interplay of sketching with a given distance metric (Pearson correlation) and subsequent algorithmic analysis was not systematically addressed, and results are reported for a specific combined pipeline

## Evidence

- [intro] Sphetcher-H total runtime on MOCA: "Sphetcher requires only 16 minutes to compute a sketch for a mouse embryonic dataset comprising two million cells."
- [readme] Compilation requirement: "A compiler that supports C++11 is needed to build sphetcher."
- [readme] Input format: "First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs)."
- [discussion] Distance metric: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric"
- [methods] MOCA dataset size: "Mouse organogenesis cell atlas (MOCA) from [Cao et al. (2019)](https://www.nature.com/articles/s41586-019-0969-x) (2,026,641 cells)"
