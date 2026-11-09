---
name: consensus-trajectory-averaging
description: Use when you have an ensemble of simulated cell state sequences (e.g.,
  from Markov sampling of an RNA-velocity transition matrix) and need a single consensus
  trajectory per terminal region.
when_to_use_negative:
- The underlying RNA velocity estimation itself is faulty or root/terminal cell states
  are not clearly identifiable (e.g., fuzzy root cell probability with no discernible
  pattern, as in the hepatocyte zonation dataset)
- Root/terminal state inference is spurious due to violated scvelo assumptions, e.g.,
  features with time-dependent degradation rates as in the simulated erythroid gastrulation
  case
- You need a single global trajectory topology upfront — Cytopath imposes no topological
  constraints, so this skill is inappropriate when a rigid tree/cycle model must be
  enforced before simulation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Cytopath
  role: Provides the simulation engine, DTW alignment, and consensus-state averaging
    workflow; writes consensus_trajectories.csv
  repo: github.com/aron0093/cytopath
- name: fastdtw
  role: Performs dynamic time warping alignment of simulations terminating at a common
    terminal state
  repo: github.com/aron0093/cytopath
- name: hausdorff
  role: Computes pairwise Euclidean Hausdorff distances between simulated trajectories
    for clustering
  repo: github.com/aron0093/cytopath
- name: scikit-learn
  role: Louvain/spectral clustering utilities to group simulations by Hausdorff distance
    within terminal regions
  repo: github.com/aron0093/cytopath
- name: scvelo
  role: Processes spliced/unspliced AnnData and supplies the RNA velocity field from
    which the transition probability matrix is built
  repo: https://cytopath.readthedocs.io/en/latest/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:aa717bc2f66dbcddf71e7aacdac9ca6e3e32124b378223586f56a00092c5279c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cytopath
  - fastdtw
  - hausdorff
  - numpy
  - scikit-learn
  - scvelo
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# consensus-trajectory-averaging

## Summary

Converts an ensemble of Markov-sampled differentiation trajectories from the Cytopath simulation engine into a consensus lineage path by aligning simulations that terminate at a common terminal state with dynamic time warping and averaging cell states at every aligned step.

## When to use

Use when you have an ensemble of simulated cell state sequences (e.g., from Markov sampling of an RNA-velocity transition matrix) and need a single consensus trajectory per terminal region.

## When NOT to use

- The underlying RNA velocity estimation itself is faulty or root/terminal cell states are not clearly identifiable (e.g., fuzzy root cell probability with no discernible pattern, as in the hepatocyte zonation dataset)
- Root/terminal state inference is spurious due to violated scvelo assumptions, e.g., features with time-dependent degradation rates as in the simulated erythroid gastrulation case
- You need a single global trajectory topology upfront — Cytopath imposes no topological constraints, so this skill is inappropriate when a rigid tree/cycle model must be enforced before simulation

## Inputs

- Ensemble of simulated cell state sequences (Markov chains sampled from the cell-to-cell transition probability matrix derived from scvelo RNA velocity)
- Terminal region assignment per simulation (from supplied labels or internal Louvain clustering)
- Pairwise Hausdorff distance matrix between simulations within each terminal region
- Spliced/unspliced AnnData processed with scvelo (for static and velocity profiles)

## Outputs

- Consensus trajectory per terminal region (averaged cell states at each aligned step), written to consensus_trajectories.csv
- Sub-trajectory clusters per terminal region (Louvain clusters on Hausdorff distances)
- Cell-to-trajectory-step assignments based on static/velocity alignment scores

## How to apply

First group simulations that terminate at the same terminal region, optionally clustering them by pairwise Euclidean Hausdorff distances (computed with the hausdorff package) and Louvain clustering to obtain sub-trajectory clusters. Align the simulations within each terminal-region cluster using fastdtw dynamic time warping so that steps of different simulated sequences become comparable. Then average the cell states at every step of the aligned ensemble; the average value is taken as the consensus state of the trajectory at that step, and per-terminal-region consensus trajectories are written to consensus_trajectories.csv. No topological constraints are imposed — each trajectory is modelled independently, so the number of simulations can be user-defined or estimated unsupervised. Finally, cells can be assigned to trajectory steps via an alignment score comparing each cell's static and velocity profiles with each consensus trajectory step.

## Related tools

- **Cytopath** (Provides the simulation engine, DTW alignment, and consensus-state averaging workflow; writes consensus_trajectories.csv) — github.com/aron0093/cytopath
- **fastdtw** (Performs dynamic time warping alignment of simulations terminating at a common terminal state) — github.com/aron0093/cytopath
- **hausdorff** (Computes pairwise Euclidean Hausdorff distances between simulated trajectories for clustering) — github.com/aron0093/cytopath
- **scikit-learn** (Louvain/spectral clustering utilities to group simulations by Hausdorff distance within terminal regions) — github.com/aron0093/cytopath
- **scvelo** (Processes spliced/unspliced AnnData and supplies the RNA velocity field from which the transition probability matrix is built) — https://cytopath.readthedocs.io/en/latest/

## Examples

```
import cytopath; cytopath.cytopath(adata, root_cells=root, terminal_states=terminal, n_trajectories=1000, method='cellrank')
```

## Evaluation signals

- Consensus trajectories for every terminal region are present in consensus_trajectories.csv, one per identified terminal state (e.g., alpha, beta, gamma, delta in pancreatic endocrinogenesis)
- DTW alignment succeeds only among simulations sharing a common terminal state — verify no cross-terminal alignments were performed
- Consensus pseudotime shows high Spearman correlation with known cell type ordering per lineage and is robust across multiple runs (as in the 18,140-cell developing mouse hippocampus dataset)
- Hausdorff-distance Louvain clustering yields coherent sub-trajectory clusters with no topological constraints imposed on individual simulations
- Number of inferred trajectories matches a user-supplied count or an unsupervised estimate, and each simulated sequence is modelled independently

## Limitations

- Pseudotime robustness to root/terminal probability thresholds was only reported as 'data not shown' (robust within [0.85, 1])
- Trajectory inference is not meaningful when the underlying RNA velocity estimation is faulty — e.g., spurious root points inferred toward the terminus in the erythroid gastrulation dataset
- Correctness of RNA velocity cannot be verified in every scenario, and the authors do not recommend this approach when root and terminal cell states are not clearly identifiable

## Evidence

- [results] Markov sampling initialized at root states with DTW-based consensus construction: "simulations that terminate at a common terminal state are aligned using dynamic time warping"
- [results] Consensus state computed by averaging aligned simulation steps: "Cell states at every step of the ensemble of aligned simulations are averaged, and the average value is considered the consensus state of the trajectory at the particular step"
- [abstract] Markov chain model and ensemble simulation define the method: "Cytopath performs this task by defining a Markov chain model, simulating an ensemble of possible differentiation trajectories, and constructing a consensus trajectory."
- [readme] No topological constraints; trajectory count user-defined or unsupervised: "No topological constraints (e.g. a tree structure) are placed on the inference as each trajectory is modelled independently. The number of trajectories to be inferred can either be defined or"
- [discussion] Not meaningful when velocity estimation is faulty: "We do not expect that trajectory inference with Cytopath is meaningful when the underlying RNA velocity estimation itself is faulty."
