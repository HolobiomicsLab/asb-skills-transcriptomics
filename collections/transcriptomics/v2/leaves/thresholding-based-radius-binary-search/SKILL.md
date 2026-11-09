---
name: thresholding-based-radius-binary-search
description: Use when a large single-cell RNA-seq expression matrix (rows = cells,
  columns = genes/PCs) must be reduced to a fixed-size representative subset that
  preserves rare cell states and covers the transcriptomic space, rather than uniform
  subsampling.
when_to_use_negative:
- When the goal is unbiased random subsampling rather than data-dependent, distance-based
  representative selection — uniform subsampling is simpler, though it risks overlooking
  rare cell states.
- When cells lack a meaningful distance measure or the input is not a sample-by-feature
  numeric matrix (rows must be cells, columns genes/PCs).
- When downstream analysis requires the full dataset's raw counts rather than a reduced
  sketch, or depends on algorithmic assumptions incompatible with a metric-space sphere
  model (the article notes compatibility was only tested with Louvain and Monocle
  2).
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Sphetcher
  role: Command-line tool that binary-searches the covering-sphere radius and runs
    the disk-friendly greedy set-cover step to pick sphere centers, emitting the sketch
    indicator vector
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Used to clone the Sphetcher repository before compiling with a C++11 compiler
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:0a988db9598c75d7024686116acbfbdd563a2efde903a78f2df13f0eefba943e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Sphetcher
  - git
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# thresholding-based-radius-binary-search

## Summary

Implements Sphetcher's spherical thresholding scheme: a radius is guessed and binary-searched while a disk-friendly greedy set-cover step selects sphere centers, yielding a fixed-size sketch of cells that evenly covers the transcriptomic space.

## When to use

Use when a large single-cell RNA-seq expression matrix (rows = cells, columns = genes/PCs) must be reduced to a fixed-size representative subset that preserves rare cell states and covers the transcriptomic space, rather than uniform subsampling.

## When NOT to use

- When the goal is unbiased random subsampling rather than data-dependent, distance-based representative selection — uniform subsampling is simpler, though it risks overlooking rare cell states.
- When cells lack a meaningful distance measure or the input is not a sample-by-feature numeric matrix (rows must be cells, columns genes/PCs).
- When downstream analysis requires the full dataset's raw counts rather than a reduced sketch, or depends on algorithmic assumptions incompatible with a metric-space sphere model (the article notes compatibility was only tested with Louvain and Monocle 2).

## Inputs

- expression_matrix.csv — expression matrix in CSV format, rows are cells, columns are features (genes, transcripts, or principal components)
- sketch_size — integer number of samples to obtain from the dataset (e.g. 1000)
- class_labels.csv (optional) — column vector of prior class labels, each class an integer between 1 and K
- l_min (optional) — minimum number of representatives to sample from each class

## Outputs

- sketch_indicator_output.csv — binary indicator vector over all n input samples with exactly sketch_size ones (selected sketch cells) and zeros elsewhere

## How to apply

Provide an expression matrix in CSV format where rows are samples (cells) and columns are features (genes, transcripts, or principal components, e.g. zeisel_pca.csv). Build the Sphetcher C++11 executable with 'make' in the cloned repository's src directory. Invoke 'sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv'; the tool iterates over covering-sphere radius guesses (a thresholding/binary-search over radii), and for each radius runs the disk-friendly greedy set-cover algorithm to pick sphere centers — trying a smaller radius if the sketch has too few cells and a larger radius if it has too many, until exactly sketch_size centers are chosen. Optionally supply class_labels.csv with a minimum representatives parameter l_min to preserve a minimum number of samples per prior category (e.g. cell type or collection time point).

## Related tools

- **Sphetcher** (Command-line tool that binary-searches the covering-sphere radius and runs the disk-friendly greedy set-cover step to pick sphere centers, emitting the sketch indicator vector) — https://github.com/canzarlab/Sphetcher
- **git** (Used to clone the Sphetcher repository before compiling with a C++11 compiler) — https://github.com/canzarlab/Sphetcher

## Examples

```
git clone https://github.com/canzarlab/Sphetcher; cd src; make; sphetcher zeisel_pca.csv 1000 sketch_indicator_output.csv
```

## Evaluation signals

- Parse sketch_indicator_output.csv and confirm it is a binary indicator vector over all input samples with exactly sketch_size (e.g. 1000) ones and zeros elsewhere.
- Verify the sketch covers the transcriptomic space under the chosen distance metric (e.g. Pearson correlation) — every cell should fall within a sphere around a selected center.
- Confirm the input matrix was correctly formatted: rows = cells, columns = features, CSV; the example uses /data/zeisel_pca.csv.
- If fairness labels were supplied, check each class retains at least l_min representatives in the sketch.
- Compare sketch-based downstream results (e.g. Louvain clustering, Monocle 2 trajectories) to full-data results to confirm the sketch faithfully represents the transcriptomic landscape.

## Limitations

- The interplay of sketching with respect to a given distance metric and subsequent algorithmic analysis was not systematically addressed in the study.
- Benefits of leveraging prior categorical knowledge (e.g. cell types) in sketch selection were only demonstrated at proof-of-principle level; additional experiments are required.
- Sphetcher's underlying model might be less compatible with assumptions made by algorithms other than Louvain and Monocle 2.
- No software version numbers are stated for Sphetcher or the downstream tools used.

## Evidence

- [intro] Sphetcher guesses a covering-sphere radius and iterates over guesses: for each radius, it computes the smallest size sketch that covers all cells via greedy set cover: "makes use of the thresholding technique originally proposed for the design of approximation algorithms for bottleneck problems to efficiently pick representative"
- [methods] A compiler that supports C++11 is needed to build sphetcher.: "A compiler that supports C++11 is needed to build sphetcher."
- [methods] First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs).: "First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs)."
- [readme] README example invocation for the thresholding sketch on the zeisel PCA input: "sphetcher zeisel_pca.csv 1000 sketch_indicator_output.csv"
- [readme] README output format specification for the indicator vector: "an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector)"
- [discussion] Distance metric used in most experiments: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric"
