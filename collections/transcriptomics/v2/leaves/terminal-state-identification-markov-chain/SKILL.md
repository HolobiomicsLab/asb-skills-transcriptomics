---
name: terminal-state-identification-markov-chain
description: Use when a pre-processed anndata object with cluster annotations and
  an RNA velocity field is available and you must determine which populations are
  terminal fates (e.g. which endocrine populations are absorbing states for Ngn3-low
  EP progenitors).
when_to_use_negative:
- When no RNA velocity field is available or velocity profiles are not robust — FateCompass
  can bias transitions with a gradient of differentiation potential instead, but that
  mode requires prior biological knowledge that is hard to supply (e.g. cancer studies)
- When the goal is transcription-factor activity inference rather than terminal-state/fate-probability
  identification — the linear gene-regulation model steps (tf_activities.m, bootstrapping.m)
  address a different question
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FateCompass
  role: Estimates transition probabilities from RNA velocity via a discrete Markov
    process on a network and ranks the most likely terminal fates; module provided
    in fatecompass.py
  repo: github.com/sarajimenez/fatecompass
- name: CellRank
  role: 'Baseline method: builds a velocity kernel on the KNN graph and uses its Markov-chain
    estimator to compute absorption probabilities and call terminal states for comparison
    on the E15.5 dataset'
- name: Scanpy
  role: Basic pre-processing of the scRNA-seq data object prior to trajectory construction
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scVelo
  role: Provides the RNA velocity field used to direct the edges of the Markov chain
  repo: https://scvelo.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:36e14808403c0eac16e8b7d852de97bead5bed01d30e728f2ff45653350b9bc1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  tools:
  - CellRank
  - FateCompass
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
yes_rate: 0.875
n_claims: 8
---

# terminal-state-identification-markov-chain

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify terminal (absorbing) cell states of a differentiation landscape by building a discrete Markov process on a KNN graph over single-cell transcriptomics, with edges directed by scVelo-derived RNA velocity, and estimating fate/absorption probabilities toward candidate terminal populations. Applied in FateCompass and benchmarked against CellRank on the mouse E15.5 pancreas dataset.

## When to use

Use when a pre-processed anndata object with cluster annotations and an RNA velocity field is available and you must determine which populations are terminal fates (e.g. which endocrine populations are absorbing states for Ngn3-low EP progenitors).

## When NOT to use

- When no RNA velocity field is available or velocity profiles are not robust — FateCompass can bias transitions with a gradient of differentiation potential instead, but that mode requires prior biological knowledge that is hard to supply (e.g. cancer studies)
- When the goal is transcription-factor activity inference rather than terminal-state/fate-probability identification — the linear gene-regulation model steps (tf_activities.m, bootstrapping.m) address a different question

## Inputs

- Pre-processed anndata object with cell-type ('clusters') annotations including Ductal and Ngn3-low EP populations
- scVelo-derived RNA velocity field on the same cells
- KNN graph of the E15.5 dataset
- List of candidate terminal populations (e.g. α, β, ε cells)

## Outputs

- Per-population fate/absorption probabilities toward each candidate terminal state
- Terminal-state calls per population (which clusters are absorbing)
- Terminal-fate ranking from FateCompass (e.g. β as most likely fate for endocrine progenitors)
- Comparison table cellrank_fate_probabilities.csv with CellRank fate probability, CellRank terminal-state call, and FateCompass terminal-fate ranking per population

## How to apply

Load the pre-processed E15.5 anndata object (with 'clusters' annotations including Ductal and Ngn3-low EP, and scVelo RNA velocity) pre-processed as suggested with Scanpy and scVelo. Build a directed transition kernel from the velocity field on the KNN graph, since FateCompass outlines differentiation trajectories 'from progenitor cells to final states using a discrete Markov Process on a network'. Estimate absorption (fate) probabilities toward each candidate terminal population with the Markov-chain estimator and record which populations are called terminal. In the E15.5 baseline, CellRank identified ε and β cells as terminal hormone-producing fates but failed to call glucagon-producing α cells terminal, whereas FateCompass ranked the β fate as the most likely terminal fate for endocrine progenitors across all tested parameters and kernels; aggregate per-cell-type fate probabilities and terminal calls from both methods into a comparison table (cellrank_fate_probabilities.csv).

## Related tools

- **FateCompass** (Estimates transition probabilities from RNA velocity via a discrete Markov process on a network and ranks the most likely terminal fates; module provided in fatecompass.py) — github.com/sarajimenez/fatecompass
- **CellRank** (Baseline method: builds a velocity kernel on the KNN graph and uses its Markov-chain estimator to compute absorption probabilities and call terminal states for comparison on the E15.5 dataset)
- **Scanpy** (Basic pre-processing of the scRNA-seq data object prior to trajectory construction) — https://scanpy.readthedocs.io/en/stable/
- **scVelo** (Provides the RNA velocity field used to direct the edges of the Markov chain) — https://scvelo.readthedocs.io/en/stable/

## Examples

```
conda env create -f fatecompass.yml && jupyter notebook tutorials/fatecompass_basics_rna_velocity.ipynb
```

## Evaluation signals

- On the E15.5 dataset including Ductal and Ngn3-low EP populations, CellRank should identify ε and β cells as terminal hormone-producing fates while NOT identifying glucagon-producing α cells as terminal (expected baseline)
- FateCompass should recover β as the most likely terminal fate for endocrine progenitors across all tested parameters and kernels
- cellrank_fate_probabilities.csv contains exactly one row per cell population with columns: CellRank fate probability, CellRank terminal-state call, and FateCompass terminal-fate ranking
- Fate probabilities are non-negative and absorption probabilities toward terminal populations are interpretable as fates of the discrete Markov process (consistency check on the estimated probabilities)
- Run FateCompass on the same adata as CellRank (same KNN graph and velocity field) so that terminal-state calls are directly comparable

## Limitations

- Pipeline input parameters must be hand tuned, although detailed tutorials are provided to guide parameter setup
- Trajectories are built on a nearest-neighbor graph in reduced gene-expression space; dimensionality-reduction choices such as UMAP lack biological interpretability and should be treated carefully
- The differentiation-potential (non-velocity) mode depends on prior biological knowledge, which is difficult to provide in some cases such as cancer studies

## Evidence

- [full_text] CellRank fails to call α cells terminal on E15.5 while FateCompass ranks β as most likely fate: [evidence span withheld — anchor status: not_found]
