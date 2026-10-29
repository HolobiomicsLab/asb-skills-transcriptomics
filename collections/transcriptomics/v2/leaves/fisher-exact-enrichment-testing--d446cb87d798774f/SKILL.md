---
name: fisher-exact-enrichment-testing
description: Use when validating a predicted gene/RBP set (e.g., an NMF cluster from
  CRIT's MATRIX.txt features) against a curated reference list such as gold_regulators.xlsx
  or POSTAR3/ENCORI CLIP-derived circRNA-binding RBPs via 2x2 overlap tests.
when_to_use_negative:
- The input is already a per-gene statistic table needing multiple-testing correction
  rather than a set-vs-set overlap test
- There is no independently curated reference set (gold regulators, circmCherry screen,
  or CLIP binding lists) to test enrichment against, making the 2x2 table ill-defined
- The question is a continuous association between gene expression and circRNA levels
  (use the rho_m Spearman correlation feature) rather than categorical set membership
edam_operation: http://edamontology.org/operation_0493
edam_topics:
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_0203
tools:
- name: R
  role: stats::fisher.test computes the one-sided exact tests on 2x2 contingency tables
    assembled from cluster assignments and reference lists
- name: CRIT
  role: provides RBP_analysis.R (candidate RBPs validation) which runs the enrichment
    tests, plus the NMF clustering (NMF.R) whose cluster output is being tested
  repo: https://github.com/BioinformaticsSTU/CRIT
- name: NMF
  role: R package used upstream to cluster the 1,344 candidate RBPs (rank R=36); its
    cluster membership is the tested set
  repo: https://cran.r-project.org/web/packages/NMF/NMF.pdf
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.omtn.2022.10.015
    title: 'CRIT: Identifying RNA-binding protein regulator in circRNA life cycle
      via non-negative matrix factorization'
  source_content_hash: sha256:5e3d20b7501e30f4387dbebbcc1556e2a32b2760099b0e38cd9303a4e1327bea
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
  edam_operation: http://edamontology.org/operation_0493
  edam_topics:
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# fisher-exact-enrichment-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A set-overlap validation step of the CRIT pipeline that tests whether NMF-derived RBP clusters and candidate circRNA-regulator lists are significantly enriched for gold-standard regulators and CLIP-derived circRNA-binding RBPs, using one-sided Fisher's exact tests on 2x2 contingency tables computed in R (stats::fisher.test).

## When to use

Use when validating a predicted gene/RBP set (e.g., an NMF cluster from CRIT's MATRIX.txt features) against a curated reference list such as gold_regulators.xlsx or POSTAR3/ENCORI CLIP-derived circRNA-binding RBPs via 2x2 overlap tests.

## When NOT to use

- The input is already a per-gene statistic table needing multiple-testing correction rather than a set-vs-set overlap test
- There is no independently curated reference set (gold regulators, circmCherry screen, or CLIP binding lists) to test enrichment against, making the 2x2 table ill-defined
- The question is a continuous association between gene expression and circRNA levels (use the rho_m Spearman correlation feature) rather than categorical set membership

## Inputs

- NMF cluster assignment files NMFgroup_XXclusterYYrun.txt (e.g., XX=36) derived from MATRIX.txt, the merged CRIT feature matrix of rho_m, Zp, Zs, Zg, and C value features
- gold_regulators.xlsx — curated list of gold-standard circRNA regulators collected from previous research
- RBP_list.RData — list of 1,344 candidate RBPs used as a background
- POSTAR3 CLIP-derived circRNA-binding RBP list (112 RBPs)
- ENCORI circRNA-binding RBP sub-lists at five stringency levels (132, 111, ... down to 27 RBPs)

## Outputs

- Identification of the gold-regulator-enriched NMF cluster (cluster 23: 90 RBPs, 17 gold regulators, p=0.00012)
- 2x2 overlap contingency tables for each candidate-list vs reference-list comparison
- Table of one-sided Fisher's exact p-values for each background (1,344 candidate RBPs and all genes), e.g., POSTAR3 7.7e-5 / 7.8e-30

## How to apply

First, load the NMF cluster assignments written to NMFgroup_36clusterYYrun.txt (rank R=36 selected by local maxima of the cophenetic correlation coefficient and dispersion indices) and identify the cluster enriched in gold regulators — expected to be cluster 23 with 90 RBPs including 17 gold-standard circRNA regulators (Fisher p=0.00012). For each comparison, build a 2x2 contingency table crossing cluster/candidate membership against the reference set over an explicit background (the 1,344 candidate RBPs from RBP_list.RData, or all genes), then run a one-sided stats::fisher.test in R, as scripted in RBP_analysis.R. Reproduce the literature overlaps: Li et al.'s 103-RBP circmCherry screen (expected 13/90, p=0.00201) and 19 curated established regulators (6/19, p=0.00112). For CLIP-database validation, test the CRIT candidate list against POSTAR3's 112 circRNA-binding RBPs and five ENCORI stringency sub-lists (132 decreasing to 27 RBPs), computing p-values against both the 1,344-RBP and all-gene backgrounds (POSTAR3 expected 7.7e-5 and 7.8e-30; ENCORI >=5 stringency 3.3e-8 and 5.8e-25). The rationale is that significant over-representation of independently curated circRNA regulators and CLIP-observed binders supports that CRIT's NMF clustering captures authentic circRNA regulators.

## Related tools

- **R** (stats::fisher.test computes the one-sided exact tests on 2x2 contingency tables assembled from cluster assignments and reference lists)
- **CRIT** (provides RBP_analysis.R (candidate RBPs validation) which runs the enrichment tests, plus the NMF clustering (NMF.R) whose cluster output is being tested) — https://github.com/BioinformaticsSTU/CRIT
- **NMF** (R package used upstream to cluster the 1,344 candidate RBPs (rank R=36); its cluster membership is the tested set) — https://cran.r-project.org/web/packages/NMF/NMF.pdf

## Examples

```
fisher.test(matrix(c(17, 90-17, T-17, 1344-T-(90-17)), nrow=2), alternative="greater")  # T = total gold regulators; expect p=0.00012 for cluster 23
```

## Evaluation signals

- The gold-regulator-enriched cluster contains exactly 90 RBPs of which 17 are gold-standard circRNA regulators, with Fisher's exact p = 0.00012
- Overlap with Li et al.'s 103-RBP circmCherry screen is 13/90 (p = 0.00201) and with 19 curated established regulators is 6/19 (p = 0.00112)
- POSTAR3 validation yields p = 7.7e-5 against the 1,344-RBP background and p = 7.8e-30 against all genes; ENCORI >=5 stringency yields 3.3e-8 and 5.8e-25
- Each computed p-value matches the corresponding reported value within an order of magnitude
- The enrichment direction is significant over-representation of circRNA-binding RBPs among CRIT candidates in every comparison

## Limitations

- Cluster composition depends on stochastic NMF runs, so recomputed overlap p-values are only expected to match reported values within an order of magnitude
- Enrichment testing is post-hoc validation of set overlap; it does not establish that individual candidate regulators bind or regulate specific circRNAs
- The article flags that generalization of the CRIT strategy (and thus its enrichment-based validation) to other diseases such as neurodegenerative disease remains untested

## Evidence
