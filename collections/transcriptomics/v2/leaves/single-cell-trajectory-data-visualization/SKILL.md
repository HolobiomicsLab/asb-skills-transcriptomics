---
name: single-cell-trajectory-data-visualization
description: Use when a pre-processed scRNA-seq anndata object with 'clusters' annotations
  and an scVelo RNA velocity field must be turned into a directed differentiation
  trajectory with per-population fate probabilities and terminal-state visualizations.
when_to_use_negative:
- The input is bulk RNA-seq or lacks cell-type ('clusters') annotations and an RNA
  velocity field, since neither CellRank's velocity kernel nor FateCompass's velocity-driven
  Markov chain can be built
- The goal is time- and lineage-specific transcription-factor activity estimation
  (the FateCompass linear-model/TF-dynamics workflow) rather than trajectory and terminal-fate
  visualization
- Only smooth pseudotime ordering is needed without directed, velocity-informed transition
  probabilities toward terminal states
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FateCompass
  role: Estimates transition probabilities from RNA velocity via a discrete Markov
    process and ranks most likely terminal fates; provided as fatecompass.py
  repo: github.com/sarajimenez/fatecompass
- name: CellRank
  role: 'Baseline method: velocity kernel on the KNN graph plus Markov-chain estimator
    to compute absorption probabilities and call terminal states'
- name: Scanpy
  role: Suggested basic pre-processing of the anndata object before trajectory estimation
- name: scVelo
  role: Derives the RNA velocity field used to direct the edges of the trajectory
    Markov chain
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:d7dffa4af9e16dd768392cca4a53016afa5f94291f3c2a3ce54eb673031c0862
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
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# single-cell-trajectory-data-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize single-cell differentiation trajectories and terminal-fate assignments by combining RNA-velocity-driven transition probabilities (FateCompass / CellRank) with cell-type annotations, as applied to the E15.5 mouse pancreas endocrine differentiation dataset.

## When to use

Use when a pre-processed scRNA-seq anndata object with 'clusters' annotations and an scVelo RNA velocity field must be turned into a directed differentiation trajectory with per-population fate probabilities and terminal-state visualizations.

## When NOT to use

- The input is bulk RNA-seq or lacks cell-type ('clusters') annotations and an RNA velocity field, since neither CellRank's velocity kernel nor FateCompass's velocity-driven Markov chain can be built
- The goal is time- and lineage-specific transcription-factor activity estimation (the FateCompass linear-model/TF-dynamics workflow) rather than trajectory and terminal-fate visualization
- Only smooth pseudotime ordering is needed without directed, velocity-informed transition probabilities toward terminal states

## Inputs

- Pre-processed anndata object (.h5ad) with cell-type annotations ('clusters', e.g., Ductal, Ngn3-low EP, α, β, ε) and scVelo RNA velocity field
- KNN graph of the single-cell dataset in a reduced gene expression space
- Optionally, a TF binding-sites matrix (rows = expressed genes, columns = TFs/motifs) for the FateCompass linear model of gene regulation
- Python environment created from the provided fatecompass.yml conda file

## Outputs

- Absorption/fate probabilities toward each candidate terminal population from CellRank's Markov-chain estimator
- FateCompass transition probabilities and ranked most likely terminal fates
- Comparison table cellrank_fate_probabilities.csv (one row per cell population; columns: CellRank fate probability, CellRank terminal-state call, FateCompass terminal-fate ranking)
- Trajectory/fate visualizations over the annotated cell populations

## How to apply

Load the pre-processed E15.5 anndata object containing cell-type annotations (including Ductal and Ngn3-low EP clusters) and the scVelo-derived RNA velocity field, pre-processed with Scanpy/scVelo as suggested for FateCompass. Build a directed transition structure on the KNN graph: run CellRank's velocity kernel plus Markov-chain estimator to record which populations are called terminal, and run FateCompass (fatecompass.py) in RNA-velocity mode, which uses velocity to direct the edges of a discrete Markov chain to estimate transition probabilities and rank terminal fates. Aggregate per-cell-type fate probabilities and terminal-state calls from both methods into one tabular comparison (cellrank_fate_probabilities.csv) with columns for CellRank fate probability, CellRank terminal-state call, and FateCompass terminal-fate ranking, suitable for trajectory visualization. Rationale: trajectories are outlined as a discrete Markov process on a nearest-neighbor graph in gene expression space, so visualizations should be interpreted relative to that graph rather than UMAP geometry alone.

## Related tools

- **FateCompass** (Estimates transition probabilities from RNA velocity via a discrete Markov process and ranks most likely terminal fates; provided as fatecompass.py) — github.com/sarajimenez/fatecompass
- **CellRank** (Baseline method: velocity kernel on the KNN graph plus Markov-chain estimator to compute absorption probabilities and call terminal states)
- **Scanpy** (Suggested basic pre-processing of the anndata object before trajectory estimation)
- **scVelo** (Derives the RNA velocity field used to direct the edges of the trajectory Markov chain)

## Examples

```
conda env create -f fatecompass.yml && jupyter notebook tutorials/fatecompass_basics_rna_velocity.ipynb
```

## Evaluation signals

- CellRank baseline on E15.5 identifies ε and β cells as terminal hormone-producing fates but does NOT identify the glucagon-producing α cells as terminal (expected baseline call)
- FateCompass recovers the β fate as the most likely terminal fate for endocrine progenitors across all tested parameters and kernels
- cellrank_fate_probabilities.csv contains exactly one row per cell population with populated columns for CellRank fate probability, CellRank terminal-state call, and FateCompass terminal-fate ranking
- Per-cell-type fate probabilities agree with cluster annotations when aggregated (highest probability toward the population the progenitors are known to differentiate into)
- Trajectory outputs are consistent with the published results summarized in the FateCompass Shiny App

## Limitations

- Commonly used dimensionality reduction methods such as UMAP lack biological interpretability, so trajectories built on the nearest-neighbor graph in reduced space should be treated carefully during visualization
- The FateCompass pipeline depends on input parameters that must be hand tuned, although detailed tutorials are provided
- Differentiation-potential mode of trajectory inference depends on prior biological knowledge, which is difficult to provide in some cases such as cancer studies

## Evidence

- [methods] FateCompass implemented in python with fatecompass.py module: [evidence span withheld — anchor status: not_found]
