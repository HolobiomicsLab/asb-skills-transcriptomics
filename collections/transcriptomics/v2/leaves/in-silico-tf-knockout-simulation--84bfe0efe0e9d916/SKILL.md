---
name: in-silico-tf-knockout-simulation
description: Use when scRNA-seq data from a reprogramming or differentiation process
  has state-annotated cells and inferred GRNs, and you need to rank candidate TF regulators
  by simulating knockouts before experimental validation.
when_to_use_negative:
- When no sample-specific GRN (Links) model has been inferred for the cell states
  — the simulation depends on state-specific GRN topology and cannot run from raw
  counts alone
- When the goal is experimental TF validation rather than computational prioritization
  — this skill only predicts perturbation effects (e.g., novel regulators like Id1,
  Fosb, Fos, Eno1, Klf4 still require experimental validation)
- When cell states are not annotated into discrete clusters/states — the workflow
  simulates KO per Monocle state and aggregates across states
edam_operation: http://edamontology.org/operation_0242
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CellOracle
  role: Infers state-specific GRNs from scRNA-seq/sci-ATAC-seq data and performs the
    systematic in silico TF knockout/overexpression simulations with vector-field
    projection of identity transitions
  repo: https://github.com/morris-lab/CellOracle
- name: Cell Ranger 6.0.1
  role: Processes 10x Chromium scRNA-seq data into the count matrices used as Oracle
    input
- name: Capybara
  role: Classifies simulated/perturbed cells for fibroblast vs. iEP identity as a
    continuum to quantify perturbation effects on cell identity
  repo: https://github.com/morris-lab/Capybara
- name: igraph
  role: R graph library used by the CellOracle network infrastructure for GRN graph
    operations
  repo: https://github.com/igraph/rigraph
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:4d7a0c0bf3a590e342316674b3745afa229447761e30a907d9180d35c6dae9d1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/morris-lab/CellOracle
  repo_url: https://github.com/morris-lab/CellOracle
  tools:
  - Capybara
  - Cell Ranger 6.0.1
  - CellOracle
  - igraph
  edam_operation: http://edamontology.org/operation_0242
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# in-silico-tf-knockout-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs CellOracle's systematic in silico knockout simulation of transcription factors on state-specific gene regulatory network (GRN) models inferred from single-cell data, ranking TFs by their predicted effect on cell fate conversion. Applied here to rank 180 active TFs during MEF-to-iEP reprogramming and to simulate Fos KO/overexpression effects on cell identity.

## When to use

Use when scRNA-seq data from a reprogramming or differentiation process has state-annotated cells and inferred GRNs, and you need to rank candidate TF regulators by simulating knockouts before experimental validation.

## When NOT to use

- When no sample-specific GRN (Links) model has been inferred for the cell states — the simulation depends on state-specific GRN topology and cannot run from raw counts alone
- When the goal is experimental TF validation rather than computational prioritization — this skill only predicts perturbation effects (e.g., novel regulators like Id1, Fosb, Fos, Eno1, Klf4 still require experimental validation)
- When cell states are not annotated into discrete clusters/states — the workflow simulates KO per Monocle state and aggregates across states

## Inputs

- Processed scRNA-seq count matrices (Cell Ranger v6.0.1-processed, e.g., day-20 Perturb-seq matrix) with cell state annotations (seven Monocle states)
- Sample-specific CellOracle Links (GRN) objects inferred per Monocle state
- Base-GRN from the Mouse sci-ATAC-seq Atlas
- List of active transcription factors (e.g., 180 TFs) to simulate
- Reference cell-type profiles for Capybara identity scoring (fibroblast vs. iEP references)

## Outputs

- Ranked TF knockout perturbation table (tf_ko_perturbation_ranking.csv) per replicate
- Pearson correlation (scatter plot) of TF scores/rankings between biological replicates (expected r = 0.72)
- Vector-field maps and transition-probability plots of simulated identity shifts over the scRNA-seq embedding
- Per-cell and per-condition Capybara identity scores (identity_scores.csv) comparing cocktails such as Hnf4α-Foxa1 ± Fos/Yap1

## How to apply

Load the processed scRNA-seq data (e.g., 48,515 iEP-reprogramming cells annotated with seven Monocle states) into CellOracle Oracle objects together with sample-specific GRN (Links) models inferred for each state. For each Monocle state and each of the active TFs (e.g., 180 in this study), simulate knockout via edge removal (and overexpression via edge reinforcement) on the state-specific GRN, then project perturbed gene expression as vector-field transition maps on the embedding. Aggregate per-state simulated perturbation scores by summing negative perturbation scores across states to obtain one total KO score per TF, rank TFs, and write the table (tf_ko_perturbation_ranking.csv). For replicate reproducibility, compute the Pearson correlation between replicate rankings (expected r = 0.72 here). Optionally score simulated identity shifts with Capybara-style classification against fibroblast and iEP reference profiles, validating that known drivers (Hnf4α-Foxa1 transgene) rank top.

## Related tools

- **CellOracle** (Infers state-specific GRNs from scRNA-seq/sci-ATAC-seq data and performs the systematic in silico TF knockout/overexpression simulations with vector-field projection of identity transitions) — https://github.com/morris-lab/CellOracle
- **Cell Ranger 6.0.1** (Processes 10x Chromium scRNA-seq data into the count matrices used as Oracle input)
- **Capybara** (Classifies simulated/perturbed cells for fibroblast vs. iEP identity as a continuum to quantify perturbation effects on cell identity) — https://github.com/morris-lab/Capybara
- **igraph** (R graph library used by the CellOracle network infrastructure for GRN graph operations) — https://github.com/igraph/rigraph

## Examples

```
pip install celloracle  # then build Oracle objects per Monocle state, load Links GRNs, run the TF KO simulation, and rank by summed negative perturbation score
```

## Evaluation signals

- Known reprogramming drivers validate the ranking: Hnf4α-Foxa1 transgene components should occupy rank 1 of the KO perturbation ranking
- Replicate reproducibility: Pearson correlation between the two biological replicates' TF scores should reach the reported agreement of r = 0.72 (Figure 3D)
- Directionality check: Fos KO simulation should displace cells away from the iEP attractor, while Fos overexpression predicts shifts from early transition to transition clusters and from dead-end to reprogrammed clusters; Fos KO produces the opposite shifts
- Identity scoring: the Hnf4α-Foxa1 + Fos + Yap1 condition should show higher iEP identity scores than the cocktail lacking Fos/Yap1, consistent with reported reprogramming efficiencies
- Output schema check: ranked table contains one total KO score per TF (180 TFs) per replicate, aggregated as summed negative perturbation scores across all seven Monocle states

## Limitations

- Simulations depend on GRN inference quality: base-GRN is derived from the Mouse sci-ATAC-seq Atlas, so regulators absent from the atlas or not expressed as active TFs cannot be ranked
- Predictions are computational and require experimental validation; the paper notes that identified novel regulators (Id1, Fosb, Fos, Eno1, Klf4) were confirmed experimentally after simulation
- Aggregating negative perturbation scores across Monocle states assumes additive state-wise effects, which may obscure state-specific antagonistic effects

## Evidence
