---
name: gene-pair-to-drug-combination-mapping
description: Use when you have a list of essential genes and gene pairs (from single-/double-gene
  knockout analysis, e.g. FBA with a VBOF) and need to convert them into approved
  single-drug and drug-combination candidates via DrugBank target mapping.
when_to_use_negative:
- When the goal is direct docking or binding-affinity prediction against viral proteins
  — the identified drugs are based on targets from network effects on the host metabolome
  (the virus is only modeled via its biomass function), not direct docking
- When candidate drug pairs must be ranked by measured potency or synergy — no drug
  response curves, IC-50, or synergy scores are produced by this skill
- When the gene list is not disease-specific essentiality output (e.g. DEG lists without
  knockout essentiality scores) — the essentiality-score and gene-pair filters cannot
  be applied meaningfully
edam_topics:
- http://edamontology.org/topic_0202
- http://edamontology.org/topic_3301
tools:
- name: R
  role: Scripting language used to run the gene-to-drug mapping, apply the drug-pair
    filters, and write drug_candidates.csv
  repo: github.com/sysbiolux/DCcov
- name: DrugBank V5
  role: Source of the drug-target interaction table and drug approval-status annotations
    used for repositioning
- name: MedDRA
  role: Source of drug side effects used for safety scoring of candidate drug pairs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2021.103331
    title: 'DCcov: Repositioning of drugs and drug combinations for SARS-CoV-2 infected
      lung through constraint-based modeling'
  source_content_hash: sha256:4160a07a94edab34f2f6d71abe8159c20bbf3fa15b9baa3ec77fdbd65d7564fb
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0202
  - http://edamontology.org/topic_3301
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# gene-pair-to-drug-combination-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Maps disease-specific essential genes and gene pairs identified by in silico knockout of context-specific metabolic models onto the DrugBank V5 drug-target network, yielding single-drug and drug-combination repositioning candidates (here: 85 single drugs and 52 combinations for COVID-19).

## When to use

Use when you have a list of essential genes and gene pairs (from single-/double-gene knockout analysis, e.g. FBA with a VBOF) and need to convert them into approved single-drug and drug-combination candidates via DrugBank target mapping.

## When NOT to use

- When the goal is direct docking or binding-affinity prediction against viral proteins — the identified drugs are based on targets from network effects on the host metabolome (the virus is only modeled via its biomass function), not direct docking
- When candidate drug pairs must be ranked by measured potency or synergy — no drug response curves, IC-50, or synergy scores are produced by this skill
- When the gene list is not disease-specific essentiality output (e.g. DEG lists without knockout essentiality scores) — the essentiality-score and gene-pair filters cannot be applied meaningfully

## Inputs

- List of disease-specific essential single genes from in silico single-gene knockout of infected context-specific lung models
- List of essential gene pairs with essentiality scores from in silico double-gene knockout
- DrugBank V5 drug-target interaction table with drug approval status annotations
- Drug safety / side-effect data (e.g. MedDRA, downloaded 26th May 2020) and mode-of-action annotations (Drug Repurposing Hub)

## Outputs

- drug_candidates.csv containing 85 single drugs and 52 drug combinations as promising candidates for COVID-19
- Intermediate list of single-target drugs (8 druggable genes yielding 45 unique drugs, Table S3)

## How to apply

Load the predicted essential single genes and gene pairs and the DrugBank V5 drug-target interaction table, discarding drugs annotated as withdrawn, nutraceutical, or experimental so only approved drugs remain. Map each single essential gene to its DrugBank targets to obtain single-target drugs (8 druggable genes yielding 45 unique drugs). For each essential gene pair, map both genes to drug targets; if both genes of a pair are targeted by the same drug, exclude that pair from combinations and add the drug to the single-drug list instead. Prioritize drug combinations by keeping pairs with more than two essentiality scores and more than one in either the number of gene pairs or safety scores (side effects extracted from MedDRA, selecting only side effects containing the pattern 'toxic'). Compile the final candidate set and write it to drug_candidates.csv in R.

## Related tools

- **R** (Scripting language used to run the gene-to-drug mapping, apply the drug-pair filters, and write drug_candidates.csv) — github.com/sysbiolux/DCcov
- **DrugBank V5** (Source of the drug-target interaction table and drug approval-status annotations used for repositioning)
- **MedDRA** (Source of drug side effects used for safety scoring of candidate drug pairs)

## Examples

```
matlab -batch "run('rFASTCORMICS for RNA-seq data/book_example_4_publish_v2.mlx')"  # includes the drug target prediction workflow; final table written to drug_candidates.csv in R
```

## Evaluation signals

- Final drug_candidates.csv contains exactly 85 unique single drugs and 52 drug combinations
- Intermediate check: mapping single essential genes to DrugBank targets yields 8 druggable genes and 45 unique drugs (Table S3)
- No drug annotated as withdrawn, nutraceutical, or experimental appears in the output
- Every retained drug combination comes from a gene pair for which the two genes are not both targeted by the same drug (such drugs were moved to the single-drug list)
- Retained drug pairs satisfy the filter: more than two essentiality scores, and more than one in either the number of gene pairs or safety scores

## Limitations

- Predicted drugs and drug pairs are based on targets identified by network effects on the host metabolome (the virus is only modeled through its biomass function) rather than direct docking on the viral proteome
- Further tests on other tissues and using other optimization functions as well as in vitro and in vivo validations are required before considering any predictions as potential drugs
- Careful assessment of the toxicity of the predicted drugs is needed in follow-up in vitro and in vivo studies; IC-50 and synergy scores for the drug combinations were not determined

## Evidence
