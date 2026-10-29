---
name: drugbank-target-interaction-mapping
description: Use when you have a list of disease-specific essential genes/gene pairs
  from constraint-based knockout analysis and want to identify approved drugs or drug
  pairs targeting them via DrugBank.
when_to_use_negative:
- The virus is being modeled through direct docking on its proteome rather than through
  a viral biomass objective function — these drug targets derive from network effects
  on the host metabolome, not from docking
- No disease-specific essential genes or gene pairs are available yet — the mapping
  step requires upstream in silico knockout results
- The drug of interest is not metabolic or not present in DrugBank V5 with target
  annotations (e.g. nutraceuticals and experimental compounds are explicitly discarded)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3332
- http://edamontology.org/topic_0208
tools:
- name: R
  role: Scripting the DrugBank V5 target mapping, drug filtering, combination prioritization,
    and writing drug_candidates.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:08866958d2047d43749bfd0e1dfdb11739054fff3d8d8746fcc08304bb1c98be
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3332
  - http://edamontology.org/topic_0208
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# drugbank-target-interaction-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps context-specific essential single genes and gene pairs (from in silico knockout of SARS-CoV-2-infected lung metabolic models) onto DrugBank V5 drug-target interactions to propose FDA-approved single drugs and drug combinations for repositioning.

## When to use

Use when you have a list of disease-specific essential genes/gene pairs from constraint-based knockout analysis and want to identify approved drugs or drug pairs targeting them via DrugBank.

## When NOT to use

- The virus is being modeled through direct docking on its proteome rather than through a viral biomass objective function — these drug targets derive from network effects on the host metabolome, not from docking
- No disease-specific essential genes or gene pairs are available yet — the mapping step requires upstream in silico knockout results
- The drug of interest is not metabolic or not present in DrugBank V5 with target annotations (e.g. nutraceuticals and experimental compounds are explicitly discarded)

## Inputs

- List of disease-specific essential single genes and gene pairs from in silico single/double gene knockout of context-specific infected lung metabolic models
- DrugBank V5 drug-target interaction table
- Drug approval status annotations (to discard withdrawn, nutraceutical, experimental drugs)
- MedDRA side-effect data (downloaded 26th May 2020) for safety/toxicity scoring
- Drug Repurposing Hub mode-of-action annotations

## Outputs

- drug_candidates.csv listing 85 single FDA-approved drugs and 52 prioritized drug combinations
- Intermediate list of 45 unique drugs targeting 8 druggable essential genes (Table S3)

## How to apply

Load the predicted essential single genes and gene pairs from the single- and double-gene knockout analysis of the infected lung models, then load the DrugBank V5 drug-target interaction table in R, discarding all drugs annotated as withdrawn, nutraceutical, or experimental so only approved drugs remain. Map each single essential gene to its DrugBank targets: in this study eight genes were druggable, yielding 45 unique drugs (Table S3). For gene pairs, map both genes to drug targets but exclude pairs for which both genes are targeted by the same drug — those drugs are moved to the single-drug list instead. Prioritize combinations by keeping drug pairs with more than two essentiality scores and more than one in either the number of gene pairs or safety scores (safety derived from MedDRA side effects containing the pattern 'toxic'). Compile the final candidate set (here 85 single drugs and 52 drug combinations) into drug_candidates.csv.

## Related tools

- **R** (Scripting the DrugBank V5 target mapping, drug filtering, combination prioritization, and writing drug_candidates.csv)

## Evaluation signals

- Only approved drugs remain after filtering — withdrawn, nutraceutical, and experimental DrugBank entries must be absent
- Drugs targeting both genes of a gene pair appear in the single-drug list, never in the drug-combination list
- Each retained drug pair has more than two essentiality scores and more than one in either gene-pair count or safety score
- Counts match the published result: 8 druggable essential genes, 45 unique single-target drugs, and a final set of 85 drugs and 52 drug combinations
- Output file drug_candidates.csv is produced with the compiled single-drug and combination candidates

## Limitations

- Identified drugs and drug pairs are based on targets from network effects on the host metabolome (the virus is only modeled through its biomass function), not on direct docking to the viral proteome
- Predictions require in vitro and in vivo validation, including careful toxicity assessment, before being considered as potential drugs
- IC-50 and drug-synergy scores for the combinations were not determined in this study
- Only 8 of the essential genes were druggable, constraining the single-drug candidates

## Evidence
