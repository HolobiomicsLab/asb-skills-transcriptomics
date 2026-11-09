---
name: cell-fate-probability-quantification
description: Use when scRNA-seq data with cluster annotations and an RNA velocity
  field are available and you need per-cell probabilities of reaching defined terminal
  fates (e.g., alpha, beta, epsilon) plus aggregate end-fate frequencies from stochastic
  Markov sampling.
when_to_use_negative:
- RNA velocity profiles are not robust and prior biological knowledge for a differentiation
  potential is unavailable (e.g., cancer studies), since FateCompass depends on one
  of these two inputs to direct the Markov chain
- The goal is to infer lineage-specific transcription factor activities rather than
  fate probabilities — the tf_activities.m and differential_motif_activity_analysis.m
  steps serve that purpose instead
- Input data lacks cluster/terminal-state annotations or has not undergone Scanpy-based
  preprocessing and scVelo velocity estimation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FateCompass
  role: Estimates transition probabilities and performs stochastic Markov simulations
    of differentiation trajectories to compute fate probabilities
  repo: github.com/sarajimenez/fatecompass
- name: scVelo
  role: Computes the RNA velocity field used to direct the edges of the discrete Markov
    chain
  repo: https://scvelo.readthedocs.io/en/stable/
- name: Scanpy
  role: Basic preprocessing of the single-cell data before FateCompass runs
  repo: https://scanpy.readthedocs.io/en/stable/
- name: Python
  role: Hosts the FateCompass module (fatecompass.py) used for the RNA-velocity-driven
    scenario
- name: Matlab
  role: Runs the reproducibility scripts, starting with differentiation_trajectories.m
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:11b6de931af27563470de99edfd1a2afba6ac0876a587d9300e5ab8e7879816d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  tools:
  - FateCompass
  - Matlab
  - Python
  - Scanpy
  - scVelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cell-fate-probability-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies end-state probabilities of cell-fate decisions by running stochastic simulations of a discrete Markov chain whose edges are directed by RNA velocity (or differentiation potential), as implemented in FateCompass. Applied to scRNA-seq differentiation systems such as the E15.5 mouse pancreas to obtain per-cell fate probabilities and aggregate end-fate frequencies.

## When to use

Use when scRNA-seq data with cluster annotations and an RNA velocity field are available and you need per-cell probabilities of reaching defined terminal fates (e.g., alpha, beta, epsilon) plus aggregate end-fate frequencies from stochastic Markov sampling.

## When NOT to use

- RNA velocity profiles are not robust and prior biological knowledge for a differentiation potential is unavailable (e.g., cancer studies), since FateCompass depends on one of these two inputs to direct the Markov chain
- The goal is to infer lineage-specific transcription factor activities rather than fate probabilities — the tf_activities.m and differential_motif_activity_analysis.m steps serve that purpose instead
- Input data lacks cluster/terminal-state annotations or has not undergone Scanpy-based preprocessing and scVelo velocity estimation

## Inputs

- Pre-processed AnnData object with cell-type ('clusters') annotations (e.g., Bastidas-Ponce E15.5, 3,696 cells)
- RNA velocity field computed with scVelo (or differentiation-potential annotations for the alternative scenario)
- Conda environment defined by fatecompass.yml

## Outputs

- Per-cell fate probability table (fate_probabilities_e155.csv)
- Aggregated end-fate frequencies across terminal states (e.g., alpha ~12.4%, beta ~77%, plus epsilon) over 2,000 iterations
- Sampled differentiation trajectories from progenitor states to terminal fates

## How to apply

Load a pre-processed AnnData object (e.g., the Bastidas-Ponce E15.5 mouse pancreas dataset with 3,696 cells) containing 'clusters' annotations and an RNA velocity field computed with scVelo following Scanpy-based preprocessing. Build a nearest-neighbor network in UMAP space with k=10 neighbors and derive transition probabilities from the RNA velocity vectors, which FateCompass uses to direct the edges of the discrete Markov chain. Run the RNA-velocity-driven trajectory sampling (the differentiation_trajectories step of the reproducibility pipeline, executed via fatecompass.py in Python or the Matlab scripts) with a large number of iterations (e.g., 2,000) from progenitor states to terminal endocrine fates. Aggregate sampled trajectories into per-cell fate probabilities (fate_probabilities_e155.csv) and overall end-fate frequencies; when RNA velocity profiles are not robust, bias transition probabilities with the gradient of a differentiation potential instead.

## Related tools

- **FateCompass** (Estimates transition probabilities and performs stochastic Markov simulations of differentiation trajectories to compute fate probabilities) — github.com/sarajimenez/fatecompass
- **scVelo** (Computes the RNA velocity field used to direct the edges of the discrete Markov chain) — https://scvelo.readthedocs.io/en/stable/
- **Scanpy** (Basic preprocessing of the single-cell data before FateCompass runs) — https://scanpy.readthedocs.io/en/stable/
- **Python** (Hosts the FateCompass module (fatecompass.py) used for the RNA-velocity-driven scenario)
- **Matlab** (Runs the reproducibility scripts, starting with differentiation_trajectories.m)

## Examples

```
conda env create -f fatecompass.yml; matlab -batch "run('reproducibility/scripts/differentiation_trajectories.m')"
```

## Evaluation signals

- Output file fate_probabilities_e155.csv exists and is populated for all starting cells
- Reproduced end-fate frequencies match the reported distribution (alpha ~12.4%, beta ~77%) within the tolerance implied by stochastic Markov sampling
- Simulated trajectories reach exactly the expected terminal endocrine fates (alpha, beta, epsilon) and none outside them
- Simulations complete the full 2,000 iterations from progenitor states without premature termination
- Transition probabilities are non-negative, row-normalized over the k=10 nearest-neighbor network derived in UMAP space

## Limitations

- Trajectories are built on a nearest-neighbor graph in UMAP space, which lacks biological interpretability and should be treated carefully
- Pipeline input parameters must be hand tuned (tutorials are provided to guide the setup)
- Results are stochastic: end-fate frequencies from 2,000 iterations vary between runs and must be compared within stochastic tolerance

## Evidence
