---
name: louvain-clustering-of-simulated-trajectories
description: Use when an ensemble of simulated differentiation trajectories (from
  RNA-velocity Markov sampling) must be partitioned into sub-trajectory groups within
  each terminal state before DTW alignment and consensus construction.
when_to_use_negative:
- The underlying RNA velocity estimation is faulty or root/terminal cell states are
  not clearly identifiable — the authors do not recommend performing trajectory inference
  with Cytopath in these cases (e.g. the hepatocyte zonation dataset showed fuzzy,
  patternless root-cell probabilities)
- Input is a pre-clustered downstream analysis problem unrelated to trajectory simulation
  ensembles — Louvain clustering here operates specifically on Hausdorff distances
  between simulated trajectories within a common terminal region
- Cell cycling with overlapping start/end regions when terminal supervision is absent
  — convergence/cycle handling requires the simulation-ensemble machinery, not plain
  cell clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Cytopath
  role: 'Hosts the full pipeline: generates the Markov-simulated trajectory ensemble
    and terminal regions on which Louvain clustering of simulations is performed'
  repo: https://github.com/aron0093/cytopath
- name: hausdorff
  role: Computes pairwise Euclidean Hausdorff distances between simulations terminating
    at the same terminal region, forming the input to clustering
- name: scikit-learn
  role: Used with numpy utilities for distance handling in the Louvain grouping of
    simulations
- name: numpy
  role: Distance matrix handling for the Hausdorff-based clustering of simulated trajectories
- name: fastdtw
  role: Aligns the simulations within each Louvain sub-trajectory cluster via dynamic
    time warping after clustering
- name: scvelo
  role: Produces the spliced/unspliced AnnData and velocity-derived transition probability
    matrix from which simulations are initialized
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:2998b65e2eddfbda257487891894ab81ec514f31b02a29a181df8a9eb1757fb4
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# louvain-clustering-of-simulated-trajectories

## Summary

Within the Cytopath pipeline, simulated Markov-chain cell-state sequences that terminate at a common terminal region are compared via pairwise Euclidean Hausdorff distances and grouped by Louvain clustering into sub-trajectory clusters prior to dynamic-time-warping alignment and consensus averaging.

## When to use

Use when an ensemble of simulated differentiation trajectories (from RNA-velocity Markov sampling) must be partitioned into sub-trajectory groups within each terminal state before DTW alignment and consensus construction.

## When NOT to use

- The underlying RNA velocity estimation is faulty or root/terminal cell states are not clearly identifiable — the authors do not recommend performing trajectory inference with Cytopath in these cases (e.g. the hepatocyte zonation dataset showed fuzzy, patternless root-cell probabilities)
- Input is a pre-clustered downstream analysis problem unrelated to trajectory simulation ensembles — Louvain clustering here operates specifically on Hausdorff distances between simulated trajectories within a common terminal region
- Cell cycling with overlapping start/end regions when terminal supervision is absent — convergence/cycle handling requires the simulation-ensemble machinery, not plain cell clustering

## Inputs

- Ensemble of simulated cell-state sequences (Markov-chain simulations from the Cytopath transition probability matrix, number user-defined or unsupervised)
- Terminal-state assignments (supplied labels or Markov random-walk estimates on the transition probability matrix)
- Spliced/unspliced AnnData processed with scvelo providing the cell-to-cell transition probability matrix

## Outputs

- Per-terminal-region sub-trajectory cluster assignments (Louvain clusters over simulations)
- Clustered simulations passed onward to fastdtw alignment
- Consensus trajectories written per terminal region to consensus_trajectories.csv

## How to apply

First identify terminal states, either from supplied root/terminal labels or from a Markov random-walk model on the cell-to-cell transition probability matrix (Louvain clustering is computed internally when no pre-computed clustering is supplied). For simulations that terminate at the same terminal region, compute pairwise Euclidean Hausdorff distances between the simulated cell-state sequences using the hausdorff package. Apply Louvain community detection on the Hausdorff distance graph (via scikit-learn/numpy utilities for distance handling) to obtain sub-trajectory clusters of similar simulations. Each cluster is then passed to fastdtw for dynamic-time-warping alignment, after which cell states are averaged per step to yield the consensus state. The number of simulations can be user-defined or estimated unsupervised, and no topological constraints are imposed on the inference.

## Related tools

- **Cytopath** (Hosts the full pipeline: generates the Markov-simulated trajectory ensemble and terminal regions on which Louvain clustering of simulations is performed) — https://github.com/aron0093/cytopath
- **hausdorff** (Computes pairwise Euclidean Hausdorff distances between simulations terminating at the same terminal region, forming the input to clustering)
- **scikit-learn** (Used with numpy utilities for distance handling in the Louvain grouping of simulations)
- **numpy** (Distance matrix handling for the Hausdorff-based clustering of simulated trajectories)
- **fastdtw** (Aligns the simulations within each Louvain sub-trajectory cluster via dynamic time warping after clustering)
- **scvelo** (Produces the spliced/unspliced AnnData and velocity-derived transition probability matrix from which simulations are initialized)

## Examples

```
from hausdorff import hausdorff_distance; import numpy as np; D = np.array([[hausdorff_distance(a, b) for b in sims] for a in sims]); import networkx as nx, community; labels = community.best_partition(nx.from_numpy_array(D))
```

## Evaluation signals

- Simulations grouped by Louvain must each terminate at a common terminal state — check cluster-to-terminal-region consistency before alignment
- The Hausdorff distance matrix must be symmetric, non-negative, and sized N_simulations x N_simulations for the terminal region being clustered
- Each resulting cluster should contain trajectory sequences of similar shape (low intra-cluster Hausdorff distance); clusters are only meaningful when enough simulations (user-defined or unsupervised estimate) were sampled
- Downstream DTW alignment must succeed for every cluster and the averaged aligned states must produce a per-step consensus trajectory in consensus_trajectories.csv
- Pseudotime from the reconstructed trajectories should show high Spearman correlation with known cell-type ordering per lineage and be robust across multiple runs

## Limitations

- Cytopath pseudotime robustness to root and terminal probability thresholds was only reported as 'data not shown' for the range [0.85, 1]
- Trajectory inference (and hence trajectory clustering) is not meaningful when the underlying RNA velocity estimation itself is faulty
- When no pre-computed clustering is supplied, Louvain clustering is computed internally, which may yield spurious terminal regions when root/terminal states are not clearly identifiable (e.g. spurious root points in the erythroid gastrulation dataset)

## Evidence

- [results] Internal Louvain clustering when no pre-computed clustering is supplied: "otherwise, a clustering is computed internally using Louvain."
- [results] DTW alignment within terminal groups: "simulations that terminate at a common terminal state are aligned using dynamic time warping"
- [readme] Simulation ensemble settings: "The number of trajectories to be inferred can either be defined or estimated in an unsupervised fashion."
- [abstract] Markov-chain ensemble to consensus pipeline: "Cytopath performs this task by defining a Markov chain model, simulating an ensemble of possible differentiation trajectories, and constructing a consensus trajectory."
