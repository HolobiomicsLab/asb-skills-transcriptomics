---
name: benjamini-hochberg-multiple-testing-correction
description: Use when a gene-set activity analysis tests many gene sets simultaneously
  and raw empirical p-values (e.g. ppv L1, ppv Median Exp from rROMA) must be adjusted
  before classifying modules as significant.
when_to_use_negative:
- Only a single gene set / single hypothesis is tested, so no multiplicity correction
  is needed
- The p-values are not independent or positively correlated in ways violating BH assumptions
  and FDR control is not the desired error criterion
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: rROMA
  role: R package that computes ppv p-values from random-gene-set nulls and applies
    the Benjamini and Hochberg correction internally to produce q L1 / q Median Exp
  repo: https://github.com/sysbio-curie/rROMA
- name: irlba
  role: R package used to compute only the first principal component (PC1) for the
    L1 and Median Exp statistics whose ppv values are corrected
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:2349d3da89ed4419bc823c1473f1a605c6c8c1559fa6f47bbacfc52516bcce36
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rROMA
  tools:
  - R
  - irlba
  - rROMA
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# benjamini-hochberg-multiple-testing-correction

## Summary

Apply Benjamini and Hochberg false-discovery-rate correction to the empirical ppv p-values that rROMA computes for each gene set (overdispersion via L1, shift via Median Exp), yielding q-values that control for the many gene sets tested in parallel (e.g. 50 MSigDB Hallmark sets).

## When to use

Use when a gene-set activity analysis tests many gene sets simultaneously and raw empirical p-values (e.g. ppv L1, ppv Median Exp from rROMA) must be adjusted before classifying modules as significant.

## When NOT to use

- Only a single gene set / single hypothesis is tested, so no multiplicity correction is needed
- The p-values are not independent or positively correlated in ways violating BH assumptions and FDR control is not the desired error criterion

## Inputs

- Empirical ppv p-values for overdispersion (ppv L1) per gene set, from comparison of observed L1 to the random-gene-set null
- Empirical ppv p-values for shift (ppv Median Exp) per gene set
- Module list (e.g. 50 MSigDB v6.2 Hallmark gene sets loaded via SelectFromMSIGdb)

## Outputs

- BH-adjusted q values per gene set (q L1, q Median Exp) stored in the rRoma ModuleMatrix table
- Module classification: shifted (ppv Median Exp <= 0.05) or overdispersed (ppv L1 <= 0.05)

## How to apply

For each gene set, rROMA compares the observed L1 statistic (% variance explained by PC1, computed via SVD with irlba) and the Median Exp statistic (median of sample projections on PC1) against null distributions built from nSamples = 100 random gene sets of matched size, producing empirical ppv p-values. Because many modules are tested at once, each ppv is converted to a BH-adjusted q value (q L1 for overdispersion, q Median Exp for shift) so that the family-wise false discovery rate is controlled across the module list. Modules are then classified with the corrected values: a gene set is called overdispersed if ppv L1 <= 0.05 and shifted if ppv Median Exp <= 0.05, and all statistics are assembled into the ModuleMatrix table. The rationale is that raw per-gene-set p-values would inflate significance when scanning large databases such as MSigDB Hallmark or KEGG.

## Related tools

- **rROMA** (R package that computes ppv p-values from random-gene-set nulls and applies the Benjamini and Hochberg correction internally to produce q L1 / q Median Exp) — https://github.com/sysbio-curie/rROMA
- **irlba** (R package used to compute only the first principal component (PC1) for the L1 and Median Exp statistics whose ppv values are corrected)

## Examples

```
rRoma.output <- rRoma.R(MatData, AllHall)  # ModuleMatrix then holds ppv/q L1 and ppv/q Median Exp per Hallmark gene set
```

## Evaluation signals

- ModuleMatrix contains both ppv columns and both BH-adjusted q columns (q L1, q Median Exp) for every tested gene set
- Every q value is >= the corresponding raw ppv value, and q values are within [0, 1]
- At threshold 0.05 on the corrected values, running rRoma.R on MatData with the 50 MSigDB v6.2 Hallmark sets reproduces the article result: 3 of 50 pathways significant (FATTY_ACID_METABOLISM, APICAL_SURFACE, COAGULATION)
- q values are monotonically consistent with the ordering of the raw ppv p-values (BH preserves rank order)
- Null distributions used for the ppv values come from nSamples = 100 random gene sets matched in size, so corrected q values reflect that null size

## Limitations

- The correction is applied to empirical ppv values whose null distributions are approximated over a log-spaced grid of gene set sizes (ApproxSample parameter), so q values inherit approximation error from shared background sets
- rRoma works only with normalized, globally centered expression data; applying the significance/correction step to raw counts invalidates the ppv values being corrected
- The article notes the CF case study lacks statistical power and would require additional transcriptional datasets and testing of several gene set reference databases

## Evidence

- [other] ppv values corrected for multiple testing by the Benjamini and Hochberg method: [evidence span withheld — anchor status: not_found]
- [discussion] uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1 and calculates the p-value by comparing the observed L1 to the null distribution.: "uses a random gene set procedure to generate a null distribution for the L1 amount of variance explained by the PC1 and calculates the p-value by comparing the observed L1 to the null distribution."
- [readme] rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation"
- [discussion] out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION.: "out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION."
