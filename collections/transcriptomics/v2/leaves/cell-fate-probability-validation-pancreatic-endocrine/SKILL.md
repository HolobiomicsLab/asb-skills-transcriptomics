---
name: cell-fate-probability-validation-pancreatic-endocrine
description: Use when simulating cell-fate decisions from a preprocessed pancreatic
  endocrine AnnData with cluster annotations and differentiation potentials, and RNA
  velocity is unreliable, so trajectories must be biased by the differentiation-potential
  gradient toward terminal fates.
when_to_use_negative:
- When RNA velocity profiles are robust for the dataset — FateCompass uses them to
  direct the edges of the Markov chain instead of differentiation potential
- When prior biological knowledge defining progenitor sources and terminal fate sinks
  is unavailable (e.g., cancer studies), since the differentiation-potential mode
  depends on such input
- When the data are not a pre-processed AnnData with at least cell-type 'clusters'
  annotations plus a TF binding-site matrix
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: FateCompass
  role: Implements the probabilistic framework that estimates transition probabilities
    from differentiation potential and performs the stochastic trajectory simulations
    whose endpoints give fate probabilities
  repo: https://github.com/sarajimenez/fatecompass
- name: Python
  role: Language of the FateCompass module (fatecompass.py) used to build the cell-neighborhood
    graph and run the differentiation-potential tutorial (fatecompass_basics_differentiation_potential.ipynb)
  repo: https://github.com/sarajimenez/fatecompass
- name: MATLAB
  role: Runs the reproducibility scripts (differentiation_trajectories.m) that generate
    the differentiation-potential-driven trajectories for the pancreatic endocrine
    datasets
  repo: https://github.com/sarajimenez/fatecompass
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:7f73bbe0e383b73113d812f542c5b4d05b3003fb083e7252b0d5d4e5994cfcf9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  repo_url: https://github.com/sarajimenez/fatecompass
  tools:
  - FateCompass
  - MATLAB
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# cell-fate-probability-validation-pancreatic-endocrine

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs the FateCompass differentiation-potential-energy transition kernel for the discrete Markov chain in the human in vitro pancreatic β-cell differentiation (Veres) dataset, propagating stochastic trajectories biased by the differentiation-potential gradient to obtain fate probabilities for SC-α, SC-β, and SC-EC terminal fates.

## When to use

Use when simulating cell-fate decisions from a preprocessed pancreatic endocrine AnnData with cluster annotations and differentiation potentials, and RNA velocity is unreliable, so trajectories must be biased by the differentiation-potential gradient toward terminal fates.

## When NOT to use

- When RNA velocity profiles are robust for the dataset — FateCompass uses them to direct the edges of the Markov chain instead of differentiation potential
- When prior biological knowledge defining progenitor sources and terminal fate sinks is unavailable (e.g., cancer studies), since the differentiation-potential mode depends on such input
- When the data are not a pre-processed AnnData with at least cell-type 'clusters' annotations plus a TF binding-site matrix

## Inputs

- Preprocessed pancreatic endocrine (Veres) AnnData object with cell-type ('clusters') annotations and differentiation potentials
- TF binding-site matrix (columns = TFs/motifs, rows = expressed genes)
- Progenitor cluster labels (NKX6-1+ NEUROG3-early, NEUROG3-mid, NEUROG3-late) and terminal fate labels (SC-α, SC-β, SC-EC)
- Stochastic-simulation parameters: |Q|=100 trajectories, exponent n=0.5

## Outputs

- fate_probabilities.csv — per-progenitor-cluster fate-probability distributions over terminal fates
- Validated fate assignments: SC-α mostly from NEUROG3-early, SC-β/SC-EC from NEUROG3-late progenitors
- MATLAB reproducibility outputs from differentiation_trajectories.m

## How to apply

Load the preprocessed pancreatic endocrine (Veres) AnnData object (cell-type cluster annotations plus differentiation potentials) and the TF binding-site matrix into the FateCompass module (fatecompass.py, Python). Build the cell-neighborhood graph and define the potential-energy landscape from differentiation potential, combining repulsive source forces at progenitor states (NKX6-1+ endocrine progenitors) with attractive sink forces at terminal fates (SC-α, SC-β, SC-EC) to bias transition probabilities beyond RNA velocity. Propagate |Q|=100 stochastic trajectories from each NEUROG3-early and NEUROG3-late progenitor cluster using exponent n=0.5, accepting proposed moves with a Metropolis-style criterion and sampling rejected moves from the rejection distribution (Equations 5–8). Aggregate trajectory endpoints into per-progenitor fate-probability distributions and write fate_probabilities.csv. Validate by checking that SC-alpha fates arise predominantly from NEUROG3-early progenitors while SC-beta/SC-EC fates come from NEUROG3-late progenitors, comparing against the reference results in the FateCompass Shiny App.

## Related tools

- **FateCompass** (Implements the probabilistic framework that estimates transition probabilities from differentiation potential and performs the stochastic trajectory simulations whose endpoints give fate probabilities) — https://github.com/sarajimenez/fatecompass
- **Python** (Language of the FateCompass module (fatecompass.py) used to build the cell-neighborhood graph and run the differentiation-potential tutorial (fatecompass_basics_differentiation_potential.ipynb)) — https://github.com/sarajimenez/fatecompass
- **MATLAB** (Runs the reproducibility scripts (differentiation_trajectories.m) that generate the differentiation-potential-driven trajectories for the pancreatic endocrine datasets) — https://github.com/sarajimenez/fatecompass

## Examples

```
matlab: run('reproducibility/scripts/differentiation_trajectories.m')
```

## Evaluation signals

- fate_probabilities.csv reproduces the reported pattern: SC-alpha probability strongly decreases for NEUROG3-mid and NEUROG3-late progenitors, while SC-beta/SC-EC probabilities increase for NEUROG3-late
- Per-progenitor fate-probability distributions aggregate exactly |Q|=100 stochastic trajectories per cluster, propagated with exponent n=0.5
- Distributions match the reference results for the Veres dataset summarized in the FateCompass Shiny App
- Re-running the MATLAB reproducibility script differentiation_trajectories.m reproduces the kernel endpoints
- Trajectory endpoints fall only in the designated terminal-fate sinks (SC-α, SC-β, SC-EC) and start from NKX6-1+ progenitor sources

## Limitations

- The differentiation-potential mode depends on prior biological knowledge to define progenitor sources and terminal fates, which is difficult to provide in some cases such as cancer studies
- Pipeline input parameters must be hand tuned, although detailed tutorials guide parameter setup
- The potential-energy construction treats the graph in gene expression space; commonly used dimensionality reduction methods such as UMAP lack biological interpretability and must be treated carefully

## Evidence
