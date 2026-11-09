---
name: metric-space-distance-thresholding
description: Use when a scRNA-seq expression matrix (cells x genes/PCs) is too large
  for downstream clustering, trajectory inference, or visualization, and a fixed-size
  representative subset that preserves rare cell states is needed.
when_to_use_negative:
- When the dataset is already small enough that current computational analysis methods
  handle routine tasks (clustering, trajectory inference, visualization) without sketching.
- When a uniform random subsample is acceptable and similarity/dissimilarity between
  gene expression patterns need not be preserved — though note uniform subsampling
  risks overlooking rare cell states.
- When downstream algorithm assumptions are incompatible with a metric-sphere model;
  the article notes Sphetcher's model might be less compatible with algorithms other
  than Louvain community detection and Monocle 2.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Sphetcher
  role: Command-line tool that performs the spherical thresholding and greedy set-cover
    sketch selection; built from source with a C++11 compiler
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Used to clone the Sphetcher source repository prior to compilation
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:50edbface5b6f0c734a344d1bcc52bbc479535a09d746ad9d9831850f96a09c4
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metric-space-distance-thresholding

## Summary

Sketching of large single-cell RNA-seq datasets by selecting representative cells within covering spheres using thresholding over sphere radii combined with greedy set cover, as implemented in Sphetcher. Produces a fixed-size subset ('spherical sketch') that evenly covers the transcriptomic space better than uniform or box-based (geometric) sketches.

## When to use

Use when a scRNA-seq expression matrix (cells x genes/PCs) is too large for downstream clustering, trajectory inference, or visualization, and a fixed-size representative subset that preserves rare cell states is needed.

## When NOT to use

- When the dataset is already small enough that current computational analysis methods handle routine tasks (clustering, trajectory inference, visualization) without sketching.
- When a uniform random subsample is acceptable and similarity/dissimilarity between gene expression patterns need not be preserved — though note uniform subsampling risks overlooking rare cell states.
- When downstream algorithm assumptions are incompatible with a metric-sphere model; the article notes Sphetcher's model might be less compatible with algorithms other than Louvain community detection and Monocle 2.

## Inputs

- expression_matrix.csv (CSV; rows = cells, columns = features/genes/transcripts/principal components)
- sketch_size (integer number of samples to select)
- class_labels.csv (optional column vector of integer class labels between 1 and K)
- l_min (optional minimum number of representatives per class)

## Outputs

- sketch_indicator_output.csv (binary indicator vector of length n with exactly sketch_size ones; 1 = cell selected into the sketch, 0 otherwise)

## How to apply

Provide an expression matrix in CSV format with rows as samples (cells) and columns as features (genes, transcripts, or principal components), such as the example zeisel_pca.csv. Clone the Sphetcher repository and compile the binary in the src directory with a C++11 compiler via make. Run the CLI with the expression matrix and a target sketch size: Sphetcher guesses a covering-sphere radius and, for each radius, computes the smallest sketch covering all cells via greedy set cover (disk-friendly greedy algorithm), decreasing the radius if the sketch contains too many cells and increasing it if too few, effectively binary-searching the radius until exactly sketch_size centers are chosen. Optionally supply class_labels.csv (integer labels 1..K) and an l_min parameter to guarantee a minimum number of representatives per prior category (e.g., cell type or collection time point). The output is sketch_indicator_output.csv, a binary indicator vector over all input samples with exactly sketch_size ones marking selected cells.

## Related tools

- **Sphetcher** (Command-line tool that performs the spherical thresholding and greedy set-cover sketch selection; built from source with a C++11 compiler) — https://github.com/canzarlab/Sphetcher
- **git** (Used to clone the Sphetcher source repository prior to compilation) — https://github.com/canzarlab/Sphetcher

## Examples

```
git clone https://github.com/canzarlab/Sphetcher && cd src && make; sphetcher zeisel_pca.csv 1000 sketch_indicator_output.csv
```

## Evaluation signals

- sketch_indicator_output.csv parses as a binary indicator vector over all input samples containing exactly sketch_size (e.g., 1000) ones and zeros elsewhere.
- The sketch size matches the requested target after the radius binary search converges (too few cells implies radius was decreased, too many implies increased).
- The resulting spherical sketch covers all cells in the transcriptomic space under the chosen distance measure (e.g., Pearson correlation was used in most experiments).
- Downstream analyses (e.g., Louvain clustering, Monocle 2 trajectory inference) run on the sketch reproduce the structure of the full dataset and can recover rare populations such as inflammatory macrophages.
- With prior class labels supplied, each of the K classes retains at least l_min representatives in the sketch.

## Limitations

- The interplay of sketching with respect to a given distance metric and subsequent algorithmic analysis was not systematically addressed in the study.
- Benefits of leveraging prior categorical knowledge (e.g., cell types) in sketch selection were only demonstrated at proof-of-principle level; additional experiments are required.
- Sphetcher's underlying model might be less compatible with assumptions made by algorithms other than Louvain and Monocle 2; compatibility was not tested.
- No software version numbers are stated for Sphetcher or the downstream tools used.

## Evidence

- [other] Sphetcher guesses a covering-sphere radius and iterates over guesses: [evidence span withheld — anchor status: not_found]
- [readme] CLI usage from README: "sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv"
- [readme] Output format from README: "an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector)."
- [readme] Compilation requirement from README: "A compiler that supports C++11 is needed to build sphetcher. You can download and compile the latest code from github as follows"
- [methods] Fairness option from article methods: "Additionally you can provide the prior information (e.g, cell label, collection time point) in case you want to preserve certain number of samples from each category."
- [discussion] Distance metric from discussion: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric"
