---
name: single-cell-reprogramming-data-analysis
description: Use when analyzing scRNA-seq time courses of TF-mediated reprogramming
  (e.g., 10x Chromium data with Monocle-annotated trajectory states) to identify and
  rank regulators of successful fate conversion.
when_to_use_negative:
- Input is bulk RNA-seq or lacks single-cell resolution — CellOracle GRN inference
  and state-specific perturbation simulation require single-cell transcriptome (+
  epigenome) data
- The question is discrete cell-type labeling rather than trajectory-based GRN reconfiguration
  and perturbation ranking — use Capybara-style classification instead
- No known reprogramming cocktail exists for validation, since the ranking should
  be sanity-checked against transgene components (Hnf4α-Foxa1 at rank 1)
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CellOracle
  role: Infers state-specific GRNs from single-cell data and simulates in silico TF
    knockout/overexpression to score and rank perturbation effects on cell identity
    transition
  repo: https://github.com/morris-lab/CellOracle
- name: Cell Ranger 6.0.1
  role: Processing of raw 10x Chromium single-cell gene expression data prior to GRN
    inference
- name: Capybara
  role: Unsupervised cell type classification of reprogrammed iEPs (e.g., with Fos
    and Yap added to the Hnf4α-Foxa1 cocktail)
  repo: https://github.com/morris-lab/Capybara
- name: newCloneCalling
  role: CellTag allowlisting and clonal analysis pipeline used to link early network
    states to reprogramming outcome
  repo: https://github.com/morris-lab/newCloneCalling
- name: Python
  role: Execution environment for CellOracle-based GRN inference and perturbation
    simulation workflows
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:c03ad180fe0462c6c44f2f1ecaa4bbf243c30fc7eb1de85f63ddc8b960b73676
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
  - Python
  - newCloneCalling
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# single-cell-reprogramming-data-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproducible single-cell analysis of direct lineage reprogramming (e.g., MEF-to-iEP conversion) that infers state-specific gene regulatory networks with CellOracle and simulates TF perturbations to rank candidate regulators of cell identity conversion.

## When to use

Use when analyzing scRNA-seq time courses of TF-mediated reprogramming (e.g., 10x Chromium data with Monocle-annotated trajectory states) to identify and rank regulators of successful fate conversion.

## When NOT to use

- Input is bulk RNA-seq or lacks single-cell resolution — CellOracle GRN inference and state-specific perturbation simulation require single-cell transcriptome (+ epigenome) data
- The question is discrete cell-type labeling rather than trajectory-based GRN reconfiguration and perturbation ranking — use Capybara-style classification instead
- No known reprogramming cocktail exists for validation, since the ranking should be sanity-checked against transgene components (Hnf4α-Foxa1 at rank 1)

## Inputs

- Processed scRNA-seq datasets for iEP reprogramming replicates (10x Chromium, Cell Ranger v6.0.1 processed, ~48,515 cells)
- Monocle trajectory state annotations (seven states per replicate)
- CellOracle GRN Links models inferred per cell state
- Base-GRN generated from the Mouse sci-ATAC-seq Atlas
- List of active TFs to perturb (180 in this study)

## Outputs

- Ranked TF knockout perturbation score table per replicate (tf_ko_perturbation_ranking.csv)
- Pearson correlation (r) between replicate TF rankings (reported r = 0.72, Figure 3D)
- Scatter plot of replicate KO score/ranking agreement
- List of candidate reprogramming regulators for experimental validation

## How to apply

Load processed scRNA-seq data (e.g., 48,515 iEP reprogramming cells across two biological replicates annotated with seven Monocle states) into CellOracle Oracle objects together with sample- and state-specific GRN (Links) models inferred from single-cell transcriptome and epigenome data (base GRN from the Mouse sci-ATAC-seq Atlas). Run CellOracle's systematic in silico knockout simulation for each active transcription factor (180 TFs in this study) against each state-specific GRN, then aggregate per-state negative perturbation scores by summation across the seven Monocle states to obtain one total KO score per TF per replicate. Rank TFs by summed negative perturbation score, export the ranking (e.g., tf_ko_perturbation_ranking.csv), and compute the Pearson correlation between replicates' rankings as a reproducibility check (target agreement r = 0.72). Validate biologically: the Hnf4α-Foxa1 reprogramming transgene components should occupy rank 1. Experimentally validate top novel candidates (this study identified Id1, Fosb, Fos, Eno1, and Klf4) and combine perturbation predictions with CellTag lineage tracing / clonal analysis to link early network states to conversion outcome.

## Related tools

- **CellOracle** (Infers state-specific GRNs from single-cell data and simulates in silico TF knockout/overexpression to score and rank perturbation effects on cell identity transition) — https://github.com/morris-lab/CellOracle
- **Cell Ranger 6.0.1** (Processing of raw 10x Chromium single-cell gene expression data prior to GRN inference)
- **Capybara** (Unsupervised cell type classification of reprogrammed iEPs (e.g., with Fos and Yap added to the Hnf4α-Foxa1 cocktail)) — https://github.com/morris-lab/Capybara
- **newCloneCalling** (CellTag allowlisting and clonal analysis pipeline used to link early network states to reprogramming outcome) — https://github.com/morris-lab/newCloneCalling
- **Python** (Execution environment for CellOracle-based GRN inference and perturbation simulation workflows)

## Evaluation signals

- Pearson correlation between the two biological replicates' 180-TF perturbation score rankings reaches the reported agreement of r = 0.72
- The Hnf4α-Foxa1 reprogramming transgene components occupy rank 1 in the summed negative perturbation score ranking
- One total KO score per TF is produced by summing negative perturbation scores across all seven Monocle states, with no missing per-state entries
- Ranked table exports as tf_ko_perturbation_ranking.csv containing all 180 active TFs with both scores and ranks per replicate
- Novel top-ranked regulators are confirmed by experimental perturbation (study validated Id1, Fosb, Fos, Eno1, and Klf4)

## Limitations

- Perturbation predictions depend on the quality of inferred state-specific GRNs; base GRN construction requires single-cell epigenome data (Mouse sci-ATAC-seq Atlas here)
- Aggregating negative perturbation scores across all Monocle states into one total KO score may mask state-specific effects of a TF
- Study mainly focused on TFs associated with installing new cell identities; effects on other aspects (e.g., maintenance vs. conversion) require separate interrogation
- In silico simulation ranks candidate regulators but must be followed by experimental validation before conclusions are drawn

## Evidence
