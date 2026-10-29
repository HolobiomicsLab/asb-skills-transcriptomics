---
name: clip-database-rbp-list-overlap-analysis
description: Use when a candidate regulator list (e.g., genes from an NMF cluster
  enriched in gold regulators) must be checked for significant over-representation
  of experimentally determined circRNA-binding RBPs from CLIP-derived resources like
  POSTAR3 or ENCORI.
when_to_use_negative:
- The goal is validating regulators against physical interaction or co-expression
  evidence from Harmonizome (e.g., Pathway Commons PPIs, TRANSFAC TF targets) rather
  than CLIP-derived circRNA-binding lists
- The candidate set is not a gene/RBP symbol list comparable to the 1,344-RBP background,
  e.g., it is still a feature matrix (rho_m, Zp, Zs, Zg, C value) awaiting NMF clustering
- Only two RBP lists are being compared without a defined background universe, making
  contingency-table construction and Fisher testing ill-posed
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CRIT
  role: Produces the candidate regulator list (NMF cluster enriched in gold regulators)
    that is tested for enrichment against CLIP binding lists
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: R
  role: Environment for building 2x2 contingency tables and running one-sided stats::fisher.test
    on each overlap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:cbbaac3f321a986c30b9ce963fa104e25c920d4bdf573939a6016e92293da7a9
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# clip-database-rbp-list-overlap-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates computationally predicted circRNA regulators (e.g., CRIT NMF candidates) by testing their enrichment among known circRNA-binding RBPs from CLIP databases (POSTAR3, ENCORI) using one-sided Fisher's exact tests on 2x2 overlap contingency tables in R.

## When to use

Use when a candidate regulator list (e.g., genes from an NMF cluster enriched in gold regulators) must be checked for significant over-representation of experimentally determined circRNA-binding RBPs from CLIP-derived resources like POSTAR3 or ENCORI.

## When NOT to use

- The goal is validating regulators against physical interaction or co-expression evidence from Harmonizome (e.g., Pathway Commons PPIs, TRANSFAC TF targets) rather than CLIP-derived circRNA-binding lists
- The candidate set is not a gene/RBP symbol list comparable to the 1,344-RBP background, e.g., it is still a feature matrix (rho_m, Zp, Zs, Zg, C value) awaiting NMF clustering
- Only two RBP lists are being compared without a defined background universe, making contingency-table construction and Fisher testing ill-posed

## Inputs

- CRIT candidate regulator gene list (genes from the NMF cluster enriched in gold regulators)
- RBP_list.RData containing the 1,344 candidate RBP background list
- POSTAR3 CLIP-derived circRNA-binding RBP list (112 RBPs)
- ENCORI circRNA-binding RBP lists at five stringency levels (132 down to 27 RBPs)
- Optional all-genes background for the second contingency-table configuration

## Outputs

- 2x2 overlap contingency tables for each binding list x background combination
- Table of one-sided Fisher's exact test p-values (e.g., POSTAR3: 7.7e-5 and 7.8e-30; ENCORI >=5: 3.3e-8 and 5.8e-25)
- Validation verdict stating whether computed p-values match reported values within an order of magnitude with the expected enrichment direction

## How to apply

Load the candidate regulator list (genes from the CRIT NMF cluster enriched in gold regulators) and the RBP background from RBP_list.RData (1,344 RBPs). Retrieve the POSTAR3 CLIP-derived circRNA-binding RBP list (112 RBPs) and the five ENCORI stringency sub-lists (132 RBPs decreasing to 27 at the highest stringency). For each binding list and each background (the 1,344 candidate RBPs, or all genes), construct a 2x2 contingency table of the overlap in R. Run one-sided stats::fisher.test on each table and assemble the resulting p-values; the enrichment direction must show significant over-representation of known circRNA-binding RBPs among the candidates. Validation compares each computed p-value against the reported values (POSTAR3: 7.7e-5 vs 1,344 RBPs and 7.8e-30 vs total genes; ENCORI stringency >=5: 3.3e-8 and 5.8e-25) within an order of magnitude.

## Related tools

- **CRIT** (Produces the candidate regulator list (NMF cluster enriched in gold regulators) that is tested for enrichment against CLIP binding lists) — https://github.com/BioinformaticsSTU/CRIT
- **R** (Environment for building 2x2 contingency tables and running one-sided stats::fisher.test on each overlap)

## Examples

```
R: fisher.test(matrix(c(n11, n12, n21, n22), nrow=2), alternative="greater")  # n11 = |CRIT candidates ∩ POSTAR3 112-RBP circRNA-binding list|
```

## Evaluation signals

- Each computed Fisher exact p-value matches the reported value within an order of magnitude (POSTAR3: 7.7e-5 vs 1,344 RBPs, 7.8e-30 vs total genes; ENCORI stringency >=5: 3.3e-8 and 5.8e-25)
- The enrichment direction is reproduced: significant over-representation of known circRNA-binding RBPs among CRIT candidates
- Overlap counts in each 2x2 table are consistent with the list sizes used (112 POSTAR3 RBPs; ENCORI sub-lists of 132 down to 27 RBPs; 1,344-RBP background)
- All tests are one-sided (alternative = over-representation), not two-sided
- Candidate and binding-list gene identifiers are harmonized (HGNC symbol/Entrez/Ensembl) before table construction, per the identifier conventions in RBP_list.RData

## Limitations

- CLIP-derived lists are resources of known circRNA-binding RBPs (112 POSTAR3 RBPs; ENCORI 132-27 RBPs by stringency), so higher ENCORI stringency shrinks lists and changes p-value scales
- The validation depends on the CRIT candidate set (genes from the NMF cluster enriched in gold regulators); candidates from other clustering choices are not covered by the reported p-values
- Matching within an order of magnitude is the acceptance criterion, accommodating minor differences in gene universe or identifier mapping rather than exact reproduction

## Evidence

- [methods] Run one-sided Fisher's exact tests with R's stats::fisher.test on each contingency table.: [evidence span withheld — anchor status: not_found]
- [results] Fisher's exact tests show CRIT candidate regulators are significantly enriched with circRNA-binding RBPs (POSTAR3: p = 7.7e-5 vs 1,344 RBPs, p = 7.8e-30 vs total genes).: [evidence span withheld — anchor status: not_found]
