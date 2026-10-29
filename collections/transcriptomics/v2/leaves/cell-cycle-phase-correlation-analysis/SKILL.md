---
name: cell-cycle-phase-correlation-analysis
description: Use when scRNA-seq data exhibit dominant cell cycling at the onset of
  differentiation and you need to check whether RNA velocity-based simulation trajectories
  recapitulate cell cycle stages.
when_to_use_negative:
- When root and terminal cell states are not clearly identifiable — the authors do
  not recommend performing trajectory inference with Cytopath in that case
- When the underlying RNA velocity estimation is faulty (e.g. time-dependent degradation
  rates), which leads to spurious root/terminal inference and meaningless trajectory
  inference
- When the dataset is static and not expected to contain a circular/cycling component,
  so cell cycle phase correlation is not a meaningful validation signal
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Cytopath
  role: 'Simulation-based trajectory inference: Markov chain model, Markov sampling
    of cell state transitions, dynamic time warping alignment, and undirected simulations
    initialized at random cells'
  repo: https://github.com/aron0093/cytopath
- name: scvelo
  role: Estimates spliced/unspliced RNA velocity for each cell, the input velocity
    field for the Cytopath Markov chain model
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:16ce9d92e7ac36ac828a4a3183c7946ccb483ea805f0e606b16db91ab978b984
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aron0093/cytopath
  tools:
  - Cytopath
  - scvelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cell-cycle-phase-correlation-analysis

## Summary

Verifies that trajectories inferred by Cytopath on cycling scRNA-seq data (e.g. pancreatic endocrinogenesis, GEO:GSE132188) faithfully recapitulate the circular cell cycle structure at the onset of differentiation by correlating annotated G1/S/G2M phase with the transition steps of individual Markov simulations.

## When to use

Use when scRNA-seq data exhibit dominant cell cycling at the onset of differentiation and you need to check whether RNA velocity-based simulation trajectories recapitulate cell cycle stages.

## When NOT to use

- When root and terminal cell states are not clearly identifiable — the authors do not recommend performing trajectory inference with Cytopath in that case
- When the underlying RNA velocity estimation is faulty (e.g. time-dependent degradation rates), which leads to spurious root/terminal inference and meaningless trajectory inference
- When the dataset is static and not expected to contain a circular/cycling component, so cell cycle phase correlation is not a meaningful validation signal

## Inputs

- scRNA-seq dataset with spliced/unspliced counts (e.g. pancreatic endocrinogenesis data, GEO:GSE132188)
- RNA velocity field estimated by scvelo
- Per-cell annotated cell cycle phase (G1/S/G2M)
- Terminal state labels (alpha, beta, gamma, delta endocrine cells) and root/terminal probability thresholds in [0.85, 1]

## Outputs

- Trajectory step assignments per cell (static+velocity alignment score based)
- Terminal state calls for consensus trajectories (e.g. recovery of alpha and delta endpoints)
- Undirected simulation results showing regions of transcriptional state switching
- Spearman correlation statistics between cell cycle phase and individual simulation transition steps
- Results table such as endocrinogenesis_cytopath_results.csv

## How to apply

Process the dataset with scvelo to estimate spliced/unspliced RNA velocity, then construct the Cytopath Markov chain transition model with terminal state supervision (root/terminal probability thresholds within the robust range [0.85, 1]). Run Cytopath Markov sampling, align simulations terminating at a common terminal state with dynamic time warping, and assign cells to trajectory steps via the alignment score of static and velocity profiles. Additionally run undirected Cytopath simulations initialized at random cells to capture the cell cycle-to-differentiation switching region. Finally, compute Spearman correlation of annotated cell cycle phase (G1/S/G2M) with the step indices of each individual simulation; faithful recapitulation means cycling phase tracks simulation transitions at the single-simulation level. Write step assignments, terminal state calls, undirected simulation results, and correlation statistics to an output CSV (e.g. endocrinogenesis_cytopath_results.csv).

## Related tools

- **Cytopath** (Simulation-based trajectory inference: Markov chain model, Markov sampling of cell state transitions, dynamic time warping alignment, and undirected simulations initialized at random cells) — https://github.com/aron0093/cytopath
- **scvelo** (Estimates spliced/unspliced RNA velocity for each cell, the input velocity field for the Cytopath Markov chain model)

## Examples

```
import scvelo as scv; scv.tl.velocity(adata, mode='stochastic')
```

## Evaluation signals

- Alpha and delta terminal states are recovered among the inferred trajectory endpoints on the pancreatic endocrinogenesis dataset
- Spearman correlation of cell cycle phase (G1/S/G2M) with transition steps of individual simulations indicates faithful recapitulation at the single-simulation level
- Undirected simulations initialized at random cells identify the cell cycle-to-differentiation transition at the onset of endocrinogenesis
- Inferred pseudotime/trajectory results are robust to root and terminal probability thresholds within [0.85, 1]
- Output CSV contains trajectory step assignments, terminal state calls, undirected simulation results, and cell cycle correlation statistics

## Limitations

- Trajectory inference with Cytopath is not meaningful when the underlying RNA velocity estimation itself is faulty (e.g. simplifying assumptions of time-invariant transcription/splicing/degradation rates)
- Incorrect RNA velocity estimation can lead to spurious inference of root and terminal cell states, as seen in the erythroid gastrulation dataset and a simulated dataset with time-dependent degradation rates
- Correctness of RNA velocity estimation cannot be verified in every scenario, even when independent marker-based information is available

## Evidence

- [results] simulations that terminate at a common terminal state are aligned using dynamic time warping: "simulations that terminate at a common terminal state are aligned using dynamic time warping"
- [results] Assignment is based on an alignment score that evaluates for each cell the similarity of its static as well as the velocity profile with each trajectory step.: "Assignment is based on an alignment score that evaluates for each cell the similarity of its static as well as the velocity profile with each trajectory step."
- [results] we initialized simulations at random cells in the dataset (Figure 4C).: "we initialized simulations at random cells in the dataset (Figure 4C)."
- [discussion] We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1]: "We found pseudotime inferred by Cytopath to be robust to root and terminal probability thresholds within a range of [0.85, 1] (data not shown)."
- [readme] Cytopath can model complex behaviours like cycling and convergence as well as cooccurring combinations of multiple: "Cytopath can model complex behaviours like cycling and convergence as well as cooccurring combinations of multiple"
