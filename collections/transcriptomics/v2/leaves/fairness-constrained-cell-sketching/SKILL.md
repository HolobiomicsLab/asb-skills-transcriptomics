---
name: fairness-constrained-cell-sketching
description: Use when a large single-cell expression matrix must be subsampled for
  trajectory or clustering analysis while preserving a minimum number of cells per
  collection time point or class label, e.g. to remove outlier cells whose transcriptomic
  state disagrees with their time point.
when_to_use_negative:
- The downstream task needs no per-class guarantees — use Sphetcher's plain three-argument
  mode (sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv) instead
  of the fairness-constrained mode.
- The goal is uniform random subsampling rather than transcriptomic-space coverage;
  uniform subsampling ignores similarity between expression patterns and risks overlooking
  rare cell states, which Sphetcher is designed to avoid.
- The input is not a cell-by-feature matrix (rows must be cells/samples, columns genes/transcripts/PCs),
  so spherical thresholding over the Pearson correlation metric cannot be applied
  directly.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Sphetcher
  role: 'Runs the fairness-constrained spherical thresholding sketch: selects sketch_size
    representative cells within spheres using Pearson correlation distance, enforcing
    l_min representatives per class label'
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Clones the Sphetcher source repository for compilation with a C++11 compiler
  repo: https://github.com/canzarlab/Sphetcher
- name: Monocle 2
  role: Downstream single-cell trajectory inference on the fairness-sketched HSMM
    cells to obtain pseudotime ordering of human skeletal muscle myoblast differentiation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:075c29745529dbeb5e2f9d26ba2e0d83beea45c5c12c83723a353c845b4f4b4c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Monocle 2
  - Sphetcher
  - git
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# fairness-constrained-cell-sketching

## Summary

Fairness-constrained spherical sketching with Sphetcher: select a fixed-size representative subset of single cells while guaranteeing a minimum number of representatives (l_min) from each pre-defined class such as collection time point, so outlier cells with pseudotime/time-point disagreement are excluded from downstream trajectory inference.

## When to use

Use when a large single-cell expression matrix must be subsampled for trajectory or clustering analysis while preserving a minimum number of cells per collection time point or class label, e.g. to remove outlier cells whose transcriptomic state disagrees with their time point.

## When NOT to use

- The downstream task needs no per-class guarantees — use Sphetcher's plain three-argument mode (sphetcher expression_matrix.csv sketch_size sketch_indicator_output.csv) instead of the fairness-constrained mode.
- The goal is uniform random subsampling rather than transcriptomic-space coverage; uniform subsampling ignores similarity between expression patterns and risks overlooking rare cell states, which Sphetcher is designed to avoid.
- The input is not a cell-by-feature matrix (rows must be cells/samples, columns genes/transcripts/PCs), so spherical thresholding over the Pearson correlation metric cannot be applied directly.

## Inputs

- expression_matrix.csv — CSV expression matrix, rows are cells, columns are features (genes, transcripts, or principal components)
- class_labels.csv — column vector of prior class labels (e.g. collection time point), each class an integer between 1 and K
- sketch_size — integer number of cells to select
- l_min — integer minimum number of representatives required per class

## Outputs

- sketch_indicator_output.csv — indicator vector of n samples with exactly sketch_size 1's (1 = cell in the sketch) and >= l_min cells per class
- figure5_hsmm_trajectory.png — Monocle 2 trajectory visualization of sketched HSMM cells (with supplementary views for Figures S5–S7)

## How to apply

Compile Sphetcher from github.com/canzarlab/Sphetcher with a C++11 compiler (git clone, cd src, make). Prepare an expression matrix CSV (rows are cells, columns are features such as genes or PCs) and a class_labels.csv column vector encoding collection time points as integers 1..K from the /data directory. Run Sphetcher in its five-argument fair-sampling mode with a chosen sketch_size and l_min (e.g. l_min set to enforce at least four representatives per time-point class), keeping Sphetcher's default Pearson correlation distance metric. Validate that sketch_indicator_output.csv contains exactly sketch_size 1's and at least l_min selected cells from every class, then restrict the expression matrix to sketched cells and run Monocle 2 trajectory inference — for the HSMM dataset this reproduces Figure 5, where fairness prevents fully differentiated cells from time point 0 with pseudotime/time-point disagreement from entering the sketch.

## Related tools

- **Sphetcher** (Runs the fairness-constrained spherical thresholding sketch: selects sketch_size representative cells within spheres using Pearson correlation distance, enforcing l_min representatives per class label) — https://github.com/canzarlab/Sphetcher
- **git** (Clones the Sphetcher source repository for compilation with a C++11 compiler) — https://github.com/canzarlab/Sphetcher
- **Monocle 2** (Downstream single-cell trajectory inference on the fairness-sketched HSMM cells to obtain pseudotime ordering of human skeletal muscle myoblast differentiation)

## Examples

```
sphetcher expression_matrix.csv sketch_size class_labels.csv 4 sketch_indicator_output.csv
```

## Evaluation signals

- sketch_indicator_output.csv contains exactly sketch_size entries equal to 1.
- Every class (integer 1..K in class_labels.csv) contributes at least l_min selected cells to the sketch.
- Downstream Monocle 2 pseudotime of sketched HSMM cells shows improved agreement with original collection time points compared with the full-data analysis, reproducing Figure 5 (right) and S5.
- Fully differentiated outlier cells from time point 0 (pseudotime/time-point disagreement) are excluded from the sketch.
- Distance metric used is Sphetcher's default Pearson correlation as in the paper's experiments.

## Limitations

- The interplay of sketching with respect to a given distance metric and subsequent algorithmic analysis was not systematically addressed in the study.
- Benefits of leveraging prior categorical knowledge (e.g., cell types) in sketch selection are shown only at proof-of-principle level; additional experiments are required.
- Sphetcher's underlying model might be less compatible with assumptions made by other algorithms; compatibility was tested mainly with Louvain community detection and Monocle 2.
- No software version numbers are stated for Sphetcher or the downstream tools used.

## Evidence

- [intro] our optimization scheme naturally allows to include fairness aspects that require to include cells of each pre-defined category: "our optimization scheme naturally allows to include fairness aspects that require to include cells of each pre-defined category that can encode prior biological or experimental knowledge such as cell"
- [abstract] fairness excludes outlier time-point-0 cells: [evidence span withheld — anchor status: not_found]
- [readme] README fair-sampling CLI usage: "sphetcher expression_matrix.csv sketch_size class_labels.csv l_min sketch_indicator_output.csv"
- [readme] README class label input format: "Additionally you can provide the prior information (e.g, cell label, collection time point) in case you want to preserve certain number of samples from each category."
- [discussion] Pearson correlation default metric: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric"
- [discussion] Monocle 2 downstream trajectory inference: "Louvain community detection and Monocle 2 are widely used for scRNA-seq clustering and the inference of single-cell trajectories"
