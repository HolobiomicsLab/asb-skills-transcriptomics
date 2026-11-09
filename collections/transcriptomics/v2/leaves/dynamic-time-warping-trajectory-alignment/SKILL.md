---
name: dynamic-time-warping-trajectory-alignment
description: Use when an ensemble of simulated cell state transitions (e.g. from Cytopath
  Markov sampling) must be converted into per-terminal-region consensus trajectories,
  or when simulations of unequal length need temporal alignment before averaging.
when_to_use_negative:
- When the underlying RNA velocity estimation is faulty — the authors do not expect
  trajectory inference with Cytopath to be meaningful if velocity itself is unrepresentative
  of true transcriptional dynamics (e.g. spurious root/terminal states in the erythroid
  gastrulation dataset)
- When root and terminal cell states are not clearly identifiable — the authors explicitly
  do not recommend performing trajectory inference with Cytopath in that situation
- When simulations do not share a common terminal state, since DTW alignment in this
  workflow is applied only within groups of simulations terminating at a common terminal
  state
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cytopath
  role: Generates the Markov-sampled ensemble of simulated differentiation trajectories
    and provides the alignment/consensus machinery
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Estimates spliced/unspliced RNA velocity that defines the transition probability
    matrix driving Markov sampling
- name: fastdtw
  role: Performs dynamic time warping alignment of simulations within each terminal-region
    cluster
- name: hausdorff
  role: Computes pairwise Euclidean Hausdorff distances between simulations terminating
    at the same terminal region
- name: scikit-learn
  role: Louvain/numpy utilities for clustering simulations on Hausdorff distances
    into sub-trajectory groups
- name: numpy
  role: Distance handling and averaging of aligned cell states to obtain consensus
    states
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:7f676e4e20312274560428061afc6f0c171ea4d5dfe88e622813b552fc23f714
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Cytopath
  - fastdtw
  - hausdorff
  - numpy
  - scikit-learn
  - scvelo
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# dynamic-time-warping-trajectory-alignment

## Summary

Aligns Cytopath's ensemble of Markov-sampled cell state sequences that terminate at a common terminal state using dynamic time warping (fastdtw), then averages cell states at every step of the aligned simulations to construct a consensus trajectory.

## When to use

Use when an ensemble of simulated cell state transitions (e.g. from Cytopath Markov sampling) must be converted into per-terminal-region consensus trajectories, or when simulations of unequal length need temporal alignment before averaging.

## When NOT to use

- When the underlying RNA velocity estimation is faulty — the authors do not expect trajectory inference with Cytopath to be meaningful if velocity itself is unrepresentative of true transcriptional dynamics (e.g. spurious root/terminal states in the erythroid gastrulation dataset)
- When root and terminal cell states are not clearly identifiable — the authors explicitly do not recommend performing trajectory inference with Cytopath in that situation
- When simulations do not share a common terminal state, since DTW alignment in this workflow is applied only within groups of simulations terminating at a common terminal state

## Inputs

- AnnData object (h5ad) with spliced/unspliced counts processed with scvelo for RNA velocity
- Cell-to-cell transition probability matrix constructed from the RNA velocity field
- Ensemble of simulated cell state sequences from Cytopath Markov sampling, initialized at root cells
- Terminal state supervision or Louvain clustering defining terminal regions (e.g. alpha, beta, gamma, delta endocrine states)

## Outputs

- Aligned simulations grouped by terminal region and sub-trajectory cluster (Louvain clusters on Hausdorff distances)
- Consensus cell state at each step of each (sub-)trajectory, written per-terminal-region to consensus_trajectories.csv
- Per-step consensus expression profiles usable for downstream cell-to-step assignment via the static+velocity alignment score

## How to apply

After Cytopath performs Markov sampling of consecutive cell state transitions from the RNA velocity-derived transition probability matrix (initialized at root states), group simulations that terminate at the same terminal region (using supplied Louvain clustering or internally computed clustering). Compute pairwise Euclidean Hausdorff distances between simulations within each terminal region using the hausdorff package, then cluster the simulations with Louvain clustering on these distances to obtain sub-trajectory clusters. Align the simulations in each cluster with fastdtw dynamic time warping, which resolves differing simulation lengths and step timings. Average the cell states at every step of the aligned ensemble — this average is taken as the consensus state of the trajectory at that step — and write per-terminal-region consensus trajectories (e.g. to consensus_trajectories.csv). Rationale: consensus states across many independently sampled simulations are needed because no topological constraints are imposed on individual trajectories; DTW enables the averaging despite variable chain lengths.

## Related tools

- **Cytopath** (Generates the Markov-sampled ensemble of simulated differentiation trajectories and provides the alignment/consensus machinery) — https://github.com/aron0093/cytopath
- **scvelo** (Estimates spliced/unspliced RNA velocity that defines the transition probability matrix driving Markov sampling)
- **fastdtw** (Performs dynamic time warping alignment of simulations within each terminal-region cluster)
- **hausdorff** (Computes pairwise Euclidean Hausdorff distances between simulations terminating at the same terminal region)
- **scikit-learn** (Louvain/numpy utilities for clustering simulations on Hausdorff distances into sub-trajectory groups)
- **numpy** (Distance handling and averaging of aligned cell states to obtain consensus states)

## Examples

```
from fastdtw import fastdtw; from scipy.spatial.distance import euclidean; d, path = fastdtw(sim_a_states, sim_b_states, dist=euclidean)
```

## Evaluation signals

- Each simulation group submitted to DTW contains only simulations terminating at a common terminal state (verified via supplied or internal Louvain clustering)
- Consensus trajectories are produced per terminal region; on the pancreatic endocrinogenesis dataset the inferred consensus trajectories should terminate in the alpha and delta endocrine states as reported
- Cell states at every step of the aligned ensemble are averaged, and cells can be assigned to trajectory steps via the static+velocity alignment score — cells should map to steps consistent with their annotated lineage ordering
- Pseudotime/trajectory results should be robust across runs; root and terminal probability thresholds within the reported robust range [0.85, 1] should not change the consensus topology
- Output schema check: per-terminal-region consensus trajectories written to consensus_trajectories.csv with one consensus state per aligned step

## Limitations

- Dynamic time warping alignment is only meaningful for simulations sharing a terminal state; no topological constraints are placed on individual trajectories, so alignment quality depends on sensible terminal-region grouping
- Alignment/consensus quality is bounded by the transition probability matrix: spurious root/terminal inference (e.g. erythroid gastrulation, hepatocyte zonation with fuzzy root cell probability) propagates into consensus trajectories
- Robustness to root and terminal probability thresholds was reported only within [0.85, 1] and characterized as 'data not shown'

## Evidence

- [results] simulations that terminate at a common terminal state are aligned using dynamic time warping: "simulations that terminate at a common terminal state are aligned using dynamic time warping"
- [results] Cell states at every step of the ensemble of aligned simulations are averaged, and the average value is considered the consensus state of the trajectory at the particular step: "Cell states at every step of the ensemble of aligned simulations are averaged, and the average value is considered the consensus state of the trajectory at the particular step"
- [results] Alignment of simulations terminating at a common terminal state using dynamic time warping: [evidence span withheld — anchor status: not_found]
- [readme] The number of trajectories to be inferred can either be defined or estimated in an unsupervised fashion.: "The number of trajectories to be inferred can either be defined or estimated in an unsupervised fashion."
- [discussion] We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable.: "We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable."
- [discussion] We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown).: "We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown)."
