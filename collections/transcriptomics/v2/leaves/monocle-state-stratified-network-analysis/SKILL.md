---
name: monocle-state-stratified-network-analysis
description: Use when scRNA-seq data spanning a dynamic process (e.g., MEF-to-iEP
  reprogramming) has been annotated into Monocle states and you need state-resolved
  GRN models for perturbation simulation or network comparison across states and replicates.
when_to_use_negative:
- Input is raw sequencing data — process through Cell Ranger (v6.0.1 in the reference
  study) first; this skill starts from processed, state-annotated data
- The study question concerns bulk or clonal lineage outcomes (e.g., CellTag-based
  clone analysis) rather than TF-level network perturbation predictions
- No state or pseudotime annotation exists and only a single static cell population
  is available — per-state GRN stratification and cross-state aggregation are not
  meaningful
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0749
tools:
- name: CellOracle
  role: Infers state-specific GRN (Links) models from scRNA-seq/epigenome data, hosts
    Oracle objects, and performs systematic in silico TF KO simulation per Monocle
    state
  repo: https://github.com/morris-lab/CellOracle
- name: Python
  role: Runtime environment for CellOracle (developed with python 3.8, tested with
    python 3.10) and for aggregating scores, ranking TFs, and computing replicate
    Pearson correlations
- name: igraph
  role: Graph library dependency used for network-level analysis of the state-specific
    GRNs
  repo: https://github.com/igraph/rigraph
- name: Cell Ranger
  role: Upstream preprocessing of 10x Chromium scRNA-seq data into the count matrices
    consumed by this skill (v6.0.1 in reference study)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:3ecd666ccab3be340a4db06526630b92d8b3d8a8bb58910d9d332bb532507b73
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
  - Cell Ranger
  - CellOracle
  - Python
  - igraph
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# monocle-state-stratified-network-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratify single-cell reprogramming data by Monocle state (e.g., seven states across 48,515 MEF-to-iEP cells) and infer state-specific gene regulatory networks with CellOracle, so that in silico TF perturbations can be simulated per state and aggregated into reproducible regulator rankings.

## When to use

Use when scRNA-seq data spanning a dynamic process (e.g., MEF-to-iEP reprogramming) has been annotated into Monocle states and you need state-resolved GRN models for perturbation simulation or network comparison across states and replicates.

## When NOT to use

- Input is raw sequencing data — process through Cell Ranger (v6.0.1 in the reference study) first; this skill starts from processed, state-annotated data
- The study question concerns bulk or clonal lineage outcomes (e.g., CellTag-based clone analysis) rather than TF-level network perturbation predictions
- No state or pseudotime annotation exists and only a single static cell population is available — per-state GRN stratification and cross-state aggregation are not meaningful

## Inputs

- Processed scRNA-seq datasets for biological replicates annotated with Monocle states (e.g., 48,515 cells, seven states)
- Sample-specific GRN (Links) models inferred per Monocle state
- Base GRN (e.g., generated from the Mouse sci-ATAC-seq Atlas)
- List of active transcription factors to simulate (e.g., 180 active TFs)

## Outputs

- Per-state CellOracle perturbation simulation scores for each TF
- Aggregated per-TF KO score per replicate (summed negative perturbation scores across states)
- Ranked TF perturbation table (e.g., tf_ko_perturbation_ranking.csv)
- Replicate-agreement scatter plot with Pearson correlation (r = 0.72 in reference study)

## How to apply

Load processed scRNA-seq datasets (e.g., two iEP reprogramming replicates totaling 48,515 cells annotated with seven Monocle states) into CellOracle Oracle objects together with sample-specific GRN (Links) models inferred for each state; the base GRN can be derived from the Mouse sci-ATAC-seq Atlas. For each Monocle state of each replicate, run CellOracle's systematic in silico knockout simulation for each active TF (180 active TFs in the reference study) using the state-specific GRN. Aggregate per-state simulated perturbation scores — for knockout screens, sum negative perturbation scores across the seven Monocle states — to obtain one total KO score per TF per replicate, then rank TFs and export the ranking (e.g., tf_ko_perturbation_ranking.csv). Assess reproducibility by computing the Pearson correlation between replicates' TF scores/rankings and rendering a scatter plot; in the reference study this reached r = 0.72 with the Hnf4α-Foxa1 transgene ranked first. Correct application means the state-specific network structure tracks the biological transition (e.g., fibroblast connections via Mef2a and Klf6 decommissioned in successfully converting cells).

## Related tools

- **CellOracle** (Infers state-specific GRN (Links) models from scRNA-seq/epigenome data, hosts Oracle objects, and performs systematic in silico TF KO simulation per Monocle state) — https://github.com/morris-lab/CellOracle
- **Python** (Runtime environment for CellOracle (developed with python 3.8, tested with python 3.10) and for aggregating scores, ranking TFs, and computing replicate Pearson correlations)
- **igraph** (Graph library dependency used for network-level analysis of the state-specific GRNs) — https://github.com/igraph/rigraph
- **Cell Ranger** (Upstream preprocessing of 10x Chromium scRNA-seq data into the count matrices consumed by this skill (v6.0.1 in reference study))

## Examples

```
sim = oracle.simulate_perturbation(sig_sp=cluster, perturb_condition={tf: 0.0}); ko_scores[state] = sum(sim[negative_scores]); tf_ranking.to_csv('tf_ko_perturbation_ranking.csv')
```

## Evaluation signals

- Hnf4α-Foxa1 reprogramming transgene components occupy rank 1 in the aggregated KO ranking (Figure 3D validation)
- Pearson correlation between the two biological replicates' TF scores/rankings reaches the reported agreement of r = 0.72
- Exactly one total KO score per TF per replicate exists after summing negative perturbation scores across all seven Monocle states
- State-specific networks reflect biology: highly connected fibroblast regulators (e.g., Mef2a, Klf6) lose connectivity in successfully converting cells
- Ranked table (tf_ko_perturbation_ranking.csv) contains all 180 active TFs with finite perturbation scores

## Limitations

- Predictions are in silico and require experimental validation — five predicted regulators (Id1, Fosb, Fos, Eno1, Klf4) were validated experimentally in the reference study, so rankings alone are hypotheses
- Scores depend on the inferred state-specific GRN quality, which in turn depends on the base GRN (Mouse sci-ATAC-seq Atlas) and the accuracy of Monocle state annotation
- Replicate agreement, while substantial (r = 0.72), is not perfect, so single-replicate rankings may misorder candidate regulators

## Evidence
