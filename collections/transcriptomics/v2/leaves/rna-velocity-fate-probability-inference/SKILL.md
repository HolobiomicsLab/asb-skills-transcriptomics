---
name: rna-velocity-fate-probability-inference
description: Use when a pre-processed AnnData with cell-type 'clusters' annotations
  and a scVelo RNA velocity field is available and you need per-cell fate probabilities
  or end-fate frequencies toward terminal states (e.g., E15.5 endocrine α, β, ε fates),
  or to benchmark against CellRank.
when_to_use_negative:
- RNA velocity profiles are not robust for the dataset — the differentiation-potential
  mode requires prior biological knowledge that is difficult to provide in some cases,
  such as cancer studies
- The adata lacks cell-type 'clusters' annotations or defined terminal populations,
  which FateCompass requires as input
- The goal is TF activity estimation or differential motif activity (tf_activities.m,
  bootstrapping.m, differential_motif_activity_analysis.m) — those are downstream
  FateCompass steps, not RNA-velocity fate-probability inference
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: FateCompass
  role: Runs the RNA-velocity-driven discrete Markov-chain trajectory sampling (fatecompass.py
    / differentiation_trajectories step) that yields per-cell fate probabilities and
    end-fate frequencies
  repo: github.com/sarajimenez/fatecompass
- name: Scanpy
  role: Suggested basic pre-processing of the E15.5 scRNA-seq AnnData before trajectory
    inference
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scVelo
  role: Computes the RNA velocity field whose vectors direct the edges of the FateCompass
    Markov chain
  repo: https://scvelo.readthedocs.io/en/stable/
- name: CellRank
  role: 'Baseline comparison: velocity kernel on the E15.5 KNN graph and Markov-chain
    estimator of terminal states used to benchmark FateCompass fate probabilities'
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:770f8e42048bdb8e86f78619a99e3b154260603fd3a96919a9ca1a46cbce653c
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# rna-velocity-fate-probability-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infers per-cell fate probabilities and end-fate frequencies by directing a discrete Markov chain on a k=10-neighbor UMAP-space graph of the Bastidas-Ponce E15.5 mouse pancreas data (3,696 cells) with scVelo RNA velocity, then sampling 2,000 stochastic differentiation trajectories from progenitors to terminal endocrine fates (α ~12.4%, β ~77%).

## When to use

Use when a pre-processed AnnData with cell-type 'clusters' annotations and a scVelo RNA velocity field is available and you need per-cell fate probabilities or end-fate frequencies toward terminal states (e.g., E15.5 endocrine α, β, ε fates), or to benchmark against CellRank.

## When NOT to use

- RNA velocity profiles are not robust for the dataset — the differentiation-potential mode requires prior biological knowledge that is difficult to provide in some cases, such as cancer studies
- The adata lacks cell-type 'clusters' annotations or defined terminal populations, which FateCompass requires as input
- The goal is TF activity estimation or differential motif activity (tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m) — those are downstream FateCompass steps, not RNA-velocity fate-probability inference

## Inputs

- Pre-processed AnnData (adata) with cell-type 'clusters' annotations and an scVelo-derived RNA velocity field (e.g., Bastidas-Ponce E15.5 mouse pancreas, 3,696 cells)
- Terminal-state cluster definitions (α, β, ε endocrine fates) and the KNN-graph parameter k=10 neighbors in UMAP space
- FateCompass Python module (fatecompass.py) or Matlab reproducibility script differentiation_trajectories.m, plus the fatecompass.yml conda environment

## Outputs

- fate_probabilities_e155.csv — per-cell fate probabilities for each starting cell and overall end-fate frequencies (α ~12.4%, β ~77%, ε remainder)
- cellrank_fate_probabilities.csv — one row per cell population with CellRank fate probability, CellRank terminal-state call, and FateCompass terminal-fate ranking (optional baseline comparison)

## How to apply

Load the pre-processed Bastidas-Ponce E15.5 AnnData (3,696 cells) with 'clusters' annotations and the scVelo-derived RNA velocity field, using the Scanpy/scVelo pre-processing suggested for FateCompass. Build the nearest-neighbor network in UMAP space with k=10 neighbors and derive transition probabilities from the RNA velocity vectors; FateCompass uses these vectors to direct the edges of the discrete Markov chain (only when velocity profiles are robust — otherwise fall back to the differentiation-potential mode). Execute the RNA-velocity-driven scenario via the FateCompass module (fatecompass.py; reproducibility script differentiation_trajectories) sampling 2,000 stochastic trajectories from progenitor states to terminal endocrine fates. Aggregate the sampled trajectories into per-starting-cell fate probabilities and overall end-fate frequencies, writing fate_probabilities_e155.csv. Optionally run the CellRank baseline (velocity kernel + Markov-chain estimator) on the same adata — including Ductal and Ngn3-low EP populations — and merge per-population calls into cellrank_fate_probabilities.csv, where CellRank is expected to call ε and β terminal but miss α while FateCompass ranks β as the most likely terminal fate.

## Related tools

- **FateCompass** (Runs the RNA-velocity-driven discrete Markov-chain trajectory sampling (fatecompass.py / differentiation_trajectories step) that yields per-cell fate probabilities and end-fate frequencies) — github.com/sarajimenez/fatecompass
- **Scanpy** (Suggested basic pre-processing of the E15.5 scRNA-seq AnnData before trajectory inference) — https://scanpy.readthedocs.io/en/stable/
- **scVelo** (Computes the RNA velocity field whose vectors direct the edges of the FateCompass Markov chain) — https://scvelo.readthedocs.io/en/stable/
- **CellRank** (Baseline comparison: velocity kernel on the E15.5 KNN graph and Markov-chain estimator of terminal states used to benchmark FateCompass fate probabilities)

## Examples

```
conda env create -f fatecompass.yml && jupyter notebook tutorials/fatecompass_basics_rna_velocity.ipynb
```

## Evaluation signals

- fate_probabilities_e155.csv exists and the reproduced end-fate frequencies match the reported distribution (α ~12.4%, β ~77%) within the tolerance implied by stochastic Markov sampling over 2,000 iterations
- Simulated trajectories from progenitor states terminate in final endocrine fates (α, β, ε), and β is recovered as the most likely terminal fate for endocrine progenitors
- CellRank baseline on the same E15.5 data (with Ductal and Ngn3-low EP) identifies ε and β as terminal but fails to identify the glucagon-producing α cells, whereas FateCompass ranks β top across all tested parameters and kernels — this divergence is visible in cellrank_fate_probabilities.csv
- Markov transition probabilities on the k=10 UMAP-space neighbor network have edges directed consistently with the scVelo RNA velocity vectors, forming a valid discrete Markov chain
- Per-cell fate probabilities aggregate cleanly by the 'clusters' annotation so each starting cell receives a probability distribution over terminal fates

## Limitations

- Relies on RNA velocity profiles being robust; the differentiation-potential alternative depends on prior biological knowledge, which is difficult to provide in some cases such as cancer studies
- Trajectories are built on a nearest-neighbor graph in a reduced gene expression space, and commonly used dimensionality reduction methods such as UMAP lack biological interpretability
- The pipeline depends on input parameters that must be hand tuned (detailed tutorials are provided to guide parameter setup)
- Stochastic sampling means end-fate frequencies vary run to run; the reported α ~12.4% and β ~77% come from 2,000 iterations
- Reproducibility scripts for the pancreatic datasets are implemented in Matlab (differentiation_trajectories.m), while the reusable module is the Python file fatecompass.py

## Evidence
