---
name: markov-chain-transition-probability-estimation
description: Use when single-cell RNA-seq data must be converted into stochastic differentiation
  trajectories and per-cell fate probabilities, especially when RNA velocity is noisy
  and a progenitor-defined differentiation potential gradient is available.
when_to_use_negative:
- Data lack cell-type annotations or any basis (robust RNA velocity or prior biological
  knowledge of progenitors) for defining trajectory direction — differentiation-potential
  mode depends on prior knowledge that is hard to supply in some cases such as cancer
  studies
- The goal is TF activity or differential TF analysis rather than trajectory/fate-probability
  estimation — the downstream linear model of gene regulation and differential_motif_activity_analysis.m
  are separate steps
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: FateCompass
  role: Implements the probabilistic framework combining RNA velocities or differentiation
    potential to estimate transition probabilities and perform stochastic simulations;
    fully implemented in python (fatecompass.py), with MATLAB reproducibility scripts
  repo: https://github.com/sarajimenez/fatecompass
- name: Scanpy
  role: Suggested for basic pre-processing of the adata object (cell-type annotated,
    e.g. 25,299 Veres stage-5 cells) used as trajectory input
- name: scVelo
  role: Suggested, together with Scanpy, for pre-processing and providing the RNA
    velocity field that can direct Markov-chain edges
- name: Python
  role: Language in which the FateCompass module (fatecompass.py) is fully implemented
    and tutorials are provided as Jupyter notebooks
  repo: https://github.com/sarajimenez/fatecompass
- name: MATLAB
  role: Runs the reproducibility scripts, starting with differentiation_trajectories.m,
    that build differentiation-potential-driven Markov-chain trajectories for the
    pancreatic endocrine datasets
  repo: https://github.com/sarajimenez/fatecompass
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:96874c9388ff22b5aeae684b60b2a148d43d1f351abe86bd641d0c2c24936a41
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
  - Scanpy
  - scVelo
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Markov-chain transition probability estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate cell-fate transition probabilities by directing the edges of a discrete Markov process on a nearest-neighbor network with RNA velocities or, when velocities are inconclusive, the gradient of differentiation potential from annotated progenitors, producing fate-probability distributions per cell for downstream TF activity inference.

## When to use

Use when single-cell RNA-seq data must be converted into stochastic differentiation trajectories and per-cell fate probabilities, especially when RNA velocity is noisy and a progenitor-defined differentiation potential gradient is available.

## When NOT to use

- Data lack cell-type annotations or any basis (robust RNA velocity or prior biological knowledge of progenitors) for defining trajectory direction — differentiation-potential mode depends on prior knowledge that is hard to supply in some cases such as cancer studies
- The goal is TF activity or differential TF analysis rather than trajectory/fate-probability estimation — the downstream linear model of gene regulation and differential_motif_activity_analysis.m are separate steps

## Inputs

- Pre-processed AnnData (adata) object with at least cell-type ('clusters') annotations and optionally an RNA velocity field (scVelo/Scanpy pre-processing suggested)
- Optional: RNA velocity estimates (La Manno et al. 2018 / Bergen et al. 2020)
- Progenitor and terminal-state annotations (e.g. NKX6-1+ progenitors; SC-α, SC-β, SC-EC terminal fates)
- MATLAB reproducibility scripts (differentiation_trajectories.m)

## Outputs

- Transition probabilities between cells/states on the Markov chain
- Fate-probability distributions per cell toward each terminal state
- Stochastic trajectory simulations of differentiation from progenitors to final states

## How to apply

Load a pre-processed adata object (cell-type annotated, e.g. 25,299 Veres et al. stage-5 cells, pre-processed with Scanpy) and, optionally, an RNA velocity field. Outline differentiation trajectories from progenitor cells to terminal states using a discrete Markov process on a network: when RNA velocity profiles are robust, use them to direct the edges of the Markov chain; otherwise bias the transition probabilities with the gradient of differentiation potential from a biologically defined progenitor population (e.g. NKX6-1+ progenitors toward SC-α, SC-β and SC-EC terminal states). For the Veres stage-5 case, run differentiation_trajectories.m from the reproducibility/scripts folder (MATLAB) to produce fate-probability distributions per cell. The rationale is that a probabilistic framework accommodates the stochasticity of cell-fate decisions, and the differentiation-potential mode recovers trajectories 'beyond RNA velocity' when projected velocities are inconclusive. Note that trajectories are built on a nearest-neighbor graph in the gene expression space rather than an embedding like UMAP, which lacks biological interpretability.

## Related tools

- **FateCompass** (Implements the probabilistic framework combining RNA velocities or differentiation potential to estimate transition probabilities and perform stochastic simulations; fully implemented in python (fatecompass.py), with MATLAB reproducibility scripts) — https://github.com/sarajimenez/fatecompass
- **Scanpy** (Suggested for basic pre-processing of the adata object (cell-type annotated, e.g. 25,299 Veres stage-5 cells) used as trajectory input)
- **scVelo** (Suggested, together with Scanpy, for pre-processing and providing the RNA velocity field that can direct Markov-chain edges)
- **Python** (Language in which the FateCompass module (fatecompass.py) is fully implemented and tutorials are provided as Jupyter notebooks) — https://github.com/sarajimenez/fatecompass
- **MATLAB** (Runs the reproducibility scripts, starting with differentiation_trajectories.m, that build differentiation-potential-driven Markov-chain trajectories for the pancreatic endocrine datasets) — https://github.com/sarajimenez/fatecompass

## Examples

```
conda env create -f fatecompass.yml; jupyter notebook tutorials/fatecompass_basics_differentiation_potential.ipynb
```

## Evaluation signals

- differentiation_trajectories.m runs without error on the annotated adata object and outputs a fate-probability distribution per cell for each terminal state (SC-α, SC-β, SC-EC)
- Fate probabilities for terminal-state cells are consistent with their annotated identity (cells annotated as a terminal fate receive high probability for that fate)
- Results for the Veres et al. stage-5 dataset match the reference trajectories summarized in the FateCompass Shiny App (https://jimenezs.shinyapps.io/FateCompass_results/)
- The trajectory topology connects the chosen progenitor population (e.g. NKX6-1+ progenitors) to all expected terminal fates rather than collapsing into a single path
- Downstream steps (tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m) run on the trajectory output and recover the reported 126 differentially active TFs

## Limitations

- Differentiation-potential mode depends on prior biological knowledge to define progenitors and terminal states, which is difficult to provide in some cases such as cancer studies
- Trajectories are built on a nearest-neighbor graph in gene expression space; commonly used dimensionality reduction methods such as UMAP should be treated carefully given their lack of biological interpretability
- Pipeline input parameters must be hand tuned, although detailed tutorials are provided to guide parameter setup

## Evidence
