---
name: random-sampling-permutation-significance-testing
description: Use when judging whether an observed overlap between a candidate regulator
  list and a reference set (e.g. gold circRNA regulators in a Harmonizome interaction
  or co-expression dataset) exceeds chance, via random-sampling permutations.
when_to_use_negative:
- When the candidate feature matrix has not yet been built and NMF clustering has
  not produced a candidate list — run the CRIT pipeline (GET_*.R scripts, NMF.R) first.
- When the question is direct circRNA-RBP binding rather than indirect association
  (PPI, co-expression, shared TFs) — use CLIP database validation instead.
- When no sensible background set (all genes or the 1344-RBP list) can be defined
  for the reference data, as the permutation null is undefined.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: R
  role: Implements the 10,000 random-sampling permutations, empirical p-value computation,
    and FDR correction; data loaded with readr/data.table
- name: CRIT
  role: Supplies the 73 candidate regulators from NMF clustering that are tested against
    the gold regulators
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: Harmonizome
  role: Source of the three evidence datasets (Pathway Commons PPI, MSigDB co-expression,
    TRANSFAC TF targets) in which overlaps are tested
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:479324fada40da5c86fb8dcd1f263d1cb305faadd9603175b03fa1aa9624e8e2
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
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# random-sampling-permutation-significance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assess whether observed overlaps between the 73 CRIT-predicted candidate circRNA regulators and gold-standard regulators in Harmonizome datasets (Pathway Commons PPI, MSigDB co-expression, TRANSFAC TF targets) are significant, by comparing to 10,000 random-sampling permutations against appropriate gene or RBP backgrounds.

## When to use

Use when judging whether an observed overlap between a candidate regulator list and a reference set (e.g. gold circRNA regulators in a Harmonizome interaction or co-expression dataset) exceeds chance, via random-sampling permutations.

## When NOT to use

- When the candidate feature matrix has not yet been built and NMF clustering has not produced a candidate list — run the CRIT pipeline (GET_*.R scripts, NMF.R) first.
- When the question is direct circRNA-RBP binding rather than indirect association (PPI, co-expression, shared TFs) — use CLIP database validation instead.
- When no sensible background set (all genes or the 1344-RBP list) can be defined for the reference data, as the permutation null is undefined.

## Inputs

- Candidate regulator list from CRIT NMF clustering (73 newly predicted candidates)
- gold_regulators.xlsx (gold-standard circRNA regulators)
- Harmonizome Pathway Commons Protein-Protein Interactions dataset
- Harmonizome MSigDB Cancer Gene Co-expression Modules dataset
- Harmonizome TRANSFAC Curated Transcription Factor Targets dataset
- RBP_list.RData (1344 RBPs, used as the RBP background for permutations)

## Outputs

- Physical-interaction enrichment table: candidates with significantly more PPIs with gold regulators at FDR < 0.05 (expected 57/73)
- Co-expression enrichment table: candidates co-expressed with gold regulators at p < 0.0001 (expected RBBP6, RPL17, RPS12, SRSF5, THRAP3)
- Shared-TF enrichment table: candidates sharing TRANSFAC TFs with gold regulators (expected 14 candidates), with gene-background p = 0.0001 and RBP-background p = 0.265
- Three association result tables reporting candidate counts, observed overlaps, and permutation-based p-values/FDR

## How to apply

Load the CRIT NMF candidate list and gold_regulators.xlsx into R with readr/data.table. For each Harmonizome evidence source (Pathway Commons Protein-Protein Interactions, MSigDB Cancer Gene Co-expression Modules, TRANSFAC Curated Transcription Factor Targets), compute the observed overlap between candidates and gold regulators, then generate 10,000 random samples of equal size from the relevant background (all genes, or the 1344-RBP list) and recompute the overlap each time to build a null distribution. Convert each candidate's observed count to an empirical p-value from the null, and apply FDR correction, selecting candidates at FDR < 0.05. For shared-TF analysis, run the permutation separately against the gene background (p = 0.0001) and the RBP background (p = 0.265), reporting both since significance against the gene background but not the RBP background indicates the effect is gene-level rather than RBP-specific. Write result tables reporting candidate counts, observed overlaps, and permutation p-values/FDR to the output directory.

## Related tools

- **R** (Implements the 10,000 random-sampling permutations, empirical p-value computation, and FDR correction; data loaded with readr/data.table)
- **CRIT** (Supplies the 73 candidate regulators from NMF clustering that are tested against the gold regulators) — https://github.com/BioinformaticsSTU/CRIT
- **Harmonizome** (Source of the three evidence datasets (Pathway Commons PPI, MSigDB co-expression, TRANSFAC TF targets) in which overlaps are tested)

## Examples

```
obs <- length(intersect(cand, gold)); p <- mean(replicate(10000, length(intersect(sample(bg, length(cand)), gold)) >= obs))
```

## Evaluation signals

- Physical-interaction analysis recovers 57 of 73 candidates with significantly more interactions with gold regulators at FDR < 0.05
- Co-expression analysis recovers exactly 5 candidates at p < 0.0001: RBBP6, RPL17, RPS12, SRSF5, THRAP3
- Shared-TF analysis yields 14 candidates with gene-background p = 0.0001 but RBP-background p = 0.265; the discrepancy must be reported, not hidden
- With 10,000 permutations, the smallest attainable empirical p-value is bounded near 0.0001; reported p-values below this indicate an error in the null construction
- Each output table reports candidate counts, observed overlap sizes, and permutation-based p-values/FDR, not just significance calls

## Limitations

- Permutation significance depends entirely on the chosen background; against the RBP background the shared-TF signal is not significant (p = 0.265), so conclusions must be qualified by background choice.
- The evidence is indirect (physical interaction, co-expression, shared TFs) — it supports functional connectedness of candidates with gold regulators but does not prove circRNA regulation.
- With 10,000 permutations the empirical p-value resolution is limited to ~1e-4, so very small p-values cannot be distinguished.

## Evidence

- [other] 57 of the 73 newly predicted candidates have significantly more physical interactions with gold circRNA regulators (FDR < 0.05): [evidence span withheld — anchor status: not_found]
