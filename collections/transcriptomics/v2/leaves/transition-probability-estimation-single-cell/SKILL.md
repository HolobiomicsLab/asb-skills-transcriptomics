---
name: transition-probability-estimation-single-cell
description: Use when scRNA-seq data capture a differentiation process and you need
  quantitative fate probabilities from progenitor clusters to terminal states, using
  RNA velocity or a biologically defined differentiation potential as the biasing
  force.
when_to_use_negative:
- Robust RNA velocity profiles are unavailable AND prior biological knowledge to define
  the differentiation potential cannot be supplied (e.g. cancer studies)
- Dimensionality reduction of the input space relies on methods like UMAP whose lack
  of biological interpretability undermines nearest-neighbor graph construction
- The goal is TF activity estimation rather than fate transition probabilities (use
  the downstream linear-model-of-gene-regulation step instead)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: FateCompass
  role: Implements the Markov-chain transition-probability estimation and stochastic
    trajectory propagation (fatecompass.py; differentiation_trajectories.m for reproducibility)
  repo: github.com/sarajimenez/fatecompass
- name: Python
  role: Language of the fully implemented FateCompass module used for the differentiation-potential
    workflow
- name: MATLAB
  role: Runs the reproducibility scripts (differentiation_trajectories.m, tf_activities.m,
    bootstrapping.m, differential_motif_activity_analysis.m)
  repo: github.com/sarajimenez/fatecompass
- name: Scanpy
  role: Suggested for basic pre-processing of the single-cell object before FateCompass
- name: scVelo
  role: Suggested for computing the RNA velocity field used to direct Markov-chain
    edges
- name: Dyngen
  role: In silico simulation engine generating ground-truth systems used to validate
    FateCompass transition estimates
  repo: dynverse/dyngen
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:2ceeb3c7cd785aa48d813c1fb13f534157ff61ac12ad24bfafb36b9cb0f0465b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  tools:
  - Dyngen
  - FateCompass
  - MATLAB
  - Python
  - Scanpy
  - scVelo
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# transition-probability-estimation-single-cell

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates cell-fate transition probabilities in single-cell differentiation data using FateCompass's discrete Markov process, biased by RNA velocity or a differentiation-potential gradient, then propagates stochastic trajectories to quantify fate probabilities per progenitor cluster.

## When to use

Use when scRNA-seq data capture a differentiation process and you need quantitative fate probabilities from progenitor clusters to terminal states, using RNA velocity or a biologically defined differentiation potential as the biasing force.

## When NOT to use

- Robust RNA velocity profiles are unavailable AND prior biological knowledge to define the differentiation potential cannot be supplied (e.g. cancer studies)
- Dimensionality reduction of the input space relies on methods like UMAP whose lack of biological interpretability undermines nearest-neighbor graph construction
- The goal is TF activity estimation rather than fate transition probabilities (use the downstream linear-model-of-gene-regulation step instead)

## Inputs

- Pre-processed AnnData (.h5ad) object with cell-type 'clusters' annotations and optional RNA velocity field (Scanpy/scVelo pre-processing recommended)
- TF binding-site matrix (rows = expressed genes, columns = TFs or motifs)
- Prior biological knowledge defining progenitor source states and terminal fate sink states and differentiation potentials
- Conda environment specification (fatecompass.yml) with FateCompass dependencies

## Outputs

- Per-progenitor-cluster fate-probability distributions over terminal states (e.g. fate_probabilities.csv)
- Stochastic trajectories of cells from progenitor to terminal states on the cell-neighborhood graph

## How to apply

Load a pre-processed AnnData object (with 'clusters' cell-type annotations and, optionally, the RNA velocity field) plus a TF binding-site matrix (rows = expressed genes, columns = TFs/motifs) into the FateCompass Python module or its MATLAB reproducibility scripts. Build a nearest-neighbor cell graph and define the transition kernel: direct Markov-chain edges with RNA velocity when velocity profiles are robust, or bias transition probabilities using the gradient of a differentiation-potential landscape that combines repulsive source forces at progenitor states with attractive sink forces at terminal states (Equations 5–8). Propagate stochastic trajectories (e.g. |Q|=100 per progenitor cluster, exponent n=0.5) accepting moves via a Metropolis-style criterion and sampling rejections from a rejection distribution, then aggregate trajectory endpoints into per-cluster fate-probability distributions (fate_probabilities.csv). In the Veres pancreatic endocrine dataset, this kernel distributes SC-alpha, SC-beta, and SC-EC fate probabilities from NEUROG3-early/mid/late progenitors, revealing SC-alpha cells arise mainly from early endocrine precursors while SC-beta/SC-EC arise from NEUROG3-late progenitors.

## Related tools

- **FateCompass** (Implements the Markov-chain transition-probability estimation and stochastic trajectory propagation (fatecompass.py; differentiation_trajectories.m for reproducibility)) — github.com/sarajimenez/fatecompass
- **Python** (Language of the fully implemented FateCompass module used for the differentiation-potential workflow)
- **MATLAB** (Runs the reproducibility scripts (differentiation_trajectories.m, tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m)) — github.com/sarajimenez/fatecompass
- **Scanpy** (Suggested for basic pre-processing of the single-cell object before FateCompass)
- **scVelo** (Suggested for computing the RNA velocity field used to direct Markov-chain edges)
- **Dyngen** (In silico simulation engine generating ground-truth systems used to validate FateCompass transition estimates) — dynverse/dyngen

## Examples

```
python -m notebook fatecompass_basics_differentiation_potential.ipynb  # or: matlab -batch differentiation_trajectories
```

## Evaluation signals

- Fate-probability distributions reproduce the reported Veres dataset pattern: SC-alpha fates predominantly from NEUROG3-early progenitors and SC-beta/SC-EC fates from NEUROG3-late progenitors (cross-check against the FateCompass Shiny App)
- Method capabilities first validated against a Dyngen-generated ground-truth in silico system before real-data application
- Aggregated trajectory endpoints across |Q|=100 propagated trajectories per progenitor cluster yield normalized fate-probability distributions over defined terminal clusters
- Pipeline order matches the documented reproducibility flow: differentiation_trajectories.m → tf_activities.m → bootstrapping.m → differential_motif_activity_analysis.m

## Limitations

- The differentiation-potential mode depends on prior biological knowledge, which is difficult to provide in some cases such as cancer studies
- Pipeline input parameters must be hand tuned, although detailed tutorials are provided
- Nearest-neighbor graphs are built in a reduced gene expression space; commonly used dimensionality reductions such as UMAP lack biological interpretability and should be treated carefully
- Input parameters (e.g. number of trajectories |Q|, potential exponent n) and the Metropolis acceptance scheme make fate probabilities stochastic; reproducibility scripts are provided in MATLAB

## Evidence
