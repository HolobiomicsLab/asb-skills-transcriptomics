---
name: dense-subnetwork-extraction
description: Use when you have a subpopulation-specific directed TF-target score matrix
  (e.g., from DC3 deconvolved expression, ATAC, and HiChIP profiles) and need to identify
  the densely cross-regulating core regulatory network plus its upstream/downstream
  effectors.
when_to_use_negative:
- Input is raw scRNA-seq/scATAC-seq/HiChIP data rather than a deconvolved, subpopulation-specific
  regulatory network — run DC3 deconvolution and coupled clustering first
- Interaction graph is undirected or lacks TF-target orientation/weights, since the
  quadratic program requires a directed weighted W_ij
- The goal is subpopulation identification or clustering rather than distilling regulatory
  circuitry from an existing subpopulation-specific network
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: DC3
  role: Produces the subpopulation-specific expression, accessibility, and enhancer-promoter
    loop profiles from which the subpopulation-specific regulatory networks (input
    W_ij) are constructed
  repo: github.com/SUwonglab/DC3
- name: scipy
  role: Solves the quadratic program (max Σ W_ij x_i y_j under L1-type constraints)
    via SLSQP or projected gradient and supports the randomization/statistics steps
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-12547-1
    title: DC3 is a method for deconvolution and coupled clustering from bulk and
      single-cell genomics data
  source_content_hash: sha256:6edce23d9089e156c73ec8c29d010cb8bd3d9c9753255f6467ec5fa6992a2627
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DC3
  - scipy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# dense-subnetwork-extraction

## Summary

Extracts a statistically dense subnetwork from each subpopulation-specific TF-gene regulatory network by solving a quadratic program over the directed edge-weight matrix, then partitions it into core, upstream, and downstream TF sets. It distills DC3's deconvolved expression/accessibility/HiChIP profiles into the core regulatory circuitry that maintains each cellular state.

## When to use

Use when you have a subpopulation-specific directed TF-target score matrix (e.g., from DC3 deconvolved expression, ATAC, and HiChIP profiles) and need to identify the densely cross-regulating core regulatory network plus its upstream/downstream effectors.

## When NOT to use

- Input is raw scRNA-seq/scATAC-seq/HiChIP data rather than a deconvolved, subpopulation-specific regulatory network — run DC3 deconvolution and coupled clustering first
- Interaction graph is undirected or lacks TF-target orientation/weights, since the quadratic program requires a directed weighted W_ij
- The goal is subpopulation identification or clustering rather than distilling regulatory circuitry from an existing subpopulation-specific network

## Inputs

- Directed weighted TF-target adjacency matrix W_ij (edge-weight scores) for a DC3-inferred subpopulation-specific regulatory network
- TF gene list defining candidate regulators (core/upstream/downstream node labels)
- Optional synthetic directed weighted graph for validation

## Outputs

- Dense subnetwork node set (nodes with nonzero x_i or y_i)
- Core / upstream / downstream partition (C = {x_i>0, y_i>0}, U = {x_i>0, y_i=0}, D = {x_i=0, y_i>0})
- Empirical p-value against 1000 degree-preserving random networks (e.g., 0.0230, 0.0180, 0.0320 for subpopulations 1–3)
- dense_subnetwork_results.tsv

## How to apply

Build the directed weighted adjacency matrix W_ij of TF-target scores, then solve the quadratic program max Σ_ij W_ij·x_i·y_j subject to the L1-type normalization constraints Σ_i x_i^β = 1, Σ_j y_j^β = 1, x_i, y_j ≥ 0 with β = 1 (the paper's choice, which promotes sparse solutions), e.g., with scipy.optimize (SLSQP or projected gradient). Define the dense subnetwork as nodes with nonzero x_i or nonzero y_i, and partition it into upstream U = {x_i > 0, y_i = 0}, core C = {x_i > 0, y_i > 0} (TFs that densely cross-regulate each other), and downstream D = {x_i = 0, y_i > 0}. Assess significance by generating 1000 degree-preserving random networks via edge-source/target switching (1000 switches per network, so each node retains in- and out-degree), re-extracting the dense subnetwork from each, and computing an empirical p-value for the real network's optimal objective. Accept the subnetwork when the p-value is significant (the paper reports p = 0.0230, 0.0180, and 0.0320 for subpopulations 1–3) and write results to dense_subnetwork_results.tsv.

## Related tools

- **DC3** (Produces the subpopulation-specific expression, accessibility, and enhancer-promoter loop profiles from which the subpopulation-specific regulatory networks (input W_ij) are constructed) — github.com/SUwonglab/DC3
- **scipy** (Solves the quadratic program (max Σ W_ij x_i y_j under L1-type constraints) via SLSQP or projected gradient and supports the randomization/statistics steps)

## Examples

```
from scipy.optimize import minimize; res = minimize(lambda z: -float(W @ x.T @ y), z0, method='SLSQP', constraints=[{'type':'eq','fun': lambda z: z[:n].sum()-1}, {'type':'eq','fun': lambda z: z[n:].sum()-1}])
```

## Evaluation signals

- All selected nodes have strictly nonzero x_i or y_i from the β = 1 constrained optimization, and constraints Σ x_i^β = 1, Σ y_j^β = 1 are satisfied within solver tolerance
- Every randomization iteration preserves each node's in-degree and out-degree (exactly 1000 switch operations per network, 1000 networks)
- Empirical p-value of the real network's objective value is significant (paper values: 0.0230, 0.0180, 0.0320 for subpopulations 1–3)
- Core partition is nonempty and core TFs show dense mutual cross-regulation within C = {x_i > 0, y_i > 0}
- Output file dense_subnetwork_results.tsv is written with the partition and p-values

## Limitations

- Density significance rests on a degree-preserving edge-switch null model; if network construction biases edge placement beyond degree structure, the p-value may be mis-calibrated
- The β = 1 choice favors sparse solutions; alternative β choices change the extracted subnetwork and are not explored in the provided context
- p-values near 0.02–0.03 for the three subpopulations indicate borderline significance at α = 0.05 for some subpopulations

## Evidence

- [results] Core/upstream/downstream partition rationale: [evidence span withheld — anchor status: not_found]
- [intro] Motivation for deriving core networks from DC3 output: "we illustrate the value of results from DC3 by using them to derive the core regulatory network and their downstream effectors in each of the subpopulation"
- [discussion] DC3 profiles as the foundation for core subnetwork extraction: "the accessibility, expression and loop profiles inferred by DC3 can serve as a foundation for further analyses of the regulatory systems, such as the extraction of core subnetworks"
