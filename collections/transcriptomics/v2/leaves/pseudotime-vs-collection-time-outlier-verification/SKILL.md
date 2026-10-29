---
name: pseudotime-vs-collection-time-outlier-verification
description: Use when a scRNA-seq trajectory inference run shows cells whose pseudotime
  ordering contradicts their known collection time point, and you want to automatically
  remove such outliers via fair sampling.
when_to_use_negative:
- When no prior categorical knowledge (cell labels or collection time points) exists
  for the cells — the unconstrained Sphetcher mode without class labels is the appropriate
  variant.
- When the goal is general clustering or rare-population detection rather than verifying/removing
  pseudotime-versus-collection-time disagreement before trajectory inference.
- When the downstream trajectory algorithm is not Louvain or Monocle 2 — the paper
  notes Sphetcher's underlying model might be less compatible with assumptions made
  by other algorithms.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Sphetcher
  role: Fair-sampling sketching tool that selects representative cells with per-class
    minimum representatives via the l_min fairness constraint, using the Pearson correlation
    distance metric
  repo: https://github.com/canzarlab/Sphetcher
- name: Monocle 2
  role: Downstream single-cell trajectory inference on the sketched HSMM cells to
    obtain pseudotime ordering
- name: git
  role: Cloning the Sphetcher repository for compilation with a C++11 compiler
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:f568637acb8016c98f6fc48f8021ce259e430de3ccc402edf108a493355f0bec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Monocle 2
  - Sphetcher
  - git
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pseudotime-vs-collection-time-outlier-verification

## Summary

A fairness-constrained sketching procedure that uses Sphetcher with class labels (collection time points) to exclude outlier cells whose transcriptomic state disagrees with their collection time point before Monocle 2 trajectory inference on the HSMM human skeletal muscle myoblast dataset.

## When to use

Use when a scRNA-seq trajectory inference run shows cells whose pseudotime ordering contradicts their known collection time point, and you want to automatically remove such outliers via fair sampling.

## When NOT to use

- When no prior categorical knowledge (cell labels or collection time points) exists for the cells — the unconstrained Sphetcher mode without class labels is the appropriate variant.
- When the goal is general clustering or rare-population detection rather than verifying/removing pseudotime-versus-collection-time disagreement before trajectory inference.
- When the downstream trajectory algorithm is not Louvain or Monocle 2 — the paper notes Sphetcher's underlying model might be less compatible with assumptions made by other algorithms.

## Inputs

- HSMM expression matrix as CSV (rows are cells, columns are features/genes/PCs)
- class_labels.csv column vector encoding collection time points (integer classes 1..K)
- sketch_size (number of cells to sample)
- l_min (minimum representatives per class, e.g. 4)

## Outputs

- sketch_indicator_output.csv (indicator vector of n samples with sketch_size 1's)
- Single-cell pseudotime ordering of human skeletal muscle myoblast differentiation from Monocle 2 on the sketched cells
- figure5_hsmm_trajectory.png (trajectory visualization reproducing Figure 5, with supplementary views for Figures S5-S7)

## How to apply

Load the HSMM expression matrix (rows are cells, columns are features such as genes/PCs) and the class-label vector encoding collection time points from the /data directory. Run Sphetcher in fair-sampling mode ('sphetcher expression_matrix.csv sketch_size class_labels.csv l_min sketch_indicator_output.csv'), setting l_min to enforce a minimum of at least four representatives per collection time-point class and using the default Pearson correlation distance metric. Verify the output indicator vector contains exactly sketch_size 1's and at least l_min selected cells from every time-point class. Restrict the expression matrix to sketched cells using the indicator vector, run Monocle 2 trajectory inference, and compare inferred pseudotime against collection time points — the fairness constraint encourages Sphetcher to exclude fully differentiated cells from time point 0 (outlier cells with pseudotime/time-point disagreement, Figures 5 right and S5). Render the trajectory plot reproducing Figure 5 with supplementary views for Figures S5–S7, and save sketch_indicator_output.csv and figure5_hsmm_trajectory.png.

## Related tools

- **Sphetcher** (Fair-sampling sketching tool that selects representative cells with per-class minimum representatives via the l_min fairness constraint, using the Pearson correlation distance metric) — https://github.com/canzarlab/Sphetcher
- **Monocle 2** (Downstream single-cell trajectory inference on the sketched HSMM cells to obtain pseudotime ordering)
- **git** (Cloning the Sphetcher repository for compilation with a C++11 compiler) — https://github.com/canzarlab/Sphetcher

## Examples

```
sphetcher expression_matrix.csv sketch_size class_labels.csv l_min sketch_indicator_output.csv
```

## Evaluation signals

- The output indicator vector has exactly sketch_size 1's among n samples.
- Every time-point class contributes at least l_min selected cells (per the paper's HSMM analysis, at least four representatives per collection time-point class).
- Outlier cells with pseudotime/time-point disagreement observed in the full-data analysis are excluded from the sketch (Figures 5 right and S5).
- Monocle 2 pseudotime on the sketched cells agrees with the original collection time points, reproducing Figure 5 (and supplementary views for Figures S5-S7).
- Outputs are saved as sketch_indicator_output.csv and figure5_hsmm_trajectory.png.

## Limitations

- Benefits of leveraging prior categorical knowledge (e.g., cell types or collection time points) in sketch selection are only demonstrated at proof-of-principle level; additional experiments are required.
- The interplay of sketching with respect to a given distance metric and subsequent algorithmic analysis was not systematically addressed in the study.
- Sphetcher's underlying model might be less compatible with assumptions made by algorithms other than Louvain community detection and Monocle 2.
- No software version numbers are stated for Sphetcher or the downstream tools used.

## Evidence

- [abstract] Fair sampling can inform the inference of the trajectory of human skeletal muscle myoblast differentiation: "We show how a fair sampling can inform the inference of the trajectory of human skeletal muscle myoblast differentiation"
- [intro] Fairness constraint on sketching with pre-defined categories: "our optimization scheme naturally allows to include fairness aspects that require to include cells of each pre-defined category that can encode prior biological or experimental knowledge such as cell"
- [readme] Fair-sampling CLI usage with class labels and l_min: "sphetcher expression_matrix.csv sketch_size class_labels.csv l_min sketch_indicator_output.csv"
- [discussion] Pearson correlation distance metric used in most experiments: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric"
- [methods] Prior information usage for preserving representatives per category: "Additionally you can provide the prior information (e.g, cell label, collection time point) in case you want to preserve certain number of samples from each category."
