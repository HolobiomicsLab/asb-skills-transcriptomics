---
name: terminal-region-clustering-assignment
description: Use when after thresholding root/terminal probabilities (Pr, Pt) at 0.99
  on the La Manno-style stationary states, the selected terminal cells must be partitioned
  into terminal regions to initialize trajectory simulations.
when_to_use_negative:
- Root and terminal cell states are not clearly identifiable from the velocity field
  — the authors state they do not recommend trajectory inference in that case (e.g.
  fuzzy root-cell probability distributed without a discernible pattern in the hepatocyte
  zonation data).
- The underlying RNA velocity estimation itself is faulty (e.g. time-dependent degradation
  rates in the scvelo-simulated data), which led to spurious root/terminal state inference,
  making the resulting regions meaningless.
- Supervised root/terminal state annotations are already provided — use them directly
  instead of re-clustering.
tools:
- name: Cytopath
  role: 'Performs terminal-region assignment: selects terminal cells at Pt ≥ 0.99
    from stationary-state estimation and groups them via the supplied clustering or
    internal Louvain clustering before Markov-chain simulation.'
  repo: github.com/aron0093/cytopath
- name: scvelo
  role: Provides the velocity-processed AnnData and velocity field from which the
    transition probability matrix and stationary-state probabilities are computed.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:8817dde27915c3f19256de676405dd854d8a9704ae9bb37c19c6007e2c214a89
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytopath
  - scvelo
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# terminal-region-clustering-assignment

## Summary

Groups the terminal cells selected by scvelo/Cytopath stationary-state estimation (terminal probability Pt ≥ 0.99) into terminal regions, using a user-supplied clustering when available or an internally computed Louvain clustering, so each terminal region seeds one set of Cytopath Markov-chain trajectory simulations.

## When to use

Use when, after thresholding root/terminal probabilities (Pr, Pt) at 0.99 on the La Manno-style stationary states, the selected terminal cells must be partitioned into terminal regions to initialize trajectory simulations.

## When NOT to use

- Root and terminal cell states are not clearly identifiable from the velocity field — the authors state they do not recommend trajectory inference in that case (e.g. fuzzy root-cell probability distributed without a discernible pattern in the hepatocyte zonation data).
- The underlying RNA velocity estimation itself is faulty (e.g. time-dependent degradation rates in the scvelo-simulated data), which led to spurious root/terminal state inference, making the resulting regions meaningless.
- Supervised root/terminal state annotations are already provided — use them directly instead of re-clustering.

## Inputs

- scvelo-processed AnnData object with spliced/unspliced counts and RNA velocity field
- Cell-to-cell transition probability matrix from the Markov random-walk model
- Terminal cell probability values (Pt) with 0.99 threshold selection
- Optional pre-computed clustering or user-supplied terminal state annotations
- Optional root cell probabilities (Pr) and root cell set

## Outputs

- stationary_states.csv with cell identifier, Pr, Pt, and assigned state fields
- Terminal cell set (Pt ≥ 0.99)
- Terminal regions grouping the selected terminal cells
- Terminal regions used to seed Cytopath Markov-chain trajectory simulations

## How to apply

First derive terminal cells from the absorbing-state Markov random-walk model built on the cell-to-cell transition probability matrix (La Manno et al. formulation), selecting cells with Pt ≥ 0.99 unless user-supplied terminal annotations exist (supervised mode). If a clustering (e.g. a cell-type annotation in the AnnData object) is supplied, use it directly to group terminal cells into terminal regions; otherwise compute a clustering internally with Louvain. Each terminal region then serves as a common terminal state so that Cytopath simulations terminating at the same region can be aligned by dynamic time warping to build the consensus trajectory. Write the terminal cell set, terminal regions, and raw Pr/Pt values to stationary_states.csv for downstream validation and simulation setup.

## Related tools

- **Cytopath** (Performs terminal-region assignment: selects terminal cells at Pt ≥ 0.99 from stationary-state estimation and groups them via the supplied clustering or internal Louvain clustering before Markov-chain simulation.) — github.com/aron0093/cytopath
- **scvelo** (Provides the velocity-processed AnnData and velocity field from which the transition probability matrix and stationary-state probabilities are computed.)

## Evaluation signals

- stationary_states.csv exists and contains fields for cell identifier, Pr, Pt, and assigned state.
- All probability values in stationary_states.csv lie within [0, 1].
- Selected terminal cells satisfy the 0.99 Pt threshold.
- Every terminal region is non-empty and every selected terminal cell is assigned to exactly one terminal region (either via the supplied clustering or the internal Louvain clustering).
- The number of inferred trajectories/terminal regions matches the number of distinct terminal regions or the user-defined trajectory count.

## Limitations

- Cytopath pseudotime is robust to root/terminal probability thresholds only within a range of [0.85, 1], and that robustness was reported only as 'data not shown'.
- Spurious terminal states can be inferred when velocity estimation is wrong — e.g. endpoints spuriously inferred mid-process in the erythroid gastrulation dataset.
- When no clustering is supplied, an internal Louvain clustering is computed, so terminal-region granularity depends on Louvain resolution rather than biological annotation.

## Evidence

- [results] Computation of internal Louvain clustering when no pre-computed clustering is supplied to determine terminal regions: "otherwise, a clustering is computed internally using Louvain."
- [results] Root and terminal states derived from the transition probability matrix: "Root and terminal states can be derived from a Markov random-walk model utilizing the transition probability matrix itself (Figure 1A.3), as described by La Manno et al.,1"
- [readme] Supervised vs unsupervised terminal state handling in Cytopath: "Cytopath can infer trajectories with or without root/terminal state supervision."
- [discussion] Do not run trajectory inference when terminal states are unclear: "We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable."
