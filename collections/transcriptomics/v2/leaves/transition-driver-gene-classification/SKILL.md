---
name: transition-driver-gene-classification
description: Use when you have MuTrans DynamicalAnalysis output (attractor basins,
  coarse-grained transition matrix, optimized membership function ρ*(x)) for a specific
  attractor-to-attractor transition and need the genes marking or driving that transition.
when_to_use_negative:
- Input is raw or un-clustered expression data with no attractor basins/membership
  function — run DynamicalAnalysis first; GeneAnalysis requires the multi-scale analysis
  output
- The dataset has very few cells (e.g., the myelopoiesis Fluidigm C1 dataset), where
  the article notes special care is needed to confirm transition-cell analysis
- The biological question is lineage/tree topology (root states, branch order) rather
  than gene classification — use InferLineage MPPT/MPFT instead
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: MuTrans
  role: Provides DynamicalAnalysis output (attractor basins, coarse-grained transition
    probabilities, membership assignment) and the GeneAnalysis / Transcendental procedure
    that computes TCS, transition entropy, and MS/IH/TD gene classes
  repo: https://github.com/cliffzhou92/MuTrans-release
- name: PyEMMA
  role: Used to compute effective transition-path currents fij+ with the coarse-grained
    dynamics for ranking transition trajectories; PyEMMA >= 2.5.6 is a required dependency
    of pyMuTrans
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25548-w
    title: Dissecting transition cells from single-cell transcriptome data through
      multiscale stochastic dynamics
  source_content_hash: sha256:265e2b5af9a7cd9aec5d904418cb985fb6ccd2f58d330e89e83380f1db3e3fdf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cliffzhou92/MuTrans-release
  tools:
  - MuTrans
  - PyEMMA
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# transition-driver-gene-classification

## Summary

Classify genes during a cell-state transition into transition-driver (TD), intermediate-hybrid (IH), and meta-stable (MS) classes using MuTrans's Transcendental downstream analysis, which orders transition cells by the transition cell score (TCS) and links gene expression to attractor/transition-cell structure.

## When to use

Use when you have MuTrans DynamicalAnalysis output (attractor basins, coarse-grained transition matrix, optimized membership function ρ*(x)) for a specific attractor-to-attractor transition and need the genes marking or driving that transition.

## When NOT to use

- Input is raw or un-clustered expression data with no attractor basins/membership function — run DynamicalAnalysis first; GeneAnalysis requires the multi-scale analysis output
- The dataset has very few cells (e.g., the myelopoiesis Fluidigm C1 dataset), where the article notes special care is needed to confirm transition-cell analysis
- The biological question is lineage/tree topology (root states, branch order) rather than gene classification — use InferLineage MPPT/MPFT instead

## Inputs

- MuTrans DynamicalAnalysis output object (attractor basins Sk*, coarse-grained transition matrix P̂ij, optimized membership function ρ*(x))
- Pre-processed single-cell gene expression matrix (N_cells x N_genes) from the MuTrans-release Data repository
- Indices i and j of the starting and target attractor states for the transition

## Outputs

- Per-cell transition cell score (TCS) and transition entropy values
- Per-gene class assignments as MS (meta-stable), IH (intermediate-hybrid), or TD (transition-driver)
- transition_cell_analysis.csv with tcs and entropy columns and gene class labels

## How to apply

Load the DynamicalAnalysis output together with the processed gene expression matrix from the MuTrans-release Data repository. For the transition from attractor Si* to Sj*, compute the transition cell score τij(x) = ρi*(x)/(ρi*(x)+ρj*(x)) and the per-cell transition entropy H(x) = −Σ_k ρk*(x) log ρk*(x) from the membership vector. Rank transition cells by TCS and flag TD genes whose expression correlates above a threshold with the TCS-ordered transition cells; define MS genes as significantly up-regulated in stable (meta-stable) cells within each cluster, and assign the remaining differentially expressed genes as IH genes co-expressed across stable and transition cells. Run this via the MuTrans GeneAnalysis(i, j, Output, par) call (Matlab) or the corresponding pyMuTrans GeneAnalysis routine, writing per-cell TCS/entropy and per-gene MS/IH/TD assignments to transition_cell_analysis.csv.

## Related tools

- **MuTrans** (Provides DynamicalAnalysis output (attractor basins, coarse-grained transition probabilities, membership assignment) and the GeneAnalysis / Transcendental procedure that computes TCS, transition entropy, and MS/IH/TD gene classes) — https://github.com/cliffzhou92/MuTrans-release
- **PyEMMA** (Used to compute effective transition-path currents fij+ with the coarse-grained dynamics for ranking transition trajectories; PyEMMA >= 2.5.6 is a required dependency of pyMuTrans)

## Examples

```
Genes = GeneAnalysis(i,j,Output,par)
```

## Evaluation signals

- transition_cell_analysis.csv exists and contains tcs and entropy columns
- All TCS and entropy values lie within [0, 1] (both are ratios/log-likelihoods of the membership vector)
- Each gene receives exactly one of the MS, IH, or TD class labels, and the three sets are disjoint
- TD genes' expression correlates above threshold with TCS-ordered transition cells; MS genes are significantly up-regulated in stable cells
- Expert plausibility review of gene classes (e.g., known EMT or hematopoiesis markers land in expected classes)

## Limitations

- Non-stationary effects due to cell cycle or cell proliferation dynamics were not considered in the current method
- A sufficiently large number of cells is required for high-resolution identification of transition cells; small datasets (e.g., myelopoiesis) need special care and confirmation
- TD/IH/MS threshold choices (expression-correlation cutoff, significance for MS genes) are not fixed by the paper and may need expert review per dataset

## Evidence

- [abstract] Downstream analysis identifies distinct genes that mark the transient states or drive the transitions.: "Downstream analysis identifies distinct genes that mark the transient states or drive the transitions."
- [intro] TD, IH, MS definitions from intro: "we are able to identify critical genes that are transition drivers (TD genes), mark the intermediate/hybrid states (IH genes) or meta-stable cells (MS genes)"
- [methods] TCS definition: [evidence span withheld — anchor status: not_found]
- [readme] GeneAnalysis call from README: "%% Transition cell and gene analysis from state transition i to j."
- [methods] Transcendental procedure: "MuTrans performs the Transcendental (transition cells and relevant analysis) procedure on each transition process to identify the transition cells from the stable cells"
