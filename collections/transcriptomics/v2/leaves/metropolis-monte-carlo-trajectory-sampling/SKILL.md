---
name: metropolis-monte-carlo-trajectory-sampling
description: Use when scRNA-seq trajectories must be inferred beyond RNA velocity,
  e.g. propagating fates from progenitor (NEUROG3) clusters to terminal fates (SC-α,
  SC-β, SC-EC) using a potential-energy-biased Markov chain.
when_to_use_negative:
- RNA velocity profiles are robust and available — FateCompass then uses velocity
  to direct the Markov-chain edges instead of the potential-energy bias
- No prior biological knowledge of progenitor/sink states is available (e.g. cancer
  studies), since the differentiation-potential mode depends on such annotations
- The goal is TF-activity inference (linear model / differential motif activity) rather
  than fate-probability estimation
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FateCompass
  role: Python module implementing the Metropolis-style stochastic simulation over
    the potential-energy-biased Markov chain (fatecompass.py; tutorial fatecompass_basics_differentiation_potential.ipynb)
  repo: github.com/sarajimenez/fatecompass
- name: MATLAB
  role: Reproducibility scripts (differentiation_trajectories.m, etc.) implementing
    the differentiation-potential-driven trajectory scenario
- name: Scanpy
  role: Suggested tool for basic preprocessing of the AnnData object before trajectory
    sampling
- name: FateCompass Shiny App
  role: Reference results for validation of fate-probability distributions
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:84f59c6fe04b44eaeb5ccd062ce752b278f5d2f837212b04e3e3803355691992
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
  - FateCompass Shiny App
  - MATLAB
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Metropolis Monte Carlo trajectory sampling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs stochastic cell-fate trajectories by sampling Markov-chain transitions whose transition kernel is biased by a differentiation-potential energy landscape, accepting or rejecting proposed moves with a Metropolis-style criterion (Equations 5-8 of FateCompass).

## When to use

Use when scRNA-seq trajectories must be inferred beyond RNA velocity, e.g. propagating fates from progenitor (NEUROG3) clusters to terminal fates (SC-α, SC-β, SC-EC) using a potential-energy-biased Markov chain.

## When NOT to use

- RNA velocity profiles are robust and available — FateCompass then uses velocity to direct the Markov-chain edges instead of the potential-energy bias
- No prior biological knowledge of progenitor/sink states is available (e.g. cancer studies), since the differentiation-potential mode depends on such annotations
- The goal is TF-activity inference (linear model / differential motif activity) rather than fate-probability estimation

## Inputs

- Preprocessed AnnData object (e.g. Veres pancreatic endocrine dataset) with cell-type cluster annotations and differentiation potentials
- TF binding-site matrix (columns = TFs/motifs, rows = expressed genes)
- Source (progenitor, e.g. NKX6-1+ / NEUROG3-early) and sink (terminal fate, e.g. SC-α, SC-β, SC-EC) cell-state definitions
- Sampling parameters: number of trajectories |Q|=100 per progenitor cluster and potential exponent n=0.5

## Outputs

- Per-progenitor-cluster fate-probability distributions over terminal fates (fate_probabilities.csv)
- Set of stochastic trajectory endpoints aggregated by sink state

## How to apply

Load a preprocessed AnnData object with cell-type ('clusters') annotations and differentiation potentials, plus a TF binding-site matrix, into the FateCompass module. Build the cell-neighborhood graph and define a potential-energy landscape from differentiation potential, combining repulsive source forces at progenitor states with attractive sink forces at terminal fates to bias transition probabilities. From each progenitor cluster (e.g. NEUROG3-early, NEUROG3-mid, NEUROG3-late), propagate |Q|=100 stochastic trajectories with exponent n=0.5, accepting proposed moves via a Metropolis-style criterion and drawing rejected moves from a rejection distribution (Equations 5-8). Aggregate trajectory endpoints into per-cluster fate-probability distributions and export them (e.g. fate_probabilities.csv). Validate by checking that the resulting distributions reproduce the known fate pattern — SC-α arising mainly from NEUROG3-early progenitors, SC-β and SC-EC from NEUROG3-late progenitors — against reference results in the FateCompass Shiny App.

## Related tools

- **FateCompass** (Python module implementing the Metropolis-style stochastic simulation over the potential-energy-biased Markov chain (fatecompass.py; tutorial fatecompass_basics_differentiation_potential.ipynb)) — github.com/sarajimenez/fatecompass
- **MATLAB** (Reproducibility scripts (differentiation_trajectories.m, etc.) implementing the differentiation-potential-driven trajectory scenario)
- **Scanpy** (Suggested tool for basic preprocessing of the AnnData object before trajectory sampling)
- **FateCompass Shiny App** (Reference results for validation of fate-probability distributions)

## Examples

```
conda env create -f fatecompass.yml; conda install notebook; jupyter notebook tutorials/fatecompass_basics_differentiation_potential.ipynb
```

## Evaluation signals

- |Q|=100 trajectories per progenitor cluster complete and endpoints aggregate into a valid probability distribution over sinks (SC-α, SC-β, SC-EC)
- Reproduced pattern: SC-α fates predominantly from NEUROG3-early progenitors, with SC-α strongly decreasing for NEUROG3-mid and NEUROG3-late progenitors
- SC-β and SC-EC fate probabilities enriched among NEUROG3-late progenitors, matching the FateCompass Shiny App reference results
- Metropolis acceptance/rejection behaves as in Equations 5-8: rejected moves are resampled from the rejection distribution rather than discarded
- Sanity network check (fatecompass_sanity_network.ipynb) passes before running on the real dataset

## Limitations

- Pipeline input parameters must be hand tuned, although detailed tutorials are provided
- Differentiation-potential mode depends on prior biological knowledge of progenitor and terminal states, which is difficult to supply in systems such as cancer
- Trajectories are built on a nearest-neighbor graph in reduced gene-expression space; dimensionality reductions like UMAP lack biological interpretability and should be treated carefully

## Evidence

- [methods] Propagate |Q|=100 stochastic trajectories from each NEUROG3-early and NEUROG3-late progenitor cluster using exponent n=0.5: [evidence span withheld — anchor status: not_found]
- [readme] we combined a probabilistic framework with RNA velocities ... or a differentiation potential to estimate transition probabilities and perform stochastic simulations: [evidence span withheld — anchor status: not_found]
