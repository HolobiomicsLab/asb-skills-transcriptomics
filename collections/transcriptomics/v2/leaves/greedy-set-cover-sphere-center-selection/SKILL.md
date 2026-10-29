---
name: greedy-set-cover-sphere-center-selection
description: Use when a large scRNA-seq expression matrix (cells x genes/PCs, e.g.
  CSV of principal components) must be reduced to a fixed number of representative
  cells that evenly cover transcriptomic space before downstream clustering or trajectory
  inference.
when_to_use_negative:
- The dataset is small enough that full-dataset analysis (e.g. SC3 consensus clustering
  or Monocle 2 on all cells) is already computationally feasible
- The analysis requires uniform random subsampling or box/geometric (k-means++) sketching
  rather than sphere-based coverage — Sphetcher specifically improves on geometric
  box approximations
- Only a per-cluster random sample is needed and cluster labels exist; clustering-then-sampling
  (dropClust) may suffice, though it provides unbalanced samples that hamper downstream
  analyses
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Sphetcher
  role: Command-line C++ tool that binary-searches the covering-sphere radius and
    runs greedy set cover to output the sketch indicator vector
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Used to clone the Sphetcher source repository before compiling with a C++11-supporting
    compiler
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:738d370915d2aad7056861cb2d83d8fc7e5c64d2d2ff901b888ba5a833be0fb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Sphetcher
  - git
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# greedy-set-cover-sphere-center-selection

## Summary

Selects a fixed-size sketch of representative cells from a single-cell expression matrix by binary-searching a covering-sphere radius and using a greedy set-cover step to pick sphere centers, as implemented in the Sphetcher tool. It approximates the transcriptomic space with spheres rather than boxes, preserving rare cell states better than uniform subsampling.

## When to use

Use when a large scRNA-seq expression matrix (cells x genes/PCs, e.g. CSV of principal components) must be reduced to a fixed number of representative cells that evenly cover transcriptomic space before downstream clustering or trajectory inference.

## When NOT to use

- The dataset is small enough that full-dataset analysis (e.g. SC3 consensus clustering or Monocle 2 on all cells) is already computationally feasible
- The analysis requires uniform random subsampling or box/geometric (k-means++) sketching rather than sphere-based coverage — Sphetcher specifically improves on geometric box approximations
- Only a per-cluster random sample is needed and cluster labels exist; clustering-then-sampling (dropClust) may suffice, though it provides unbalanced samples that hamper downstream analyses

## Inputs

- expression_matrix.csv: expression matrix in CSV format, rows = cells, columns = features (genes, transcripts, or principal components), e.g. /data/zeisel_pca.csv
- sketch_size: integer number of samples (sphere centers) to select, e.g. 1000
- class_labels.csv (optional): column vector of prior class labels, each an integer 1..K
- l_min (optional): minimum number of representatives required per class

## Outputs

- sketch_indicator_output.csv: binary indicator vector of length n (number of input samples) with exactly sketch_size 1's marking the selected sketch cells and 0's elsewhere

## How to apply

Compile Sphetcher from github.com/canzarlab/Sphetcher with a C++11 compiler via 'git clone https://github.com/canzarlab/Sphetcher; cd src; make'. Provide an expression matrix CSV where rows are cells and columns are features (genes, transcripts, or principal components), plus the desired sketch size (e.g. 1000). The tool guesses a covering-sphere radius, computes via the disk-friendly greedy set-cover algorithm the smallest sketch whose spheres cover all cells, and shrinks or enlarges the radius (thresholding/binary search) until the sketch contains exactly the requested number of centers. Optionally supply a class_labels.csv column vector with classes 1..K and an l_min value to guarantee a minimum number of representatives per pre-defined category (fairness). The output sketch_indicator_output.csv is a binary indicator vector over all input samples whose 1's mark the selected sketch cells.

## Related tools

- **Sphetcher** (Command-line C++ tool that binary-searches the covering-sphere radius and runs greedy set cover to output the sketch indicator vector) — https://github.com/canzarlab/Sphetcher
- **git** (Used to clone the Sphetcher source repository before compiling with a C++11-supporting compiler) — https://github.com/canzarlab/Sphetcher

## Examples

```
git clone https://github.com/canzarlab/Sphetcher && cd src && make && sphetcher zeisel_pca.csv 1000 sketch_indicator_output.csv
```

## Evaluation signals

- sketch_indicator_output.csv contains exactly sketch_size (e.g. 1000) ones, one per selected sphere center, and zeros elsewhere over all n input samples
- Every input cell is covered by a sphere of the final thresholded radius centered at a selected sketch cell (full coverage of the transcriptomic space)
- The tool converged: the thresholding scheme terminated with the requested sketch size, neither too few nor too many centers for the final radius guess
- When class_labels.csv and l_min are supplied, each of the K classes contributes at least l_min selected representatives
- On large datasets (e.g. the 2-million-cell MOCA dataset), the sketch computation completes in practical time (~16 minutes when combined with prior grid sampling)

## Limitations

- The interplay of distance-metric-based sketching (Pearson correlation was used in most experiments) with internal preprocessing of downstream algorithms was not systematically evaluated
- Fairness via prior categorical labels was demonstrated only at proof-of-principle level; additional experiments are required to fully assess the benefits of leveraging prior knowledge such as cell types
- Sphetcher's underlying model might be less compatible with assumptions made by downstream algorithms other than Louvain and Monocle 2; no software version numbers are stated

## Evidence

- [other] Sphetcher guesses a covering-sphere radius and iterates over guesses: [evidence span withheld — anchor status: not_found]
- [methods] Thresholding scheme in methods: "A compiler that supports C++11 is needed to build sphetcher. You can download and compile the latest code from github"
- [readme] README usage: "sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv"
- [readme] README output format: "an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector)"
- [readme] README input format: "First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs)."
- [intro] Intro objective: "we propose algorithm Sphetcher that makes use of the thresholding technique originally proposed for the design of approximation algorithms for bottleneck problems to efficiently pick representative"
