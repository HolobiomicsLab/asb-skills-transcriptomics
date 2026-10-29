---
name: aucell-activity-scoring
description: Use when you need per-cell activity scores for gene-regulatory modules
  (one regulator + its targets) to build a cell-specific GRN from single-cell expression
  data, e.g. in the pySCENIC pipeline on dyngen simulations.
when_to_use_negative:
- Input is bulk transcriptomics rather than single-cell profiles, since AUCell ranking
  and recovery-curve scoring are defined per cell
- Regulon modules (regulator-to-target sets) have not yet been inferred — the arboreto/GRNBoost
  coexpression step must run before AUCell scoring
- The analysis requires motif-based regulon filtering on real data with motif information;
  dyngen simulations provide no motif data, so this benchmarking variant skips that
  step
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: AUCell
  role: Computes per-cell activity scores (area under the recovery curve) for each
    regulon module
- name: pySCENIC
  role: CSNI pipeline whose steps 1 and 5 supply the coexpression modules and AUCell
    scoring for building the cell-specific GRN
- name: arboreto
  role: Infers coexpression modules with stochastic gradient boosting machines to
    define the modules scored by AUCell
- name: dyngen
  role: Generates the simulated single-cell datasets and the ground-truth cellwise
    regulatory networks against which the scores are evaluated
  repo: https://github.com/dynverse/dyngen
- name: dyngen_manuscript
  role: Contains the reproduction scripts for the network-inference use case (analysis/usecase_network_inference)
  repo: https://github.com/dynverse/dyngen_manuscript
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-24152-2
    title: Spearheading future omics analyses using dyngen, a multi-modal simulator
      of single cells
  source_content_hash: sha256:21c23ec9885b803d1b1b4b3679147c08a5422d92ccf6ccb9fe970a8b5010c5da
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dynverse/dyngen
  tools:
  - AUCell
  - arboreto
  - dyngen
  - dyngen_manuscript
  - pySCENIC
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# aucell-activity-scoring

## Summary

Applies AUCell to compute per-cell regulon activity scores, converting coexpression modules into a cell-specific gene regulatory network so that CSNI methods such as pySCENIC can be quantitatively benchmarked against dyngen's ground-truth cellwise regulatory networks.

## When to use

Use when you need per-cell activity scores for gene-regulatory modules (one regulator + its targets) to build a cell-specific GRN from single-cell expression data, e.g. in the pySCENIC pipeline on dyngen simulations.

## When NOT to use

- Input is bulk transcriptomics rather than single-cell profiles, since AUCell ranking and recovery-curve scoring are defined per cell
- Regulon modules (regulator-to-target sets) have not yet been inferred — the arboreto/GRNBoost coexpression step must run before AUCell scoring
- The analysis requires motif-based regulon filtering on real data with motif information; dyngen simulations provide no motif data, so this benchmarking variant skips that step

## Inputs

- Normalized single-cell expression profiles from a dyngen simulation (one of the 42 datasets, 14 backbones x 3 seeds)
- Regulon modules from pySCENIC step 1: groups of one regulator and its targets (top 10 regulators per target gene, inferred with arboreto)
- Ground-truth cell-specific regulatory network from dyngen (regulatory effect of each regulator on its target)

## Outputs

- Per-cell AUCell activity score for each regulon module
- Cell-specific GRN as (cell, regulator, target, score) tuples, top 10,000 interactions per cell
- Per-dataset mean AUROC and mean AUPR over individual cells, bounded in [0, 1]

## How to apply

First infer coexpression modules with pySCENIC step 1 (classical network inference via arboreto's stochastic gradient boosting machines) on the normalized expression matrix, keep the top 10 regulators of every target gene, and group interactions into modules of one regulator and its targets; skip the motif-analysis filtering step because dyngen does not generate motif data. AUCell then ranks all genes per cell and computes the area under the recovery curve of each module's target genes, yielding one activity score per (cell, module). Recombine modules and scores into a cell-specific GRN of (cell, regulator, target, score) tuples. Retain the top 10,000 interactions per cell and compute AUROC and AUPR against the ground-truth cell-specific GRN, averaging scores across all cells per dataset (42 dyngen datasets = 14 predefined backbones x 3 seeds).

## Related tools

- **AUCell** (Computes per-cell activity scores (area under the recovery curve) for each regulon module)
- **pySCENIC** (CSNI pipeline whose steps 1 and 5 supply the coexpression modules and AUCell scoring for building the cell-specific GRN)
- **arboreto** (Infers coexpression modules with stochastic gradient boosting machines to define the modules scored by AUCell)
- **dyngen** (Generates the simulated single-cell datasets and the ground-truth cellwise regulatory networks against which the scores are evaluated) — https://github.com/dynverse/dyngen
- **dyngen_manuscript** (Contains the reproduction scripts for the network-inference use case (analysis/usecase_network_inference)) — https://github.com/dynverse/dyngen_manuscript

## Examples

```
pyscenic aucell expr.loom regulons.dat --output aucs.loom
```

## Evaluation signals

- Each dataset yields a mean AUROC and mean AUPR value bounded in [0, 1] for the SCENIC condition
- The AUCell output is a per-cell score matrix covering all modules (one regulator + its targets) for every cell in the dataset
- Per-cell AUROC/AUPR against the dyngen ground-truth cell-specific GRN averages to scores comparable to those reported in the paper's CSNI benchmark (pySCENIC > LIONESS > SSN)
- All 42 datasets (14 predefined backbones x 3 seeds) produce valid scores without errors
- Top-10,000 interactions per cell are retained before metric computation, matching the paper's protocol

## Limitations

- dyngen does not generate motif data, so the motif-analysis filtering step of the pySCENIC pipeline must be skipped when scoring dyngen simulations
- dyngen assumes cells are well-mixed and independent from each other, so scores reflect simulated dynamics only
- No software version numbers were reported for pySCENIC/AUCell in the benchmark, complicating exact reproducibility

## Evidence

- [results] Per-cell AUCell activity scores and mean AUROC/AUPR evaluation: "For each dataset, we computed the mean Area Under the Receiver Operating Characteristic-curve (AUROC) and Area Under the Precision-Recall curve (AUPR) scores of the individual cells"
- [abstract] dyngen applications include cell-specific regulatory network inference: "three applications: aligning cell developmental trajectories, cell-specific regulatory network inference and estimation of RNA velocity"
- [readme] Reproduction scripts for the network-inference use case: [evidence span withheld — anchor status: not_found]
- [discussion] dyngen simulations evaluate single-cell computational methods: "dyngen’s single-cell simulations can be used to evaluate common single-cell omics computational methods"
- [results] Motivation for quantitative CSNI assessment: "a quantitative assessment of their performance is until now lacking"
