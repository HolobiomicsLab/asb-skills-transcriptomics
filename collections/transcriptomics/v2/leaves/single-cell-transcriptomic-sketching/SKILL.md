---
name: single-cell-transcriptomic-sketching
description: Use when a single-cell expression matrix (rows = cells, columns = genes/PCs)
  is too large for downstream clustering, trajectory inference, or visualization,
  and a fair, size-controlled cell subset that evenly covers the transcriptomic space
  is needed.
when_to_use_negative:
- The dataset is already small enough that clustering, trajectory inference, or visualization
  runs directly on all cells — sketching adds approximation error without benefit
- A uniform random subsample is acceptable and ignoring similarity between gene expression
  patterns poses no risk of overlooking rare cell states
- The analysis depends on a distance metric incompatible with the Pearson-correlation
  sphere model used here (the paper notes Sphetcher's model might be less compatible
  with algorithms other than Louvain and Monocle 2)
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Sphetcher
  role: Core binary that performs spherical thresholding and greedy set-cover sketch
    selection, emitting the sketch indicator vector
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Used to clone the Sphetcher repository before compiling with a C++11 compiler
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:f97e5464dcd9396925358e80016cb1a8caf2212be573e027d51cca2f28819358
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Sphetcher
  - git
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# single-cell-transcriptomic-sketching

## Summary

Sphetcher sketches massive single-cell RNA-seq datasets by selecting representative cells within covering spheres of the transcriptomic space via spherical thresholding and greedy set cover, producing a more faithful representation than box-based geometric sketching and surfacing rare cell states.

## When to use

Use when a single-cell expression matrix (rows = cells, columns = genes/PCs) is too large for downstream clustering, trajectory inference, or visualization, and a fair, size-controlled cell subset that evenly covers the transcriptomic space is needed.

## When NOT to use

- The dataset is already small enough that clustering, trajectory inference, or visualization runs directly on all cells — sketching adds approximation error without benefit
- A uniform random subsample is acceptable and ignoring similarity between gene expression patterns poses no risk of overlooking rare cell states
- The analysis depends on a distance metric incompatible with the Pearson-correlation sphere model used here (the paper notes Sphetcher's model might be less compatible with algorithms other than Louvain and Monocle 2)

## Inputs

- expression_matrix.csv — expression matrix in CSV format, rows are cells, columns are features (genes, transcripts, or principal components PCs)
- sketch_size — integer number of samples (cells) to obtain, e.g. 1%, 4%, or 10% of the dataset
- class_labels.csv (optional) — column vector of prior categories, each class an integer between 1 and K
- l_min (optional) — minimum number of representatives to sample from each prior class

## Outputs

- sketch_indicator_output.csv — binary indicator vector of length n (number of input cells) containing exactly sketch_size ones (selected sketch cells) and zeros elsewhere

## How to apply

Clone github.com/canzarlab/Sphetcher and compile the C++11 source with `make` in the src directory. Provide an expression matrix CSV (rows are samples/cells, columns are features/genes/transcripts/principal components PCs) and a target sketch_size; optionally supply class_labels.csv with integers 1..K per cell and an l_min to preserve a minimum number of representatives per prior category (cell type, collection time point). Run the binary, which guesses a covering-sphere radius and, for each radius, computes the smallest sketch covering all cells via greedy set cover (disk-friendly greedy), shrinking or growing the radius until the sketch contains exactly the requested number of centers. Use Pearson correlation as the distance metric, as in most experiments in the paper. The output sketch_indicator_output.csv is an indicator vector over all n input samples with exactly sketch_size ones marking the selected sketch cells.

## Related tools

- **Sphetcher** (Core binary that performs spherical thresholding and greedy set-cover sketch selection, emitting the sketch indicator vector) — https://github.com/canzarlab/Sphetcher
- **git** (Used to clone the Sphetcher repository before compiling with a C++11 compiler) — https://github.com/canzarlab/Sphetcher

## Examples

```
sphetcher zeisel_pca.csv 1000 sketch_indicator_output.csv
```

## Evaluation signals

- sketch_indicator_output.csv is a binary indicator vector over all n input samples containing exactly sketch_size ones and zeros elsewhere
- Hausdorff distance between the sketch and the full dataset is smaller than (or competitive with) the geometric sketch of Hie et al. at the same sketch size, across 10 random trials
- The sketch should reproduce expected downstream results, e.g. detecting the rare population of inflammatory macrophages and informing the Monocle 2 trajectory of human skeletal muscle myoblast differentiation
- When prior labels and l_min are supplied, each of the K classes retains at least l_min representatives in the sketch
- For very large datasets (Saunders 665,858 cells; MOCA 2,026,641 cells), prior grid sampling sketches should achieve Hausdorff distances competitive with geometric sketching at 1%–10% sizes

## Limitations

- The interplay of sketching with respect to a given distance metric and subsequent algorithmic analysis was not systematically addressed in the study
- Benefits of leveraging prior categorical knowledge (e.g., cell types) in sketch selection are shown only at proof-of-principle level; additional experiments are required
- Sphetcher's underlying model might be less compatible with assumptions made by algorithms other than Louvain community detection and Monocle 2
- No software version numbers are stated for Sphetcher or the downstream tools used

## Evidence

- [abstract] thresholding technique to pick representative sphere centers: "we propose algorithm Sphetcher that makes use of the thresholding technique to efficiently pick representative cells within spheres"
- [methods] compile with C++11: "A compiler that supports C++11 is needed to build sphetcher."
- [methods] input matrix format: "First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs)."
- [methods] prior labels usage: "Additionally you can provide the prior information (e.g, cell label, collection time point) in case you want to preserve certain number of samples from each category."
- [discussion] distance metric: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric"
- [readme] sketch definition: "It selects a small subset of cells referred to as sketch that evenly cover the transcriptomic space occupied by the original dataset."
- [readme] output format: "an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector)."
