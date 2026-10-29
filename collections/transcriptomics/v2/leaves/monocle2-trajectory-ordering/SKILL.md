---
name: monocle2-trajectory-ordering
description: 'Use when benchmarking pseudotime inference: a single-cell gene expression
  matrix with a known or hypothesized developmental ordering is available and Monocle2
  should be run as a comparison method whose cell ordering is evaluated against a
  reference ordering.'
when_to_use_negative:
- When the goal is to use Monocle2's own branching detection as the primary analysis
  rather than a pseudotime-ordering baseline against another method's trajectory
- When the dataset lacks a reference ordering (known stages, prior trajectory, or
  simulation ground truth), since the Kendall rank correlation evaluation cannot be
  computed
- When the input is a dimensionality-reduced embedding (e.g., approximate principal
  components) rather than a gene expression matrix that Monocle2 can consume directly
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Monocle2
  role: R-based pseudotime trajectory inference method; produces the baseline cell
    ordering being evaluated in this skill
- name: DTFLOW
  role: Python method compared against Monocle2; its reported Kendall tau (0.862 on
    the ME dataset) defines the reference the Monocle2 baseline is contrasted with
  repo: https://github.com/statway/DTFLOW
- name: Scanpy
  role: Provides the DPT implementation (p=1 special case of RWR) used as a second
    comparator alongside Monocle2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.08.003
    title: 'DTFLOW: Inference and Visualization of Single-Cell Pseudotime Trajectory
      Using Diffusion Propagation'
  source_content_hash: sha256:9d9ec6875793fdd4f257a9c5adf2504d8c42de81966adfb215897115868f3556
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/statway/DTFLOW
  tools:
  - DTFLOW
  - Monocle2
  - Python
  - Scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# monocle2-trajectory-ordering

## Summary

Applies Monocle2 to order single cells along a differentiation pathway (pseudotime) so its trajectory inference can serve as a published baseline against diffusion-propagation methods such as DTFLOW, scored by Kendall rank correlation.

## When to use

Use when benchmarking pseudotime inference: a single-cell gene expression matrix with a known or hypothesized developmental ordering is available and Monocle2 should be run as a comparison method whose cell ordering is evaluated against a reference ordering.

## When NOT to use

- When the goal is to use Monocle2's own branching detection as the primary analysis rather than a pseudotime-ordering baseline against another method's trajectory
- When the dataset lacks a reference ordering (known stages, prior trajectory, or simulation ground truth), since the Kendall rank correlation evaluation cannot be computed
- When the input is a dimensionality-reduced embedding (e.g., approximate principal components) rather than a gene expression matrix that Monocle2 can consume directly

## Inputs

- Pre-processed single-cell gene expression matrix (N cells x D genes; e.g., ME RT-PCR dataset, 438 cells x 48 genes)
- Reference cell ordering or developmental stage labels (e.g., the seven known ME developmental stages) for Kendall tau computation

## Outputs

- Monocle2-inferred pseudotime ordering of all cells
- Kendall rank correlation coefficient (kendall_tau) for Monocle2 on the dataset, e.g., written to kendall_correlations.csv with columns method and kendall_tau

## How to apply

Run Monocle2 (R) on the same pre-processed single-cell expression matrix used by the other methods — in the DTFLOW benchmark this was the mouse-embryo (ME) high-throughput RT-PCR dataset (guo2010) with 438 cells and 48 genes spanning seven developmental stages from 1-cell zygote to 64-cell blastocyst. Extract Monocle2's inferred pseudotime ordering of the 438 cells, then compute the Kendall rank correlation coefficient between the Monocle2 ordering and the known developmental stage ordering. Write the coefficient with columns 'method' and 'kendall_tau' (expected value ~0.761 for the ME dataset in the DTFLOW paper). Use the resulting tau as the baseline against which DTFLOW (restart probability p=0.9, k=10 nearest neighbors, BKFD/SVD embedding) and Scanpy's DPT (the p=1 special case of random walk with restart) are compared. Judge success when the computed tau is close to the paper's reported value, allowing small deviation from implementation differences.

## Related tools

- **Monocle2** (R-based pseudotime trajectory inference method; produces the baseline cell ordering being evaluated in this skill)
- **DTFLOW** (Python method compared against Monocle2; its reported Kendall tau (0.862 on the ME dataset) defines the reference the Monocle2 baseline is contrasted with) — https://github.com/statway/DTFLOW
- **Scanpy** (Provides the DPT implementation (p=1 special case of RWR) used as a second comparator alongside Monocle2)

## Evaluation signals

- Kendall tau for Monocle2 on the ME dataset is close to the paper's reported 0.761 (DTFLOW: 0.862, Scanpy: 0.796 reported alongside)
- The inferred Monocle2 ordering covers all cells in the input matrix (no cells dropped during ordering)
- Output CSV conforms to the schema: columns 'method' and 'kendall_tau', one row per method compared
- Monocle2 is run on the identical pre-processed expression matrix as DTFLOW and Scanpy so the Kendall taus are directly comparable

## Limitations

- Monocle2 achieved the lowest Kendall tau of the three methods on both the ME dataset (0.761) and the MFG dataset (0.569) in the DTFLOW benchmark
- The article reports no software versions or standardized parameters for the compared methods, so exact reproduction may vary across Monocle2 releases

## Evidence

- [results] Kendall coefficients ME dataset: "The calculated Kendall rank correlation coefficients are 0.862, 0.796, 0.761 for DTFLOW, Scanpy, and Monocle2, respectively"
- [intro] Monocle2 among pseudotime methods: "These methods include Wanderlust [6], Wishbone [7], TSCAN [8], Monocle [5], Monocle2 [9], Waterfall [10], SCOUT [11], DensityPath [12], and SoptSC [13]."
- [methods] DPT special case of RWR: "the RWR algorithm can be regarded as a more general approach and DPT is a special case of the RWR algorithm (i.e., p=1)"
- [results] Kendall coefficients MFG dataset: "The calculated Kendall rank correlation coefficients are 0.761, 0.702, and 0.569 for DTFLOW, Scanpy, and Monocle2, correspondingly."
