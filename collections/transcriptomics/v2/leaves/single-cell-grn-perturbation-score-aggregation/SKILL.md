---
name: single-cell-grn-perturbation-score-aggregation
description: Use when scRNA-seq data has been clustered into pseudotime/Monocle states,
  CellOracle GRNs are inferred per state, and you need to rank TFs by predicted effect
  of knockout on a fate-transition process.
when_to_use_negative:
- When the goal is to identify cis-regulatory elements or build the base GRN from
  epigenomic data (e.g. the Mouse sci-ATAC-seq Atlas base-GRN) rather than to score
  perturbation effects on an already-inferred GRN
- When the input lacks a biologically expected perturbation control (like the reprogramming
  transgene) with which to sanity-check the top of the ranking
- When no replicate structure exists — the ranking's reproducibility (Pearson r) cannot
  be assessed from a single sample
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CellOracle
  role: Runs the systematic in silico knockout simulation on state-specific GRN objects
    and produces the per-TF perturbation scores that are aggregated and ranked
  repo: https://github.com/morris-lab/CellOracle
- name: Python
  role: Execution environment for CellOracle (developed and tested with python 3.8/3.10)
    and for aggregating scores, ranking TFs, computing the Pearson correlation, and
    plotting replicate agreement
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:7425f2659dacbf29d3b76224f75a745bf815478e2d1401ae1586157ae5fd1444
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
  - CellOracle
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Single-cell GRN in silico perturbation score aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Systematic in silico knockout simulation of active transcription factors using CellOracle on state-specific GRNs (Links), followed by aggregation of per-state perturbation scores into a single ranked TF list, with reproducibility checked between biological replicates.

## When to use

Use when scRNA-seq data has been clustered into pseudotime/Monocle states, CellOracle GRNs are inferred per state, and you need to rank TFs by predicted effect of knockout on a fate-transition process.

## When NOT to use

- When the goal is to identify cis-regulatory elements or build the base GRN from epigenomic data (e.g. the Mouse sci-ATAC-seq Atlas base-GRN) rather than to score perturbation effects on an already-inferred GRN
- When the input lacks a biologically expected perturbation control (like the reprogramming transgene) with which to sanity-check the top of the ranking
- When no replicate structure exists — the ranking's reproducibility (Pearson r) cannot be assessed from a single sample

## Inputs

- Processed scRNA-seq datasets for two biological replicates (48,515 cells) annotated with seven Monocle states
- State-specific GRN (Links) models inferred per replicate and per Monocle state
- List of 180 active transcription factors to simulate
- Known positive-control perturbation (Hnf4α-Foxa1 transgene components)

## Outputs

- tf_ko_perturbation_ranking.csv: 180 TFs ranked by summed negative perturbation score per replicate
- Pearson correlation coefficient between replicate TF scores (expected r = 0.72) and a replicate-agreement scatter plot
- Top-ranked candidate regulators for experimental validation (e.g. Id1, Fosb, Fos, Eno1, Klf4)

## How to apply

Load processed scRNA-seq datasets (here 48,515 cells from two iEP reprogramming replicates annotated with seven Monocle states) into CellOracle Oracle objects together with the sample-specific, state-specific GRN (Links) models. For each Monocle state of each replicate, run CellOracle's systematic in silico knockout simulation for each of the 180 active TFs using that state's GRN. Aggregate per-state results by summing the negative perturbation scores across the seven Monocle states to yield one total KO score per TF per replicate. Rank the TFs by summed negative perturbation score (e.g. write tf_ko_perturbation_ranking.csv) and compute the Pearson correlation between replicates' scores to quantify reproducibility. Validate the ranking biologically: the Hnf4α-Foxa1 reprogramming transgene components should occupy rank 1, and replicate agreement should reach r = 0.72.

## Related tools

- **CellOracle** (Runs the systematic in silico knockout simulation on state-specific GRN objects and produces the per-TF perturbation scores that are aggregated and ranked) — https://github.com/morris-lab/CellOracle
- **Python** (Execution environment for CellOracle (developed and tested with python 3.8/3.10) and for aggregating scores, ranking TFs, computing the Pearson correlation, and plotting replicate agreement)

## Examples

```
pip install celloracle
```

## Evaluation signals

- Hnf4α-Foxa1 transgene components occupy rank 1 in the KO perturbation ranking, as expected for the reprogramming-driving factors
- Pearson correlation between the two biological replicates' TF scores reaches approximately r = 0.72 (Figure 3D)
- Top-ranked TFs are shared between independent biological replicates
- Every one of the 180 active TFs yields a perturbation score in each of the seven Monocle states before summation, so no TF or state is silently dropped
- Experimentally validated new regulators (Id1, Fosb, Fos, Eno1, Klf4) are recoverable among the top-ranked candidates

## Limitations

- Predicted perturbation scores are computational simulations of cell identity transition and require subsequent experimental validation; the study validated only a subset (Id1, Fosb, Fos, Eno1, Klf4)
- Scores depend on the quality of state-specific GRN inference, which itself integrates scRNA-seq with epigenomic base-GRN information and can reflect network reconfiguration differences between states
- The study mainly focused on TFs associated with installing new cell identities, so rankings may not capture regulators of other aspects of reprogramming

## Evidence
