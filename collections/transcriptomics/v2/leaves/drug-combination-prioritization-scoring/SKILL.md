---
name: drug-combination-prioritization-scoring
description: Use when you have predicted essential single genes and gene pairs (e.g.
  from in silico knockout of context-specific metabolic models) and need to convert
  them into candidate single drugs and drug combinations from an approved-drug database
  like DrugBank V5.
when_to_use_negative:
- When the goal is direct docking against the viral proteome — the DCcov predictions
  are based on targets identified by network effects on the host metabolome (the virus
  is only modeled through its biomass function), not on docking.
- When essential genes/gene pairs have not yet been identified; this skill requires
  knockout-derived essentiality results as input.
- When the drug database cannot be restricted to approved drugs (withdrawn, nutraceutical,
  or experimental drugs must be discarded for this workflow).
edam_topics:
- http://edamontology.org/topic_3336
tools:
- name: R
  role: Language used to map essential genes to DrugBank targets, apply the drug and
    drug-pair filters, and compile drug_candidates.csv
- name: DrugBank V5
  role: Drug-target interaction database used to map essential genes to approved drugs
- name: MedDRA
  role: Source of side-effect terms used to compute drug safety scores (only 'toxic'
    pattern side effects selected)
- name: Drug Repurposing Hub
  role: Source of drug mode-of-action annotations used in the repositioning analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:348d1e14ac74a77b03db9e4216065281687c90673a76810645a473945eba32e2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sysbiolux/DCcov
  tools:
  - Drug Repurposing Hub
  - DrugBank V5
  - MedDRA
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3336
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# drug-combination-prioritization-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In silico repositioning of FDA-approved single drugs and drug combinations against disease-specific essential genes/gene pairs by mapping essential genes to DrugBank V5 drug targets and scoring/prioritizing drug pairs on essentiality, gene-pair coverage, and safety (toxicity) metrics, as done in the DCcov COVID-19 study.

## When to use

Use when you have predicted essential single genes and gene pairs (e.g. from in silico knockout of context-specific metabolic models) and need to convert them into candidate single drugs and drug combinations from an approved-drug database like DrugBank V5.

## When NOT to use

- When the goal is direct docking against the viral proteome — the DCcov predictions are based on targets identified by network effects on the host metabolome (the virus is only modeled through its biomass function), not on docking.
- When essential genes/gene pairs have not yet been identified; this skill requires knockout-derived essentiality results as input.
- When the drug database cannot be restricted to approved drugs (withdrawn, nutraceutical, or experimental drugs must be discarded for this workflow).

## Inputs

- Predicted essential single genes and gene pairs from in silico single- and double-gene knockout of context-specific infected lung models
- DrugBank V5 drug-target interaction table
- MedDRA database side-effects data (downloaded 26th May 2020)
- Drug mode-of-action annotations from the Drug Repurposing Hub

## Outputs

- Final candidate list of 85 single drugs and 52 drug combinations written to drug_candidates.csv
- Scored/ranked drug pairs with essentiality scores, number of gene pairs, and safety scores

## How to apply

Load the predicted essential single genes and gene pairs from the single- and double-gene knockout analysis and map each essential gene to its DrugBank V5 targets; in DCcov 8 druggable genes yielded 45 unique drugs. Discard drugs annotated as withdrawn, nutraceutical, or experimental so only approved drugs remain. For combinations, map each essential gene pair to drug targets, excluding gene pairs for which both genes are targeted by the same drug (those drugs are added to the single-drug list instead). Prioritize drug combinations by keeping drug pairs with more than two essentiality scores and more than one in either the number of gene pairs or safety scores; safety scores were derived from MedDRA side effects (only side effects containing the pattern 'toxic') and modes of action from the Drug Repurposing Hub. Compile the final candidate set (85 single drugs and 52 drug combinations in DCcov) and write it to drug_candidates.csv.

## Related tools

- **R** (Language used to map essential genes to DrugBank targets, apply the drug and drug-pair filters, and compile drug_candidates.csv)
- **DrugBank V5** (Drug-target interaction database used to map essential genes to approved drugs)
- **MedDRA** (Source of side-effect terms used to compute drug safety scores (only 'toxic' pattern side effects selected))
- **Drug Repurposing Hub** (Source of drug mode-of-action annotations used in the repositioning analysis)

## Evaluation signals

- Single-drug layer: only druggable essential genes contribute — in DCcov 8 genes yielded 45 unique drugs from DrugBank; a much larger druggable-gene count suggests the approved-drug filter (withdrawn/nutraceutical/experimental discarded) was not applied.
- Combination layer: gene pairs whose two genes are targeted by the same drug must be absent from the combination list and their drugs must appear in the single-drug list instead.
- Drug-pair filter check: retained combinations have more than two essentiality scores and more than one in either the number of gene pairs or safety scores.
- Final counts: the pipeline should reproduce 85 single drugs and 52 drug combinations as promising candidates for COVID-19.
- Output schema: results are compiled into drug_candidates.csv with both single drugs and drug combinations.

## Limitations

- Predictions are based on targets identified by network effects on the host metabolome (the virus is only modeled through its biomass function) rather than direct docking on the viral proteome.
- Drug response curves and IC-50/synergy experiments were not performed, so combination efficacy/synergy is unvalidated.
- Toxicity of predicted drugs requires careful assessment in follow-up in vitro and in vivo studies; predictions need further tests on other tissues and in vitro/in vivo validation before being considered as potential drugs.
- Genes (and their connected drugs) without pathways in Recon3D_01 subSystems were discarded from the tripartite network, so non-metabolic targets are out of scope.

## Evidence
