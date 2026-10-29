---
name: approved-drug-annotation-filtering
description: Use when you have essential genes/gene pairs from an in silico knockout
  analysis and a drug-target interaction table (e.g. DrugBank V5) and need to derive
  approved single-drug and combination candidates.
when_to_use_negative:
- The goal is direct docking on the viral proteome — in this workflow drug targets
  arise from network effects on the host metabolome (the virus is modeled only through
  its viral biomass function), not from docking.
- The input drug list includes withdrawn, nutraceutical, or experimental compounds
  that cannot first be filtered to approved status — retaining them invalidates the
  FDA-approved screening premise.
- Both genes of an essential gene pair are targeted by the same drug — such pairs
  must be excluded from combinations and added to the single-drug list, not scored
  as combinations.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3336
- http://edamontology.org/topic_0602
tools:
- name: R
  role: Used to load DrugBank V5 drug-target annotations, filter drug approval statuses,
    map essential genes/gene pairs to targets, and write drug_candidates.csv
- name: DrugBank V5
  role: Source of drug-target interaction pairs and drug status annotations (approved/withdrawn/nutraceutical/experimental)
    used for repositioning
- name: MedDRA
  role: Source of side-effect terms used to compute safety scores for prioritizing
    drug pairs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:8841eae49b30c8906a543c1db501d447e85a375ebb1762037d9a2dc1783e2a9a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sysbiolux/DCcov
  tools:
  - DrugBank V5
  - MedDRA
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3336
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# approved-drug-annotation-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A drug-repositioning filtering skill that restricts a drug-target interaction source (e.g. DrugBank V5) to approved single drugs, maps them onto disease-specific essential genes and gene pairs, and compiles prioritized single drugs (85) and drug combinations (52) as candidate outputs (drug_candidates.csv).

## When to use

Use when you have essential genes/gene pairs from an in silico knockout analysis and a drug-target interaction table (e.g. DrugBank V5) and need to derive approved single-drug and combination candidates.

## When NOT to use

- The goal is direct docking on the viral proteome — in this workflow drug targets arise from network effects on the host metabolome (the virus is modeled only through its viral biomass function), not from docking.
- The input drug list includes withdrawn, nutraceutical, or experimental compounds that cannot first be filtered to approved status — retaining them invalidates the FDA-approved screening premise.
- Both genes of an essential gene pair are targeted by the same drug — such pairs must be excluded from combinations and added to the single-drug list, not scored as combinations.

## Inputs

- Predicted essential single genes and gene pairs from in silico single- and double-gene knockout of context-specific infected lung metabolic models
- DrugBank V5 drug-target interaction table
- Drug approval status annotations (withdrawn / nutraceutical / experimental / approved) from DrugBank V5
- Essentiality scores for gene/drug pairs
- Safety scores from MedDRA side effects (side effects containing the pattern 'toxic')

## Outputs

- List of approved single drugs targeting disease-specific essential genes (85 in DCcov)
- List of prioritized drug combinations targeting essential gene pairs (52 in DCcov)
- drug_candidates.csv containing the compiled candidate set

## How to apply

Load the predicted essential single genes and gene pairs from the single- and double-gene knockout analysis of context-specific infected lung models (rFASTCORMICS models with VBOF), then load the DrugBank V5 drug-target interaction table and discard all drugs annotated as withdrawn, nutraceutical, or experimental so only approved drugs remain. Map each single essential gene to its DrugBank targets to obtain single-target drugs (in DCcov, 8 druggable genes yielded 45 unique drugs). Map each essential gene pair to drug targets, excluding gene pairs for which both genes are targeted by the same drug and adding those drugs to the single-drug list instead. Prioritize drug combinations by keeping drug pairs with more than two essentiality scores and more than one in either the number of gene pairs or safety scores (safety derived from MedDRA side effects containing the pattern 'toxic'). Finally, compile the candidate set (85 single drugs and 52 drug combinations in the DCcov study) and write it to drug_candidates.csv.

## Related tools

- **R** (Used to load DrugBank V5 drug-target annotations, filter drug approval statuses, map essential genes/gene pairs to targets, and write drug_candidates.csv)
- **DrugBank V5** (Source of drug-target interaction pairs and drug status annotations (approved/withdrawn/nutraceutical/experimental) used for repositioning)
- **MedDRA** (Source of side-effect terms used to compute safety scores for prioritizing drug pairs)

## Evaluation signals

- All retained drugs must have approved status: withdrawn, nutraceutical, and experimental annotations are fully absent from the candidate set.
- Single-drug count matches the druggable-gene mapping — in DCcov, 8 druggable essential genes yield 45 unique drugs, contributing to a final 85 single drugs.
- No candidate drug combination corresponds to a gene pair where both genes are targeted by the same drug; such drugs appear only in the single-drug list.
- Every retained drug combination satisfies the filter: more than two essentiality scores AND more than one in either the number of gene pairs or safety scores.
- Final output file drug_candidates.csv contains exactly 85 single drugs and 52 drug combinations as reported for the DCcov pipeline.

## Limitations

- Drug targets are based on network effects on the host metabolome (the virus is modeled only through its biomass function) rather than direct docking on the viral proteome.
- Toxicity of the predicted drugs requires careful assessment in follow-up in vitro and in vivo studies; side-effect filtering used only MedDRA terms containing the pattern 'toxic'.
- Predictions require further tests on other tissues and other optimization functions, plus in vitro and in vivo validation, before considering candidates as potential drugs.

## Evidence

- [abstract] 85 drugs and 52 drug combinations were predicted as promising candidates for COVID-19: "85 drugs and 52 drug combinations were predicted as promising candidates for COVID-19"
- [discussion] Drugs that were withdrawn, nutraceutical, or experimental were discarded from the analysis.: "Drugs that were withdrawn, nutraceutical, or experimental were discarded from the analysis."
- [results] eight genes are druggable by 45 unique drugs (Table S3) from DrugBank: "eight genes are druggable by 45 unique drugs (Table S3) from DrugBank (Wishart et al., 2018)"
- [results] Gene pairs for which both genes are targeted by the same drug were excluded from the candidate list for drug combinations: "Gene pairs for which both genes are targeted by the same drug were excluded from the candidate list for drug combinations and added to the list of single drugs"
- [results] drug pair filtering by essentiality and safety scores: "we filtered by keeping drug pairs with more than two essentiality scores, and more than one in either the number of gene pairs or safety scores"
