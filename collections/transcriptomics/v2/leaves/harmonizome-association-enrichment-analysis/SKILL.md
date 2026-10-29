---
name: harmonizome-association-enrichment-analysis
description: Use when a list of newly predicted candidate RBPs (e.g., CRIT NMF cluster
  candidates) must be functionally connected to a gold-standard regulator set via
  physical interaction, co-expression, or shared transcription factor modulation evidence.
when_to_use_negative:
- Input is not a candidate regulator list but raw omics data (TCGA CNV/mutation/expression,
  MiOncoCirc, or RNAct files) — run the CRIT feature-collection and NMF.R steps first
- The goal is direct circRNA-RBP binding validation — use CLIP database analysis or
  the RNAct/C value feature instead of indirect Harmonizome associations
- The background distribution cannot be defined (no gene or RBP background lists available)
  — permutation-based p-values require a defined random-sampling background
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3557
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0203
tools:
- name: CRIT
  role: Upstream pipeline whose NMF clustering (NMF.R) produces the 73 candidate regulators
    that this validation step tests against gold regulators
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: R
  role: Implementation language; readr/data.table load candidate and gold regulator
    lists, permutation enrichment computed in R, results written to output directory
- name: Harmonizome
  role: 'Source of the three evidence datasets: Pathway Commons Protein-Protein Interactions,
    MSigDB Cancer Gene Co-expression Modules, and TRANSFAC Curated Transcription Factor
    Targets'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:30d1675ab5d666d6abff2b708753c175f2b6fce5a9ea97cb1f5017834ec88b0e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/BioinformaticsSTU/CRIT
  repo_url: https://github.com/BioinformaticsSTU/CRIT
  tools:
  - CRIT
  - Harmonizome
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3557
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# harmonizome-association-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates newly predicted circRNA regulator candidates by testing their biological association with gold-standard regulators using three Harmonizome datasets (Pathway Commons Protein-Protein Interactions, MSigDB Cancer Gene Co-expression Modules, TRANSFAC Curated Transcription Factor Targets) against 10,000 random-sampling permutation controls.

## When to use

Use when a list of newly predicted candidate RBPs (e.g., CRIT NMF cluster candidates) must be functionally connected to a gold-standard regulator set via physical interaction, co-expression, or shared transcription factor modulation evidence.

## When NOT to use

- Input is not a candidate regulator list but raw omics data (TCGA CNV/mutation/expression, MiOncoCirc, or RNAct files) — run the CRIT feature-collection and NMF.R steps first
- The goal is direct circRNA-RBP binding validation — use CLIP database analysis or the RNAct/C value feature instead of indirect Harmonizome associations
- The background distribution cannot be defined (no gene or RBP background lists available) — permutation-based p-values require a defined random-sampling background

## Inputs

- Candidate regulator list from CRIT NMF clustering (chained input from NMF.R)
- gold_regulators.xlsx — gold-standard circRNA regulators collected from previous research
- Harmonizome Pathway Commons Protein-Protein Interactions dataset
- Harmonizome MSigDB Cancer Gene Co-expression Modules dataset
- Harmonizome TRANSFAC Curated Transcription Factor Targets dataset

## Outputs

- Physical-interaction association table (candidate counts, observed overlaps, permutation p-values/FDR; 57/73 candidates at FDR < 0.05)
- Co-expression association table (5 candidates at p < 0.0001: RBBP6, RPL17, RPS12, SRSF5, THRAP3)
- Shared-TF association table (14 candidates; gene-background p = 0.0001, RBP-background p = 0.265)

## How to apply

Load the candidate regulator list from the CRIT NMF clustering step and the gold_regulators.xlsx file into R using readr/data.table. For physical interactions, intersect candidates and gold regulators in the Harmonizome Pathway Commons Protein-Protein Interactions dataset and assess enrichment by comparing the observed overlap against 10,000 random-sampling permutations, selecting candidates significant at FDR < 0.05 (expect 57 of 73 candidates). For co-expression, query the Harmonizome MSigDB Cancer Gene Co-expression Modules dataset for candidates co-expressed with gold regulators, using 10,000 random-sampling controls against both gene and RBP backgrounds (expect 5 candidates at p < 0.0001: RBBP6, RPL17, RPS12, SRSF5, THRAP3). For shared TFs, use the Harmonizome TRANSFAC Curated Transcription Factor Targets dataset to identify candidates regulated by the same TFs as gold regulators, with 10,000 random-sampling controls against the gene background (p = 0.0001) and RBP background (p = 0.265), expecting 14 candidates. Write the three association result tables, each reporting candidate counts, observed overlaps, and permutation-based p-values/FDR, to the output directory.

## Related tools

- **CRIT** (Upstream pipeline whose NMF clustering (NMF.R) produces the 73 candidate regulators that this validation step tests against gold regulators) — https://github.com/BioinformaticsSTU/CRIT
- **R** (Implementation language; readr/data.table load candidate and gold regulator lists, permutation enrichment computed in R, results written to output directory)
- **Harmonizome** (Source of the three evidence datasets: Pathway Commons Protein-Protein Interactions, MSigDB Cancer Gene Co-expression Modules, and TRANSFAC Curated Transcription Factor Targets)

## Examples

```
Rscript RBP_analysis.R  # after sourcing CRIT_settings.R with ./DATA/input/Harmonizome files (Pathway Commons PPI, MSigDB Co-expression, TRANSFAC) and gold_regulators.xlsx in place
```

## Evaluation signals

- Physical-interaction analysis yields 57 of 73 candidates significant at FDR < 0.05 against gold circRNA regulators
- Co-expression analysis recovers exactly 5 candidates at p < 0.0001: RBBP6, RPL17, RPS12, SRSF5, THRAP3
- Shared-TF analysis identifies 14 candidates, with gene-background p = 0.0001 and RBP-background p = 0.265
- All three result tables report observed overlaps and permutation-based p-values/FDR computed from 10,000 random-sampling permutations
- Three separate output tables are written to the output directory, one per Harmonizome evidence type

## Limitations

- Shared-TF evidence is not significant against the RBP background (p = 0.265), so TF modulation support is background-sensitive and weaker than the gene-background result
- Association evidence is indirect (interaction, co-expression, TF sharing), not direct circRNA-RBP binding; binding validation requires CLIP databases
- Only 57 of 73 candidates pass the physical-interaction FDR < 0.05 threshold, so non-significant candidates may still be valid regulators

## Evidence

- [other] 57 of the 73 newly predicted candidates have significantly more physical interactions with gold circRNA regulators (FDR < 0.05): [evidence span withheld — anchor status: not_found]
