---
name: rbp-candidate-validation-against-gold-standard
description: Use when an NMF (or other) clustering of candidate RBPs has produced
  cluster assignments and you must test whether any single cluster is significantly
  enriched for known gold-standard circRNA regulators before accepting the clustering
  strategy.
when_to_use_negative:
- You have not yet generated NMF cluster assignments (NMFgroup_XXclusterYYrun.txt)
  from the CRIT feature matrix — run NMF.R first.
- Your candidate set is not RBPs or lacks a curated gold-standard regulator list to
  test enrichment against.
- You need de novo prediction of regulators rather than validation of an existing
  clustering against known regulators (use supervised-learning extensions, noted as
  unimplemented in the paper).
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_0203
tools:
- name: CRIT
  role: Pipeline whose NMF-based 'Validate candidates' component performs the gold-regulator
    enrichment testing of clusters
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: R
  role: Implementation environment; Fisher's exact tests run with R stats, survival
    log-rank tests via the R 'survival' package
- name: NMF
  role: R package used in NMF.R to cluster candidate RBPs into the groups that are
    then validated against gold regulators
  repo: https://cran.r-project.org/web/packages/NMF/NMF.pdf
- name: RBP_analysis.R
  role: CRIT script for candidate RBP validation against the gold-regulator list
  repo: https://github.com/BioinformaticsSTU/CRIT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:89eaa11d21b92f7d3b882dfc8beb461522abba316eaa38fda936b72ab4be8fa7
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
  - NMF
  - R
  - RBP_analysis.R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# rbp-candidate-validation-against-gold-standard

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validates NMF-derived RBP clusters from the CRIT pipeline by Fisher's exact tests of cluster membership against gold-standard circRNA regulator lists, demonstrating that the clustering captures known regulators (cluster 23: 17/90 gold regulators, p=0.00012).

## When to use

Use when an NMF (or other) clustering of candidate RBPs has produced cluster assignments and you must test whether any single cluster is significantly enriched for known gold-standard circRNA regulators before accepting the clustering strategy.

## When NOT to use

- You have not yet generated NMF cluster assignments (NMFgroup_XXclusterYYrun.txt) from the CRIT feature matrix — run NMF.R first.
- Your candidate set is not RBPs or lacks a curated gold-standard regulator list to test enrichment against.
- You need de novo prediction of regulators rather than validation of an existing clustering against known regulators (use supervised-learning extensions, noted as unimplemented in the paper).

## Inputs

- MATRIX.txt — merged candidate RBP feature matrix (ρm, Zp, Zs, Zg, C value features)
- RBP_list.RData — list of 1344 candidate RBPs
- NMFgroup_36clusterYYrun.txt — cluster assignments per run (XX=36 clusters, YY=run number)
- gold_regulators.xlsx — gold-standard circRNA regulators collected from previous research
- Li et al. 103-RBP circmCherry screen list (external benchmark)
- Curated list of 19 established circRNA regulators (external benchmark)

## Outputs

- Identification of the gold-regulator-enriched cluster (expected: cluster 23 with 90 RBPs, 17 of them gold regulators)
- Fisher's exact test p-values for gold regulators (0.00012), circmCherry screen overlap (13/90, p=0.00201), and curated regulators (6/19, p=0.00112)
- Validated candidate RBP set carried forward for Harmonizome evidence collection and downstream validation

## How to apply

Run the CRIT NMF step (NMF.R) on the merged candidate feature matrix (MATRIX.txt) for the 1344 candidate RBPs in RBP_list.RData, selecting rank R=36 by local maxima of the cophenetic correlation coefficient (CPCC) and dispersion (DC) indices, and write each run's assignments to NMFgroup_36clusterYYrun.txt using the XXclusterYYrun naming convention. Then, in R, perform Fisher's exact tests (stats package, e.g. via RBP_analysis.R) comparing each cluster's membership against gold_regulators.xlsx to identify the gold-regulator-enriched cluster. Repeat the overlap tests against external benchmark sets: Li et al.'s 103-RBP circmCherry screen and a 19-member curated set of established regulators. The clustering is validated only if the enriched cluster's size, gold-regulator count, and all three Fisher's exact p-values reproduce the reported values.

## Related tools

- **CRIT** (Pipeline whose NMF-based 'Validate candidates' component performs the gold-regulator enrichment testing of clusters) — https://github.com/BioinformaticsSTU/CRIT
- **R** (Implementation environment; Fisher's exact tests run with R stats, survival log-rank tests via the R 'survival' package)
- **NMF** (R package used in NMF.R to cluster candidate RBPs into the groups that are then validated against gold regulators) — https://cran.r-project.org/web/packages/NMF/NMF.pdf
- **RBP_analysis.R** (CRIT script for candidate RBP validation against the gold-regulator list) — https://github.com/BioinformaticsSTU/CRIT

## Examples

```
Rscript NMF.R  # then Rscript RBP_analysis.R  # Fisher's exact test: fisher.test(matrix(c(17,73,?,?),nrow=2)) comparing cluster 23 membership vs gold_regulators.xlsx
```

## Evaluation signals

- The enriched cluster contains exactly 90 RBPs, of which 17 are gold-standard circRNA regulators.
- Fisher's exact test against gold_regulators.xlsx yields p = 0.00012.
- Overlap with Li et al.'s 103-RBP circmCherry screen is 13/90 RBPs with p = 0.00201.
- Overlap with the 19 curated established regulators is 6/19 with p = 0.00112.
- Cluster assignment files follow the XXclusterYYrun naming convention (e.g. NMFgroup_36clusterYYrun.txt) for the rank R=36 chosen by CPCC and dispersion (DC) local maxima.

## Limitations

- Validation depends on the completeness/quality of gold_regulators.xlsx, which is collected from previous research and assays based on literature; incomplete gold lists could mask or bias enrichment.
- The rank R=36 choice rests on CPCC and dispersion (DC) local maxima; a different rank could change which cluster is enriched and the reported p-values.
- Generalization of CRIT beyond cancers (e.g. to neurodegenerative disease) is untested, so gold-regulator validation in other disease contexts is not established.

## Evidence

- [methods] Perform Fisher's exact tests (R stats) comparing cluster membership against gold_regulators.xlsx to identify the gold-regulator-enriched cluster, expecting cluster 23 with 90 RBPs of which 17 are gold regulators (p=0.00012).: [evidence span withheld — anchor status: not_found]
- [results] NMF clustering yielded cluster 23 containing 90 RBPs, of which 17 are gold-standard circRNA regulators, a significant enrichment (Fisher's exact test p = 0.00012): [evidence span withheld — anchor status: not_found]
- [methods] Run the NMF algorithm (R 'NMF' package, NMF.R) on the features matrix with rank R=36 selected by local maxima of the cophenetic correlation coefficient (CPCC) and dispersion (DC) indices.: [evidence span withheld — anchor status: not_found]
