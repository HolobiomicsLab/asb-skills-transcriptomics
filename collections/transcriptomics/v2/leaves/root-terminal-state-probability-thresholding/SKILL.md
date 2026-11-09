---
name: root-terminal-state-probability-thresholding
description: Use when a scvelo-processed AnnData object with a velocity field must
  be decomposed into root and terminal cell sets before simulating Markov-chain differentiation
  trajectories with Cytopath, and no prior root/terminal annotations are supplied.
when_to_use_negative:
- The underlying RNA velocity estimation is faulty or unrepresentative — the article
  reports spurious root/terminal inference in the erythroid gastrulation dataset and
  a simulated dataset with time-dependent degradation rates in such cases
- Root and terminal cell states are not clearly identifiable (e.g. the hepatocyte
  zonation dataset showed fuzzy root cell probability with no discernible pattern)
- Reliable prior root/terminal annotations already exist — in supervised mode these
  should be used directly rather than thresholding computed probabilities
tools:
- name: Cytopath
  role: Computes root/terminal stationary-state probabilities from the transition
    probability matrix, applies the 0.99 threshold or accepts supervised annotations,
    clusters terminal cells, and writes stationary_states.csv
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Provides the upstream RNA velocity field and processed AnnData (spliced/unspliced
    counts) from which the transition probability matrix for absorbing-state estimation
    is built
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:39d07dbfe4033e36ebbbe5d3a7a2c0a9bfb5a7ab09829685cbb3703b187802fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Cytopath
  - scvelo
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# root-terminal-state-probability-thresholding

## Summary

Identifies root and terminal cell states for Cytopath's Markov-chain trajectory simulations by computing stationary-state probabilities (root probability Pr and terminal probability Pt) from an absorbing-state Markov random-walk on the RNA velocity-derived transition probability matrix, then thresholding at 0.99 to select the cell sets.

## When to use

Use when a scvelo-processed AnnData object with a velocity field must be decomposed into root and terminal cell sets before simulating Markov-chain differentiation trajectories with Cytopath, and no prior root/terminal annotations are supplied.

## When NOT to use

- The underlying RNA velocity estimation is faulty or unrepresentative — the article reports spurious root/terminal inference in the erythroid gastrulation dataset and a simulated dataset with time-dependent degradation rates in such cases
- Root and terminal cell states are not clearly identifiable (e.g. the hepatocyte zonation dataset showed fuzzy root cell probability with no discernible pattern)
- Reliable prior root/terminal annotations already exist — in supervised mode these should be used directly rather than thresholding computed probabilities

## Inputs

- scvelo-processed AnnData object (.h5ad) with spliced/unspliced counts and an RNA velocity field
- Cell-to-cell transition probability matrix built from the RNA velocity field (Markov random walk)
- Optional: user-supplied root/terminal state annotations (supervised mode)
- Optional: pre-computed clustering used to group terminal cells into terminal regions

## Outputs

- stationary_states.csv containing cell identifier, root probability (Pr), terminal probability (Pt), and assigned state
- Root cell set (Pr >= 0.99) used to initialize Markov simulations
- Terminal cell set (Pt >= 0.99) grouped into terminal regions

## How to apply

Load the scvelo-processed AnnData containing spliced/unspliced counts and the velocity field, and build the cell-to-cell transition probability matrix from RNA velocity. Compute root cell probability (Pr) and terminal cell probability (Pt) by absorbing-state estimation on this transition matrix, following the La Manno et al. stationary-state formulation. Threshold Pr and Pt at 0.99 to select root and terminal cell sets; alternatively, if user-supplied root/terminal annotations exist (supervised mode), use them directly instead of thresholding. Group the selected terminal cells into terminal regions using a supplied clustering or an internal Louvain clustering when none is provided. Write the root cell set, terminal cell set, terminal regions, and raw Pr/Pt values to stationary_states.csv as the handoff to Cytopath's trajectory simulations.

## Related tools

- **Cytopath** (Computes root/terminal stationary-state probabilities from the transition probability matrix, applies the 0.99 threshold or accepts supervised annotations, clusters terminal cells, and writes stationary_states.csv) — https://github.com/aron0093/cytopath
- **scvelo** (Provides the upstream RNA velocity field and processed AnnData (spliced/unspliced counts) from which the transition probability matrix for absorbing-state estimation is built)

## Evaluation signals

- stationary_states.csv exists and contains fields for cell identifier, Pr, Pt, and assigned state
- All probability values in stationary_states.csv lie within [0, 1]
- All selected root/terminal cells satisfy the 0.99 probability threshold
- Every selected terminal cell is assigned to a terminal region (via supplied clustering or internal Louvain clustering)
- Pseudotime downstream should be robust to threshold choice within [0.85, 1], per the article's sensitivity statement

## Limitations

- Robustness of Cytopath pseudotime to root and terminal probability thresholds within [0.85, 1] is reported only as 'data not shown'
- Incorrect RNA velocity estimation (e.g. time-invariant transcription/splicing/degradation rate assumptions) leads to spurious inference of root and terminal cell states, as seen for the erythroid gastrulation dataset
- Root cell probability may be fuzzy and distributed without a discernible pattern (hepatocyte zonation dataset), making the 0.99 threshold uninformative

## Evidence

- [results] Root and terminal states can be derived from a Markov random-walk model utilizing the transition probability matrix itself (Figure 1A.3), as described by La Manno et al.,1: "Root and terminal states can be derived from a Markov random-walk model utilizing the transition probability matrix itself (Figure 1A.3), as described by La Manno et al.,1"
- [results] otherwise, a clustering is computed internally using Louvain.: "otherwise, a clustering is computed internally using Louvain."
- [readme] Cytopath can infer trajectories with or without root/terminal state supervision.: "Cytopath can infer trajectories with or without root/terminal state supervision."
- [discussion] We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown).: "We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown)."
- [discussion] We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable.: "We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable."
