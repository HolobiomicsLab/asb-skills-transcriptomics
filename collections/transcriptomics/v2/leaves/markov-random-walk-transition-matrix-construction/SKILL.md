---
name: markov-random-walk-transition-matrix-construction
description: Use when an scvelo-processed AnnData object with spliced/unspliced counts
  and a velocity field is available and root/terminal cell states must be identified
  (or supplied in supervised mode) before running Cytopath trajectory simulations.
when_to_use_negative:
- When the underlying RNA velocity estimation is faulty — the authors do not expect
  Cytopath trajectory inference to be meaningful, and showed spurious root/terminal
  inference on the erythroid gastrulation dataset and a simulated dataset with time-dependent
  degradation rates
- When root and terminal cell states are not clearly identifiable — the authors explicitly
  do not recommend performing trajectory inference with Cytopath in this situation
  (e.g. hepatocyte zonation, where root cell probability appeared fuzzy with no discernible
  pattern)
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Cytopath
  role: Implements the Markov random-walk absorbing-state estimation of root/terminal
    states, thresholding, terminal-region grouping, and downstream Markov-chain trajectory
    simulation
  repo: github.com/aron0093/cytopath
- name: scvelo
  role: Supplies the RNA velocity field and processed AnnData object from which the
    transition probability matrix is built
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:ad8c156d33d852412c3f2a02c375b3d065d88b8f312c97ca8051eb71ecc2d35c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytopath
  - scvelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Markov random-walk transition matrix construction

## Summary

Constructs the cell-to-cell transition probability matrix from an RNA velocity field and performs absorbing-state (stationary-state) estimation following La Manno et al. to derive root cell probability (Pr) and terminal cell probability (Pt) for Cytopath's Markov-chain trajectory simulations.

## When to use

Use when an scvelo-processed AnnData object with spliced/unspliced counts and a velocity field is available and root/terminal cell states must be identified (or supplied in supervised mode) before running Cytopath trajectory simulations.

## When NOT to use

- When the underlying RNA velocity estimation is faulty — the authors do not expect Cytopath trajectory inference to be meaningful, and showed spurious root/terminal inference on the erythroid gastrulation dataset and a simulated dataset with time-dependent degradation rates
- When root and terminal cell states are not clearly identifiable — the authors explicitly do not recommend performing trajectory inference with Cytopath in this situation (e.g. hepatocyte zonation, where root cell probability appeared fuzzy with no discernible pattern)

## Inputs

- scvelo-processed AnnData object (.h5ad) containing spliced/unspliced counts and the RNA velocity field
- Optional: user-supplied root/terminal state annotations (supervised mode)
- Optional: pre-computed cell clustering for defining terminal regions

## Outputs

- stationary_states.csv with cell identifier, root probability (Pr), terminal probability (Pt), and assigned state fields
- Root cell set and terminal cell set (Pr, Pt >= 0.99)
- Terminal regions from supplied or internal Louvain clustering

## How to apply

Build the Markov random-walk transition probability matrix from the RNA velocity field of the scvelo-processed AnnData object, then compute root cell probability (Pr) and terminal cell probability (Pt) by absorbing-state estimation on this matrix following the La Manno et al. stationary-state formulation. Threshold Pr and Pt at 0.99 to select the root and terminal cell sets; if user-supplied root/terminal annotations are available, use them directly (supervised mode) instead of the unsupervised estimates. Group selected terminal cells into terminal regions using a supplied clustering or an internal Louvain clustering computed when no pre-computed clustering exists. Write the root cell set, terminal cell set, terminal regions, and raw Pr/Pt values to stationary_states.csv. The authors report that Cytopath pseudotime is robust to root and terminal probability thresholds within [0.85, 1], so the 0.99 cutoff is not highly sensitive.

## Related tools

- **Cytopath** (Implements the Markov random-walk absorbing-state estimation of root/terminal states, thresholding, terminal-region grouping, and downstream Markov-chain trajectory simulation) — github.com/aron0093/cytopath
- **scvelo** (Supplies the RNA velocity field and processed AnnData object from which the transition probability matrix is built)

## Evaluation signals

- stationary_states.csv exists and contains fields for cell identifier, Pr, Pt, and assigned state
- All probability values in stationary_states.csv lie within [0, 1]
- Selected root and terminal cells satisfy the 0.99 threshold (and remain stable for any threshold in the reported robust range [0.85, 1])
- Root/terminal assignments are consistent with the supplied or internal Louvain clustering used to define terminal regions
- In supervised mode, selected states match the user-supplied annotations exactly rather than the unsupervised estimates

## Limitations

- Pseudotime robustness to root/terminal probability thresholds within [0.85, 1] was reported only as 'data not shown'
- Incorrect RNA velocity estimation leads to spurious inference of root and terminal states (e.g. spurious root points toward the terminus in the erythroid gastrulation dataset; fuzzy, patternless root probability in the hepatocyte zonation dataset)
- Correctness of RNA velocity estimation cannot be verified in every scenario, even when independent marker-based information is available

## Evidence

- [results] Root and terminal states can be derived from a Markov random-walk model utilizing the transition probability matrix itself (Figure 1A.3), as described by La Manno et al.,1: "Root and terminal states can be derived from a Markov random-walk model utilizing the transition probability matrix itself (Figure 1A.3), as described by La Manno et al.,1"
- [results] otherwise, a clustering is computed internally using Louvain.: "otherwise, a clustering is computed internally using Louvain."
- [discussion] We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown).: "We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown)."
- [discussion] We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable.: "We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable."
- [readme] Cytopath can infer trajectories with or without root/terminal state supervision.: "Cytopath can infer trajectories with or without root/terminal state supervision."
