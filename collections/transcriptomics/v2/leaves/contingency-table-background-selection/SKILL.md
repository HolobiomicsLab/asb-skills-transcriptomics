---
name: contingency-table-background-selection
description: 'Use when validating an over-representation claim: a predicted/clustered
  gene list must be tested for significant overlap with a curated binding list (e.g.,
  112 POSTAR3 CLIP RBPs or ENCORI stringency sub-lists) against a defined background
  set.'
when_to_use_negative:
- The candidate list has no defined background set (e.g., no RBP_list.RData or all-gene
  universe) — enrichment cannot be computed without both margins.
- The validation target is a continuous association (e.g., correlation or interaction
  score) rather than a categorical overlap between two gene sets — a rank/correlation
  test, not a Fisher test, is appropriate.
- You are predicting novel regulators or computing CRIT features (Zp, Zs, Zg, rho_m,
  C value) rather than validating an already-predicted candidate list against external
  binding evidence.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CRIT
  role: Produces the candidate regulator list (NMF cluster enriched in gold regulators)
    that is tested for enrichment against CLIP-derived binding lists
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: R
  role: Runtime for constructing the 2x2 contingency tables and running one-sided
    stats::fisher.test; also supplies RBP_list.RData
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:d78c9e43616ab3cb3a06cf762725aabda00c58bb58d97aefe3deb860dfa8cabf
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
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# contingency-table-background-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs 2x2 overlap contingency tables of a candidate regulator list against CLIP-derived circRNA-binding RBP lists (POSTAR3, ENCORI) under two backgrounds (1,344 candidate RBPs from RBP_list.RData; all genes) and applies one-sided Fisher's exact tests to quantify enrichment of known circRNA-binding RBPs among CRIT predictions.

## When to use

Use when validating an over-representation claim: a predicted/clustered gene list must be tested for significant overlap with a curated binding list (e.g., 112 POSTAR3 CLIP RBPs or ENCORI stringency sub-lists) against a defined background set.

## When NOT to use

- The candidate list has no defined background set (e.g., no RBP_list.RData or all-gene universe) — enrichment cannot be computed without both margins.
- The validation target is a continuous association (e.g., correlation or interaction score) rather than a categorical overlap between two gene sets — a rank/correlation test, not a Fisher test, is appropriate.
- You are predicting novel regulators or computing CRIT features (Zp, Zs, Zg, rho_m, C value) rather than validating an already-predicted candidate list against external binding evidence.

## Inputs

- CRIT candidate regulator list (genes from the NMF cluster enriched in gold regulators)
- 1,344-RBP background list (RBP_list.RData)
- POSTAR3 CLIP-derived circRNA-binding RBP list (112 RBPs)
- ENCORI circRNA-binding RBP sub-lists at five stringency levels (132 down to 27 RBPs)

## Outputs

- 2x2 overlap contingency table per binding-list/background pair
- One-sided Fisher's exact p-values per pair (e.g., POSTAR3: 7.7e-5 and 7.8e-30)
- Assembled p-value comparison table against reported values

## How to apply

Load the candidate regulator list (genes from the NMF cluster enriched in gold regulators) and the 1,344-RBP background from RBP_list.RData. Retrieve the CLIP-derived binding lists: the 112 POSTAR3 circRNA-binding RBPs and the five ENCORI stringency sub-lists (132 RBPs, decreasing to 27). For each binding list, build 2x2 tables twice — once against the 1,344 candidate-RBP background and once against all genes — counting overlap vs non-overlap cells. Run one-sided R stats::fisher.test on each table; the one-sided alternative encodes the hypothesis of over-representation of known circRNA-binding RBPs among candidates. Compare computed p-values to reported values (POSTAR3: 7.7e-5 vs 1,344 RBPs, 7.8e-30 vs total genes; ENCORI stringency >=5: 3.3e-8 and 5.8e-25) and confirm the enrichment direction. Using both a restricted (RBP-only) and a genome-wide background distinguishes enrichment among plausible RNA binders from trivial enrichment among all genes.

## Related tools

- **CRIT** (Produces the candidate regulator list (NMF cluster enriched in gold regulators) that is tested for enrichment against CLIP-derived binding lists) — https://github.com/BioinformaticsSTU/CRIT
- **R** (Runtime for constructing the 2x2 contingency tables and running one-sided stats::fisher.test; also supplies RBP_list.RData)

## Examples

```
fisher.test(matrix(c(length(overlap), n_binding - length(overlap), n_candidates - length(overlap), 1344 - n_binding - n_candidates + length(overlap)), nrow = 2), alternative = "greater")
```

## Evaluation signals

- Each computed one-sided Fisher exact p-value matches the reported value within an order of magnitude (POSTAR3: 7.7e-5 vs 1,344 RBPs and 7.8e-30 vs total genes).
- ENCORI highest-stringency (>=5) p-values reproduce within an order of magnitude (3.3e-8 vs 1,344 RBPs, 5.8e-25 vs total genes).
- Enrichment direction is significant over-representation (odds ratio > 1) of known circRNA-binding RBPs among CRIT candidates, not depletion.
- Contingency table row/column sums equal the chosen background sizes (1,344 candidate RBPs; total gene count) and the binding-list sizes (112 POSTAR3; 132 to 27 ENCORI).
- The candidate list matches the NMF cluster that was enriched in gold regulators, not an arbitrary output cluster.

## Limitations

- Enrichment magnitude depends on background choice: p-values against the 1,344-RBP background (e.g., 7.7e-5) are far weaker than against all genes (7.8e-30), so background selection changes the strength of the claim.
- ENCORI results are stringency-dependent (sub-lists shrink from 132 to 27 RBPs), so the tested hypothesis varies with the chosen cutoff.
- The candidate set is specific to the gold-regulator-enriched NMF cluster; applying the test to other clusters is not validated here.
- CLIP-derived binding lists are database snapshots (POSTAR3, ENCORI); reproducing exact p-values may fail if database versions change.

## Evidence

- [other] Fisher's exact tests show CRIT candidate regulators are significantly enriched with circRNA-binding RBPs (POSTAR3: p = 7.7e-5 vs 1,344 RBPs, p = 7.8e-30 vs total genes).: [evidence span withheld — anchor status: not_found]
