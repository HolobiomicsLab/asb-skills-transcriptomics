---
name: markov-chain-trajectory-simulation
description: Use when scRNA-seq data has reliable spliced/unspliced velocity estimates
  and the goal is trajectory inference across linear, circular, bifurcated, tree-like,
  or convergent topologies, especially where static-expression methods fail (e.g.,
  cyclical cell-cycle patterns).
when_to_use_negative:
- When root and terminal cell states are not clearly identifiable — the authors do
  not recommend performing trajectory inference with Cytopath in that case
- When the underlying RNA velocity estimation itself is faulty or unrepresentative
  (e.g., time-dependent degradation rates, single-regime assumption violations), as
  spurious root/terminal states will be inferred
- When the input lacks spliced/unspliced layers and no velocity field can be estimated
  with scvelo
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2229
tools:
- name: Cytopath
  role: Implements the Markov chain simulation, DTW alignment, and consensus trajectory
    construction
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Preprocesses spliced/unspliced data and estimates the RNA velocity field feeding
    the transition probability matrix
- name: fastdtw
  role: Dynamic time warping alignment of simulations terminating at a common terminal
    state
- name: hausdorff
  role: Pairwise Euclidean Hausdorff distances between simulations for sub-trajectory
    clustering
- name: scikit-learn
  role: Louvain clustering on Hausdorff distances to group sub-trajectories
- name: velocyto
  role: Produces spliced/unspliced/ambiguous count matrices from raw reads (e.g.,
    Burns et al. inner ear data)
  repo: https://github.com/velocyto-team/velocyto-notebooks
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_007
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:33ef011779d8cd9953087a56af23c01ddde04c9fc4c815eb43dbb54cb2c71507
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Angle
  - Cytopath
  - PAGA
  - fastdtw
  - hausdorff
  - numpy
  - pandas
  - scikit-learn
  - scipy
  - scvelo
  - velocyto
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# markov-chain-trajectory-simulation

## Summary

Cytopath's simulation-based trajectory inference engine: define a Markov chain model from scvelo-derived RNA velocity transition probabilities, simulate an ensemble of differentiation trajectories initialized at root states, and collapse simulations into consensus trajectories via dynamic time warping and per-step averaging.

## When to use

Use when scRNA-seq data has reliable spliced/unspliced velocity estimates and the goal is trajectory inference across linear, circular, bifurcated, tree-like, or convergent topologies, especially where static-expression methods fail (e.g., cyclical cell-cycle patterns).

## When NOT to use

- When root and terminal cell states are not clearly identifiable — the authors do not recommend performing trajectory inference with Cytopath in that case
- When the underlying RNA velocity estimation itself is faulty or unrepresentative (e.g., time-dependent degradation rates, single-regime assumption violations), as spurious root/terminal states will be inferred
- When the input lacks spliced/unspliced layers and no velocity field can be estimated with scvelo

## Inputs

- AnnData object with spliced/unspliced count matrices processed by scvelo (velocity fields)
- Cell-to-cell transition probability matrix derived from the RNA velocity field
- Optional root/terminal state labels or cell clustering (otherwise Louvain clustering computed internally)

## Outputs

- Ensemble of simulated cell state trajectories per terminal region
- Consensus trajectory (per-step averaged consensus states) per terminal region, e.g. consensus_trajectories.csv
- Per-cell pseudotime from alignment-score-based step assignment
- Terminal-region sub-trajectory clusters from Louvain clustering on Hausdorff distances

## How to apply

First estimate RNA velocity with scvelo (>=0.1.25) on the spliced/unspliced AnnData and build the cell-to-cell transition probability matrix; identify root cells via a Markov random-walk on the transition probability matrix (as described by La Manno et al.) or supplied root labels, with terminal regions taken from a supplied clustering or computed internally using Louvain. Initialize Markov-chain simulations at root states and sample consecutive cell state transitions according to transition matrix probabilities, producing an ensemble of simulated cell state sequences (count user-defined or estimated unsupervised, with no topological constraints imposed). For simulations terminating at a common terminal state, compute pairwise Euclidean Hausdorff distances (hausdorff package), group them by Louvain clustering on those distances, then align each group with fastdtw dynamic time warping. Average cell states at every step of the aligned ensemble to obtain the consensus state at each trajectory step. Finally, assign cells to trajectory steps via an alignment score evaluating the similarity of each cell's static and velocity profile with each trajectory step, yielding pseudotime.

## Related tools

- **Cytopath** (Implements the Markov chain simulation, DTW alignment, and consensus trajectory construction) — https://github.com/aron0093/cytopath
- **scvelo** (Preprocesses spliced/unspliced data and estimates the RNA velocity field feeding the transition probability matrix)
- **fastdtw** (Dynamic time warping alignment of simulations terminating at a common terminal state)
- **hausdorff** (Pairwise Euclidean Hausdorff distances between simulations for sub-trajectory clustering)
- **scikit-learn** (Louvain clustering on Hausdorff distances to group sub-trajectories)
- **velocyto** (Produces spliced/unspliced/ambiguous count matrices from raw reads (e.g., Burns et al. inner ear data)) — https://github.com/velocyto-team/velocyto-notebooks

## Examples

```
import cytopath; cytopath.terminal_states(adata), cytopath.simulate(adata), cytopath.pseudotime(adata)  # after scvelo velocity estimation on the AnnData
```

## Evaluation signals

- Consensus trajectories are written per terminal region with per-step averaged cell states (consensus_trajectories.csv) — check the file structure and per-terminal grouping
- Simulated trajectories terminate only in identified terminal regions; on the U2OS cell cycle data, the reconstructed trajectory should form a single uninterrupted circle with no spurious endpoints
- Inferred pseudotime should show high Spearman correlation with known cell type ordering (hippocampus/inner ear) that is robust across 10 independent runs
- Alignment scores of static and velocity profiles should assign each cell to a unique trajectory step; verify coverage of cells across steps
- Root/terminal probability thresholds within [0.85, 1] should not materially change inferred pseudotime (reported robustness range)

## Limitations

- Trajectory inference is not meaningful when the underlying RNA velocity estimation is faulty; incorrect velocity leads to spurious root/terminal states (seen in the erythroid gastrulation dataset and a simulated dataset with time-dependent degradation rates)
- RNA velocity simplifying assumptions (time-invariant transcription/splicing/degradation rates, single regulatory regime per gene) can make velocity unrepresentative of true dynamics
- Pseudotime robustness to root/terminal probability thresholds was only reported as 'data not shown' for the range [0.85, 1]
- On the hepatocyte zonation dataset, root cell probability appeared fuzzy and distributed without a discernible pattern, yielding spurious directionality

## Evidence

- [abstract] Cytopath performs this task by defining a Markov chain model, simulating an ensemble of possible differentiation trajectories, and constructing a consensus trajectory.: "Cytopath performs this task by defining a Markov chain model, simulating an ensemble of possible differentiation trajectories, and constructing a consensus trajectory."
- [results] Markov sampling of consecutive cell state transitions is performed based on the probabilities of the transition matrix, resulting in an ensemble of simulated cell state sequences.: "Markov sampling of consecutive cell state transitions is performed based on the probabilities of the transition matrix, resulting in an ensemble of simulated cell state sequences."
- [results] simulations that terminate at a common terminal state are aligned using dynamic time warping: "simulations that terminate at a common terminal state are aligned using dynamic time warping"
- [results] Cell states at every step of the ensemble of aligned simulations are averaged, and the average value is considered the consensus state of the trajectory at the particular step: "Cell states at every step of the ensemble of aligned simulations are averaged, and the average value is considered the consensus state of the trajectory at the particular step"
- [readme] Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model; simulation of this model yields an ensemble of possible differentiation trajectories: "Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model; simulation of this model yields an ensemble of possible differentiation trajectories"
