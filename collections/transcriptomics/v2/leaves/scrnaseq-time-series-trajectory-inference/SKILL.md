---
name: scrnaseq-time-series-trajectory-inference
description: Use when a large single-cell RNA-seq dataset with collection time-point
  labels needs a balanced subsample before trajectory inference, especially to exclude
  cells whose pseudotime disagrees with their collection time point.
when_to_use_negative:
- The input is already a small or pre-filtered dataset where uniform subsampling would
  not risk overlooking rare cell states or unbalanced time-point representation
- The downstream algorithm's assumptions are incompatible with Sphetcher's model —
  the article notes compatibility was only shown for Louvain community detection and
  Monocle 2
- No prior categorical information (cell type, collection time point) exists and fair
  sampling per class is not needed; use Sphetcher's two-argument mode instead
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: Sphetcher
  role: Computes the spherical sketch of cells with fairness constraints (class_labels.csv
    and l_min) using the default Pearson correlation distance metric, producing sketch_indicator_output.csv
  repo: https://github.com/canzarlab/Sphetcher
- name: Monocle 2
  role: Runs single-cell trajectory inference on the restricted expression matrix
    of sketched HSMM cells to obtain pseudotime ordering of myoblast differentiation
- name: git
  role: Used to clone the Sphetcher repository for compilation with a C++11 compiler
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:64ac2cfc3b986d13d63361b04e0c1f49fd03ad328191b754bd7900231a25cc54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Monocle 2
  - Sphetcher
  - git
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# scrnaseq-time-series-trajectory-inference

## Summary

A fairness-constrained single-cell sketching workflow: Sphetcher selects a representative sketch of cells from an scRNA-seq expression matrix while enforcing a minimum number of representatives (l_min) per collection time-point class, and the sketch feeds Monocle 2 trajectory inference on the HSMM human skeletal muscle myoblast differentiation data.

## When to use

Use when a large single-cell RNA-seq dataset with collection time-point labels needs a balanced subsample before trajectory inference, especially to exclude cells whose pseudotime disagrees with their collection time point.

## When NOT to use

- The input is already a small or pre-filtered dataset where uniform subsampling would not risk overlooking rare cell states or unbalanced time-point representation
- The downstream algorithm's assumptions are incompatible with Sphetcher's model — the article notes compatibility was only shown for Louvain community detection and Monocle 2
- No prior categorical information (cell type, collection time point) exists and fair sampling per class is not needed; use Sphetcher's two-argument mode instead

## Inputs

- HSMM expression matrix in CSV format (rows are cells, columns are features such as genes/PCs), e.g. expression_matrix.csv from /data
- Class-label column vector encoding collection time points as integers 1..K, e.g. class_labels.csv
- sketch_size: number of cells to select
- l_min: minimum number of representatives to preserve from each time-point class (set to at least 4)

## Outputs

- sketch_indicator_output.csv: indicator vector of n samples with exactly sketch_size 1's marking selected cells
- Trajectory visualization reproducing Figure 5 (e.g. figure5_hsmm_trajectory.png), with supplementary views for Figures S5-S7

## How to apply

Load the HSMM expression matrix (rows are cells, columns are features such as genes/PCs) and the class-label vector encoding collection time points from /data. Run Sphetcher in its fair-sampling mode, 'sphetcher expression_matrix.csv sketch_size class_labels.csv l_min sketch_indicator_output.csv', setting l_min to enforce at least four representatives per time-point class and keeping Sphetcher's default Pearson correlation distance metric; the fairness constraint encourages exclusion of fully differentiated cells from time point 0 whose pseudotime disagrees with their collection time. Verify the returned indicator vector contains exactly sketch_size 1's and at least l_min selected cells from every class. Restrict the expression matrix to sketched cells and run Monocle 2 trajectory inference to obtain pseudotime ordering of HSMM differentiation, then compare inferred pseudotime against collection time points to confirm outlier cells were excluded, rendering the trajectory plot reproducing Figure 5 (with supplementary views for Figures S5-S7).

## Related tools

- **Sphetcher** (Computes the spherical sketch of cells with fairness constraints (class_labels.csv and l_min) using the default Pearson correlation distance metric, producing sketch_indicator_output.csv) — https://github.com/canzarlab/Sphetcher
- **Monocle 2** (Runs single-cell trajectory inference on the restricted expression matrix of sketched HSMM cells to obtain pseudotime ordering of myoblast differentiation)
- **git** (Used to clone the Sphetcher repository for compilation with a C++11 compiler) — https://github.com/canzarlab/Sphetcher

## Examples

```
sphetcher zeisel_pca.csv 1000 zeisel_pca_labels.csv 3 sketch_indicator_output.csv
```

## Evaluation signals

- sketch_indicator_output.csv has exactly sketch_size entries equal to 1 out of n samples
- Every time-point class contributes at least l_min selected cells to the sketch
- Cells observed in the full-data analysis with pseudotime/collection-time-point disagreement (fully differentiated cells from time point 0) are excluded from the sketch, as in Figures 5 (right) and S5
- Monocle 2 pseudotime ordering of sketched HSMM cells is consistent with the original collection time points, and the rendered trajectory reproduces Figure 5 (and supplementary Figures S5-S7)
- Both output files are produced: sketch_indicator_output.csv and figure5_hsmm_trajectory.png

## Limitations

- The interplay of sketching with respect to a given distance metric and subsequent algorithmic analysis was not systematically addressed in the study
- Benefits of leveraging prior categorical knowledge (e.g., cell types, collection time points) in sketch selection are demonstrated only at proof-of-principle level; additional experiments are required
- Sphetcher's underlying model might be less compatible with assumptions made by algorithms other than Louvain and Monocle 2, and no software version numbers are stated for Sphetcher or downstream tools

## Evidence

- [abstract] We show how a fair sampling can inform the inference of the trajectory of human skeletal muscle myoblast differentiation: "We show how a fair sampling can inform the inference of the trajectory of human skeletal muscle myoblast differentiation"
- [results] Fairness excludes time-point-outlier cells: [evidence span withheld — anchor status: not_found]
- [readme] Fair-sampling CLI usage: "sphetcher expression_matrix.csv sketch_size class_labels.csv l_min sketch_indicator_output.csv"
- [discussion] Pearson correlation default metric: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric"
- [discussion] Monocle 2 for trajectories: "Louvain community detection and Monocle 2 are widely used for scRNA-seq clustering and the inference of single-cell trajectories"
