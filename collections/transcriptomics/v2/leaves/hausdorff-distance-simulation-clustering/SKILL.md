---
name: hausdorff-distance-simulation-clustering
description: Use when an ensemble of simulated cell-state sequences from the Cytopath
  Markov sampler must be partitioned into sub-trajectory clusters (per terminal region)
  before dynamic time warping alignment and consensus averaging.
when_to_use_negative:
- The underlying RNA velocity estimation is faulty — the article notes spurious root/terminal
  inference (e.g., the erythroid gastrulation dataset) makes trajectory inference
  not meaningful
- Root and terminal cell states are not clearly identifiable — the authors explicitly
  do not recommend performing trajectory inference with Cytopath in this case
- No ensemble of simulated trajectories or transition probability matrix exists yet
  (e.g., raw AnnData without scvelo velocity processing) — the clustering operates
  on simulations, not raw expression data
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cytopath
  role: Method that performs Markov sampling of trajectories and within which the
    Hausdorff-distance clustering step groups simulations per terminal region
  repo: https://github.com/aron0093/cytopath
- name: hausdorff
  role: Python package used to compute pairwise Euclidean Hausdorff distances between
    simulated trajectories terminating at the same terminal state
- name: scikit-learn
  role: Provides utilities (with numpy) for Louvain clustering on the Hausdorff distance
    matrix to obtain sub-trajectory clusters
- name: scvelo
  role: Produces the spliced/unspliced AnnData and RNA velocity field from which the
    cell-to-cell transition probability matrix is constructed
- name: fastdtw
  role: Performs dynamic time warping alignment of simulations within each Hausdorff-derived
    cluster before consensus averaging
- name: numpy
  role: Distance handling utilities supporting the Louvain clustering on Hausdorff
    distances
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:07f221b0ab70873c774a0e8e76b101caae9d8c6732ce4ceabf7d628591a857c4
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
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# hausdorff-distance-simulation-clustering

## Summary

Groups Cytopath's simulated Markov-chain differentiation trajectories terminating at the same terminal state by computing pairwise Euclidean Hausdorff distances between simulations and applying Louvain clustering, so that only simulations within a cluster are aligned and averaged into a consensus trajectory.

## When to use

Use when an ensemble of simulated cell-state sequences from the Cytopath Markov sampler must be partitioned into sub-trajectory clusters (per terminal region) before dynamic time warping alignment and consensus averaging.

## When NOT to use

- The underlying RNA velocity estimation is faulty — the article notes spurious root/terminal inference (e.g., the erythroid gastrulation dataset) makes trajectory inference not meaningful
- Root and terminal cell states are not clearly identifiable — the authors explicitly do not recommend performing trajectory inference with Cytopath in this case
- No ensemble of simulated trajectories or transition probability matrix exists yet (e.g., raw AnnData without scvelo velocity processing) — the clustering operates on simulations, not raw expression data

## Inputs

- Ensemble of simulated cell-state sequences (Markov-chain samples from the scvelo-derived cell-to-cell transition probability matrix)
- Terminal-state assignment per simulation (supplied labels or internally computed Louvain clustering)
- Spliced/unspliced AnnData processed with scvelo (upstream context)

## Outputs

- Pairwise Hausdorff distance matrix between simulations within each terminal region
- Louvain cluster (sub-trajectory) assignments of simulations per terminal region
- Clustered simulation sets passed to fastdtw alignment for consensus trajectory construction (consensus_trajectories.csv downstream)

## How to apply

After Markov sampling of consecutive cell state transitions from the cell-to-cell transition probability matrix (initialized at root cells), identify simulations that terminate at a common terminal region (determined from supplied labels or internal Louvain clustering). For each terminal region, compute pairwise Euclidean Hausdorff distances between the simulated cell-state sequences using the hausdorff package. Group the simulations by applying Louvain clustering (via scikit-learn/numpy utilities) to the Hausdorff distance matrix, yielding sub-trajectory clusters within each terminal region. Only simulations within the same cluster proceed to fastdtw dynamic time warping alignment, after which cell states at every step are averaged to form the consensus trajectory written to consensus_trajectories.csv. No topological constraints are imposed; the number of trajectories can be user-defined or estimated unsupervised.

## Related tools

- **Cytopath** (Method that performs Markov sampling of trajectories and within which the Hausdorff-distance clustering step groups simulations per terminal region) — https://github.com/aron0093/cytopath
- **hausdorff** (Python package used to compute pairwise Euclidean Hausdorff distances between simulated trajectories terminating at the same terminal state)
- **scikit-learn** (Provides utilities (with numpy) for Louvain clustering on the Hausdorff distance matrix to obtain sub-trajectory clusters)
- **scvelo** (Produces the spliced/unspliced AnnData and RNA velocity field from which the cell-to-cell transition probability matrix is constructed)
- **fastdtw** (Performs dynamic time warping alignment of simulations within each Hausdorff-derived cluster before consensus averaging)
- **numpy** (Distance handling utilities supporting the Louvain clustering on Hausdorff distances)

## Examples

```
from hausdorff import hausdorff_distance; d = hausdorff_distance(sim_a, sim_b, distance='euclidean')
```

## Evaluation signals

- Simulations are only clustered against others terminating at the same terminal region — the Hausdorff distance matrix must be computed per terminal region, not globally
- Every simulated trajectory must receive exactly one Louvain cluster assignment within its terminal region (a valid partition of the ensemble)
- Each resulting cluster should contain trajectories whose Hausdorff distances are small relative to trajectories in other clusters of the same terminal region
- Downstream check: every cluster yields an aligned consensus trajectory step sequence; per-terminal consensus trajectories must appear in consensus_trajectories.csv
- Results should be robust across multiple runs, consistent with the reported robustness of Cytopath pseudotime across repeated runs

## Limitations

- Robustness of pseudotime to root and terminal probability thresholds was reported only as 'data not shown' for a range of [0.85, 1]
- Trajectory inference (and hence clustering-based consensus) is not meaningful when the underlying RNA velocity estimation itself is faulty, e.g., genes with time-dependent degradation rates leading to spurious root/terminal states
- Root cell probability may be fuzzy and patternless (hepatocyte zonation dataset), undermining terminal-region grouping of simulations

## Evidence

- [results] Ensemble of simulated trajectories clustered per terminal region: "In the first step, Markov sampling of consecutive cell state transitions is performed based on the probabilities of the transition matrix, resulting in an ensemble of simulated cell state sequences."
- [results] Alignment per terminal state: "simulations that terminate at a common terminal state are aligned using dynamic time warping"
- [results] Consensus averaging: "Cell states at every step of the ensemble of aligned simulations are averaged, and the average value is considered the consensus state of the trajectory at the particular step"
- [readme] Unsupervised trajectory count estimation: "The number of trajectories to be inferred can either be defined or estimated in an unsupervised fashion."
- [readme] No topological constraints: "No topological constraints (e.g. a tree structure) are placed on the inference as each trajectory is modelled independently."
