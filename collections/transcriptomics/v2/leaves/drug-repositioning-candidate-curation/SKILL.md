---
name: drug-repositioning-candidate-curation
description: Use when a set of essential single genes and gene pairs from FBA knockout
  analysis (e.g. reducing viral biomass without affecting host biomass) must be converted
  into druggable candidates and curated drug combinations.
when_to_use_negative:
- The analysis goal is direct docking or targeting of the viral proteome — this skill's
  candidates are based on targets identified by network effects on the host metabolome,
  as the virus is only modeled through its biomass function
- The essential genes have no DrugBank target annotations (e.g. non-metabolic or non-druggable
  genes) — the mapping step will yield no approved-drug hits
- Drug annotations include withdrawn, nutraceutical, or experimental drugs and cannot
  be filtered — the workflow requires retaining only approved drugs
tools:
- name: R
  role: Implementation language for the drug-target mapping, filtering, and scoring
    steps that compile drug_candidates.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:ef739e7382728218596d307a278f3e3ceb5d8adabf1b1b091d9d701468347649
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sysbiolux/DCcov
  tools:
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# drug-repositioning-candidate-curation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps disease-specific essential single genes and gene pairs (from in silico knockout of context-specific metabolic models) to DrugBank V5 approved-drug targets, then filters and prioritizes the mappings into 85 single drugs and 52 drug combinations for COVID-19 repositioning.

## When to use

Use when a set of essential single genes and gene pairs from FBA knockout analysis (e.g. reducing viral biomass without affecting host biomass) must be converted into druggable candidates and curated drug combinations.

## When NOT to use

- The analysis goal is direct docking or targeting of the viral proteome — this skill's candidates are based on targets identified by network effects on the host metabolome, as the virus is only modeled through its biomass function
- The essential genes have no DrugBank target annotations (e.g. non-metabolic or non-druggable genes) — the mapping step will yield no approved-drug hits
- Drug annotations include withdrawn, nutraceutical, or experimental drugs and cannot be filtered — the workflow requires retaining only approved drugs

## Inputs

- Predicted essential single genes and gene pairs from in silico single- and double-gene knockout of context-specific infected lung models (Recon2.04/Recon3D_01 + VBOF)
- DrugBank V5 drug-target interaction table with drug group annotations (approved/withdrawn/nutraceutical/experimental)
- MedDRA database (downloaded 26th May 2020) side-effect annotations
- Drug Repurposing Hub mode-of-action annotations

## Outputs

- drug_candidates.csv with the final set of 85 single drugs and 52 drug combinations
- Intermediate per-gene and per-gene-pair drug-target mappings with essentiality scores, number of gene pairs, and safety scores

## How to apply

Load the essential single genes and gene pairs from the single- and double-gene knockout analysis of the context-specific infected lung models, and the DrugBank V5 drug-target interaction table, discarding all drugs annotated as withdrawn, nutraceutical, or experimental so only approved drugs remain. Map each single essential gene to its DrugBank targets to get single-target drugs (in this study 8 druggable genes yielded 45 unique drugs, Table S3). Map each essential gene pair to drug targets, excluding gene pairs for which both genes are targeted by the same drug — those drugs are added to the single-drug list instead. Compute essentiality scores, number of gene pairs, and safety scores (side effects extracted from MedDRA, selecting only side effects containing the pattern 'toxic'), then keep drug pairs with more than two essentiality scores and more than one in either the number of gene pairs or safety scores. Compile the final candidate set of 85 single drugs and 52 drug combinations and write it to drug_candidates.csv.

## Related tools

- **R** (Implementation language for the drug-target mapping, filtering, and scoring steps that compile drug_candidates.csv)

## Evaluation signals

- Final output contains exactly 85 single drugs and 52 drug combinations, matching the published result
- Single-gene mapping on the 8 druggable genes yields 45 unique drugs (Table S3) before combination-level curation
- No drugs annotated as withdrawn, nutraceutical, or experimental remain in the candidate list
- Every gene pair for which both genes are targeted by the same drug is absent from the combination list and its drugs appear in the single-drug list
- All retained drug pairs satisfy the filter: more than two essentiality scores, and more than one in either the number of gene pairs or safety scores

## Limitations

- Drug and drug-pair predictions are based on targets identified by network effects on the host metabolome (the virus is only modeled through its biomass function) rather than direct docking on the viral proteome
- No drug response curves, IC-50, or synergy experiments were performed to validate the proposed combinations
- Predictions are limited by the modeled lung cell lines (A549, Calu-3, H1299, NHBE) and require further in vitro/in vivo validation, including careful toxicity assessment, before being considered drug candidates

## Evidence
