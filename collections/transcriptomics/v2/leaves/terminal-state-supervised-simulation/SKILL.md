---
name: terminal-state-supervised-simulation
description: Use when scRNA-seq data with RNA velocity should be resolved into directed
  differentiation trajectories whose endpoints are known terminal cell states (e.g.
  annotated alpha/beta/gamma/delta endocrine clusters), especially with cycling or
  convergent topologies.
when_to_use_negative:
- When root and terminal cell states are not clearly identifiable — the authors explicitly
  do not recommend performing Cytopath trajectory inference in this case.
- When the underlying RNA velocity estimation is faulty (e.g. time-dependent degradation
  rates, multiple regulatory regimes per gene), which yields spurious root/terminal
  inference as shown for the erythroid gastrulation dataset.
- When a topology-constrained tree inference is needed for fully static data without
  RNA velocity — Cytopath relies on velocity-derived transition probabilities.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Cytopath
  role: 'Simulation-based trajectory inference: defines the Markov chain model from
    RNA velocity, runs terminal-state-supervised and undirected simulations, aligns
    them by dynamic time warping, and builds consensus trajectories'
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Estimates spliced/unspliced RNA velocity for each cell, which forms the transition
    field input to Cytopath's Markov chain
- name: cytopath-notebooks
  role: Sample notebooks demonstrating Cytopath application on publicly available
    single-cell RNA-seq datasets
  repo: https://github.com/aron0093/cytopath-notebooks
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:375647668d8e19e4ea9c0ae2a251c065b1ed4a3535cbd0c665d076dbeff33ec0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Cytopath
  - cytopath-notebooks
  - scvelo
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# terminal-state-supervised-simulation

## Summary

Run Cytopath's Markov chain simulations on an RNA velocity field while supervising terminal states with annotated cell fates (e.g. alpha, beta, gamma, delta endocrine cells), so that the inferred trajectories terminate in biologically validated endpoints. This recovers directed differentiation paths, including complex topologies such as the circular cell-cycle structure feeding into endocrinogenesis.

## When to use

Use when scRNA-seq data with RNA velocity should be resolved into directed differentiation trajectories whose endpoints are known terminal cell states (e.g. annotated alpha/beta/gamma/delta endocrine clusters), especially with cycling or convergent topologies.

## When NOT to use

- When root and terminal cell states are not clearly identifiable — the authors explicitly do not recommend performing Cytopath trajectory inference in this case.
- When the underlying RNA velocity estimation is faulty (e.g. time-dependent degradation rates, multiple regulatory regimes per gene), which yields spurious root/terminal inference as shown for the erythroid gastrulation dataset.
- When a topology-constrained tree inference is needed for fully static data without RNA velocity — Cytopath relies on velocity-derived transition probabilities.

## Inputs

- scRNA-seq dataset with spliced/unspliced counts (e.g. loom/h5ad) and estimated RNA velocity (scvelo)
- Annotated terminal cell-state clusters or labels (e.g. alpha, beta, gamma, delta endocrine cells)
- Optional pre-computed cell clustering; otherwise Louvain clustering is computed internally

## Outputs

- Consensus differentiation trajectories terminated at supervised terminal states (terminal state calls)
- Per-cell trajectory step assignments from the static+velocity alignment score
- Undirected simulation results identifying transcriptional state-switching regions
- Correlation statistics (e.g. Spearman) of cell cycle phase with individual simulation steps, written to e.g. endocrinogenesis_cytopath_results.csv

## How to apply

First estimate spliced/unspliced RNA velocity with scvelo (e.g. from the pancreatic endocrinogenesis data GEO:GSE132188) and construct the Cytopath Markov chain transition model from the velocity field. Supply terminal state supervision using annotated terminal clusters, and derive root/terminal states from a Markov random-walk model on the transition probability matrix; the paper reports pseudotime to be robust to root and terminal probability thresholds within [0.85, 1]. Run Markov sampling of consecutive cell state transitions initialized at root states, then align simulations terminating at a common terminal state with dynamic time warping and assign cells to trajectory steps via an alignment score that combines static expression and velocity profiles. For mixed topologies (e.g. cell cycle feeding differentiation), additionally run undirected simulations initialized at random cells to detect regions of transcriptional state switching, and verify fidelity by correlating annotated cell cycle phase (G1/S/G2M) with the step indices of individual simulations.

## Related tools

- **Cytopath** (Simulation-based trajectory inference: defines the Markov chain model from RNA velocity, runs terminal-state-supervised and undirected simulations, aligns them by dynamic time warping, and builds consensus trajectories) — https://github.com/aron0093/cytopath
- **scvelo** (Estimates spliced/unspliced RNA velocity for each cell, which forms the transition field input to Cytopath's Markov chain)
- **cytopath-notebooks** (Sample notebooks demonstrating Cytopath application on publicly available single-cell RNA-seq datasets) — https://github.com/aron0093/cytopath-notebooks

## Evaluation signals

- The inferred consensus trajectories terminate in the biologically expected terminal states — for pancreatic endocrinogenesis, alpha and delta endocrine states must be recovered among the trajectory endpoints.
- Annotated cell cycle phase (G1/S/G2M) shows Spearman correlation with the transition steps of individual simulations, confirming faithful recapitulation of the cell cycle at the onset of differentiation.
- Pseudotime and terminal calls are stable when root/terminal probability thresholds are varied within the reported robust range [0.85, 1].
- Simulations terminating at a common terminal state align coherently after dynamic time warping, and per-cell alignment scores (static + velocity) assign cells to consistent trajectory steps across multiple runs.
- Root and terminal cell probabilities concentrate on biologically identifiable populations rather than being fuzzy or distributed without a discernible pattern.

## Limitations

- Trajectory inference with Cytopath is not meaningful when the underlying RNA velocity estimation itself is faulty (e.g. due to time-invariant rate assumptions or single regulatory regime per gene).
- Spurious root and terminal states can be inferred when velocity is incorrect — demonstrated on the erythroid gastrulation dataset and a simulated scvelo dataset with time-dependent degradation rates.
- Robustness of pseudotime to root/terminal probability thresholds within [0.85, 1] was reported only as 'data not shown'.
- With no pre-computed clustering supplied, terminal regions depend on an internally computed Louvain clustering.

## Evidence

- [results] simulations that terminate at a common terminal state are aligned using dynamic time warping: "simulations that terminate at a common terminal state are aligned using dynamic time warping"
- [discussion] robust to root/terminal thresholds: "We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown)."
- [results] root/terminal derivation from random-walk model: "Root and terminal states can be derived from a Markov random-walk model utilizing the transition probability matrix itself"
- [results] assignment via static+velocity alignment score: "Assignment is based on an alignment score that evaluates for each cell the similarity of its static as well as the velocity profile with each trajectory step."
- [readme] supervision optional per README: "Cytopath can infer trajectories with or without root/terminal state supervision."
- [discussion] do not recommend without clear endpoints: "We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable."
