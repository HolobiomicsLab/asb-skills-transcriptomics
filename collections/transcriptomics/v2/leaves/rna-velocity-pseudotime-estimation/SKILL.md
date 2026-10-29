---
name: rna-velocity-pseudotime-estimation
description: Use when scRNA-seq data has pre-computed RNA velocity (scvelo/velocyto),
  root and terminal cell states are identifiable or supplied, and per-lineage pseudotime
  must be inferred and validated against a known cell type ordering.
when_to_use_negative:
- When the underlying RNA velocity estimation itself is faulty — spurious root/terminal
  inference was observed on the erythroid gastrulation dataset and on scvelo-simulated
  data with time-dependent degradation rates
- When root and terminal cell states are not clearly identifiable (authors explicitly
  do not recommend trajectory inference with Cytopath in this case), e.g. fuzzy root
  probabilities in the hepatocyte zonation dataset
- When a method with topology-specific assumptions is required instead of independent
  per-trajectory modelling — Cytopath places no tree/cycle constraints, and Slingshot/Monocle3
  are the baselines for tree-like/graph topologies
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Cytopath
  role: 'Core method: defines the Markov chain model from RNA velocity, simulates
    trajectory ensembles, and infers pseudotime per trajectory'
  repo: github.com/aron0093/cytopath
- name: scvelo
  role: Preprocessing and RNA velocity estimation required before running Cytopath
    (>=0.1.25)
- name: velocyto
  role: Produces spliced/unspliced/ambiguous count matrices from raw reads (e.g. Burns
    et al. GSE71982 inner ear data)
  repo: velocyto-team/velocyto-notebooks
- name: Slingshot
  role: Baseline trajectory inference for tree-like topology used for pseudotime comparison
- name: PAGA
  role: Baseline with directionality enabled by velocity pseudotime for graph models
- name: Monocle3
  role: Baseline trajectory inference method for pseudotime comparison
- name: VeTra
  role: Velocity-aware baseline trajectory inference method
- name: Cellpath
  role: Velocity-aware baseline trajectory inference method
- name: cytopath-notebooks
  role: Sample notebooks demonstrating Cytopath on public scRNA-seq datasets
  repo: https://github.com/aron0093/cytopath-notebooks
provenance:
  source_task_ids:
  - task_005
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100359
    title: Simulation-based inference of differentiation trajectories from RNA velocity
      fields
  source_content_hash: sha256:7f04606fcaa10fed6386cbb89baef1cee34351f99974394c8dfb71d4877b1d94
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cellpath
  - Cytopath
  - Monocle3
  - PAGA
  - Slingshot
  - VeTra
  - cytopath-notebooks
  - scvelo
  - velocyto
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# rna-velocity-pseudotime-estimation

## Summary

Estimate per-cell pseudotime by defining a Markov chain model from RNA velocity, simulating an ensemble of differentiation trajectories (Cytopath), aligning them via dynamic time warping, and assigning cells to trajectory steps; pseudotime quality is judged by Spearman correlation with known cell type ordering.

## When to use

Use when scRNA-seq data has pre-computed RNA velocity (scvelo/velocyto), root and terminal cell states are identifiable or supplied, and per-lineage pseudotime must be inferred and validated against a known cell type ordering.

## When NOT to use

- When the underlying RNA velocity estimation itself is faulty — spurious root/terminal inference was observed on the erythroid gastrulation dataset and on scvelo-simulated data with time-dependent degradation rates
- When root and terminal cell states are not clearly identifiable (authors explicitly do not recommend trajectory inference with Cytopath in this case), e.g. fuzzy root probabilities in the hepatocyte zonation dataset
- When a method with topology-specific assumptions is required instead of independent per-trajectory modelling — Cytopath places no tree/cycle constraints, and Slingshot/Monocle3 are the baselines for tree-like/graph topologies

## Inputs

- AnnData object (h5ad) with pre-computed RNA velocity from scvelo, or spliced/unspliced/ambiguous count loom matrices from velocyto
- Supplied root and terminal cell states (or labels/clustering for unsupervised root/terminal estimation)
- Known ordering of cell types per lineage for validation

## Outputs

- Per-cell pseudotime per inferred trajectory (cell-to-trajectory-step assignments via alignment score)
- Consensus trajectories (DTW-aligned, averaged expression states per step)
- Per-method/per-lineage/per-run Spearman correlation table (e.g. hippocampus_pseudotime_correlations.csv)

## How to apply

Preprocess the AnnData object and estimate RNA velocity with scvelo (>=0.1.25), since Cytopath depends on scvelo to process the data; loom files with spliced/unspliced/ambiguous counts can be produced upstream by velocyto. Define a Markov chain model from the velocity field and perform Markov sampling of consecutive cell state transitions initialized at predefined root states to generate an ensemble of simulated cell state sequences. Align simulations terminating at a common terminal state using dynamic time warping, average cell states at every step to obtain consensus expression states, and assign cells to trajectory steps via an alignment score comparing static and velocity profiles. Root and terminal states can either be supplied or derived unsupervised from the Markov random-walk model using the transition probability matrix; if no clustering is supplied, terminal regions are determined with internal Louvain clustering. Run inference over multiple independent initialisations (e.g. 10 runs on the 18,140-cell developing mouse hippocampus dataset) and validate by computing scipy.stats.spearmanr between inferred pseudotime and the known cell type ordering per lineage; pseudotime is reported robust to root and terminal probability thresholds within [0.85, 1].

## Related tools

- **Cytopath** (Core method: defines the Markov chain model from RNA velocity, simulates trajectory ensembles, and infers pseudotime per trajectory) — github.com/aron0093/cytopath
- **scvelo** (Preprocessing and RNA velocity estimation required before running Cytopath (>=0.1.25))
- **velocyto** (Produces spliced/unspliced/ambiguous count matrices from raw reads (e.g. Burns et al. GSE71982 inner ear data)) — velocyto-team/velocyto-notebooks
- **Slingshot** (Baseline trajectory inference for tree-like topology used for pseudotime comparison)
- **PAGA** (Baseline with directionality enabled by velocity pseudotime for graph models)
- **Monocle3** (Baseline trajectory inference method for pseudotime comparison)
- **VeTra** (Velocity-aware baseline trajectory inference method)
- **Cellpath** (Velocity-aware baseline trajectory inference method)
- **cytopath-notebooks** (Sample notebooks demonstrating Cytopath on public scRNA-seq datasets) — https://github.com/aron0093/cytopath-notebooks

## Examples

```
from scipy.stats import spearmanr; rho, p = spearmanr(cytopath_pseudotime, known_celltype_order)
```

## Evaluation signals

- Spearman correlation between inferred pseudotime and known cell type ordering per lineage is high and consistent across 10 independent initialisations
- Consensus trajectories align correctly: simulations terminating at a common terminal state superimpose after dynamic time warping
- Root and terminal state estimates are stable when the root/terminal probability threshold is varied within [0.85, 1]
- Inferred pseudotime recovers expected biological ordering even under complex topologies (cycling, convergence, e.g. the two convergent inner ear lineages and the hippocampus lineages)
- Recorded baseline failure modes (Monocle3 disconnected-graph failure, reversed PAGA directionality) confirm supervision with identical root/terminal states was applied consistently

## Limitations

- Robustness of Cytopath pseudotime to root and terminal probability thresholds is reported only as 'data not shown' for the range [0.85, 1]
- RNA velocity estimation rests on simplifying assumptions (time-invariant transcription/splicing/degradation rates, single regulatory regime per gene), so correctness cannot be verified in every scenario
- Faulty velocity yields spurious endpoints: the erythroid gastrulation dataset and a simulated time-dependent degradation dataset showed roots inferred toward the terminus; hepatocyte zonation showed fuzzy, patternless root probabilities

## Evidence

- [abstract] Cytopath performs this task by defining a Markov chain model, simulating an ensemble of possible differentiation trajectories, and constructing a consensus trajectory.: "Cytopath performs this task by defining a Markov chain model, simulating an ensemble of possible differentiation trajectories, and constructing a consensus trajectory."
- [results] Alignment score cell assignment: "Assignment is based on an alignment score that evaluates for each cell the similarity of its static as well as the velocity profile with each trajectory step."
- [results] Unsupervised root/terminal estimation: "Root and terminal states can be derived from a Markov random-walk model utilizing the transition probability matrix itself (Figure 1A.3), as described by La Manno et al.,1"
- [results] Validation via Spearman correlation: "The Spearman correlation between pseudotime inferred by Cytopath for each trajectory to known ordering of cell types is high (Figure 2D) and robust across multiple runs (Figure 2E)."
- [discussion] Recommendation against unclear endpoints: "We do not recommend performing trajectory inference with Cytopath when the root and terminal cell states are not clearly identifiable."
- [readme] Velocity dependence from README: "Transcriptional activity information from RNA velocity of single cells is used to define a Markov chain model; simulation of this model yields an ensemble of possible differentiation trajectories"
