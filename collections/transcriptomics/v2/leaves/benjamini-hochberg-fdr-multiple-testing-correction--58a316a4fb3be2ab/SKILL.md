---
name: benjamini-hochberg-fdr-multiple-testing-correction
description: Use when many parallel hypothesis tests must be thresholded with false-discovery-rate
  control — e.g., hundreds of GO over-representation tests on top-ranked communities
  per donor per time point — and an adjusted-p (BH FDR 0.05) significance call is
  required.
when_to_use_negative:
- The analysis has a single pre-specified hypothesis test rather than a large family
  of parallel tests (e.g., all GO BP terms across 316 gene sets), where no multiplicity
  correction is needed
- Family-wise error rate (e.g., Bonferroni) control is explicitly required instead
  of false-discovery-rate control
- The goal is ranking terms by unadjusted effect size or enrichment fold change rather
  than making FDR-controlled significance calls
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: clusterProfiler
  role: Runs the enrichGO over-representation analysis whose GO BP term p-values are
    subsequently Benjamini-Hochberg adjusted at FDR 0.05
- name: MuDCoD
  role: Produces the 316 top-ranked community gene sets (MuDCoD.fit_predict with max_K=50,
    n_iter=30, alpha=0.05, beta=0.05) on which the BH-corrected enrichment tests are
    run
  repo: https://github.com/bo1929/MuDCoD
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad592
    title: 'MuDCoD: multi-subject community detection in personalized dynamic gene
      networks from single-cell RNA sequencing'
  source_content_hash: sha256:093b0414f0cccd55bfbeaec48d2d5fbb6ee6c528dbdb604637548c97edd8da19
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bo1929/MuDCoD
  tools:
  - Dozer
  - MuDCoD
  - clusterProfiler
  - numpy
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# benjamini-hochberg-fdr-multiple-testing-correction

## Summary

Applies the Benjamini-Hochberg step-up procedure to control the false discovery rate when calling significant results across many parallel hypothesis tests, here the enrichGO over-representation tests run on 316 CRank-ranked MuDCoD communities from the Soskic-2022 CD4+ T cell activation dataset at adjusted FDR 0.05.

## When to use

Use when many parallel hypothesis tests must be thresholded with false-discovery-rate control — e.g., hundreds of GO over-representation tests on top-ranked communities per donor per time point — and an adjusted-p (BH FDR 0.05) significance call is required.

## When NOT to use

- The analysis has a single pre-specified hypothesis test rather than a large family of parallel tests (e.g., all GO BP terms across 316 gene sets), where no multiplicity correction is needed
- Family-wise error rate (e.g., Bonferroni) control is explicitly required instead of false-discovery-rate control
- The goal is ranking terms by unadjusted effect size or enrichment fold change rather than making FDR-controlled significance calls

## Inputs

- Raw p-values (or the full enrichGO result object) from clusterProfiler enrichGO biological-process over-representation analysis for each of the 316 gene sets
- Per-gene-set mapping of each gene set to its donor and time point (0-, 16-, 24-, 40-h post-stimulation) from the MuDCoD community memberships
- Significance threshold: Benjamini-Hochberg adjusted FDR 0.05

## Outputs

- BH-adjusted q-values and binary significant/not-significant calls for each GO BP term per gene set
- Per-time-point frequency counts of significantly enriched GO terms across donors
- Top-15 most frequent enriched GO terms per time point written to go_enrichment_top15_table.csv

## How to apply

For each of the 316 gene sets (the top CRank-ranked community for every donor at every time point of the Dozer-constructed Soskic-2022 CD4+ T cell activation networks), run clusterProfiler enrichGO over-representation analysis on the biological-process ontology and extract raw term p-values. Apply the Benjamini-Hochberg adjustment to each gene set's term p-values and call a GO term significant when its BH-adjusted FDR is below 0.05. For each time point, count how frequently each significantly enriched GO term appears across donors, rank terms by frequency, and keep the top 15 per time point. The rationale is that testing all GO BP terms across 316 gene sets inflates false positives, so unadjusted p-values cannot be used for the Fig. 6b-style term-frequency table.

## Related tools

- **clusterProfiler** (Runs the enrichGO over-representation analysis whose GO BP term p-values are subsequently Benjamini-Hochberg adjusted at FDR 0.05)
- **MuDCoD** (Produces the 316 top-ranked community gene sets (MuDCoD.fit_predict with max_K=50, n_iter=30, alpha=0.05, beta=0.05) on which the BH-corrected enrichment tests are run) — https://github.com/bo1929/MuDCoD

## Examples

```
enrichGO(gene = top_community_genes, OrgDb = org.Hs.eg.db, keyType = "ENTREZID", ont = "BP", pAdjustMethod = "BH", pAdjustMethod cutoff: pvalueCutoff = 0.05, qvalueCutoff = 0.05)
```

## Evaluation signals

- The output table lists exactly 15 terms per time point (0-, 16-, 24-, 40-h)
- 'positive regulation of T cell activation' appears among the top-15 most frequent enriched GO terms only at the 16-h and 40-h time points, matching Fig. 6b
- For every GO term, the BH-adjusted p-value is greater than or equal to the raw p-value, and no term with raw p > 0.05 passes the adjusted FDR 0.05 cutoff
- Significance calls are computed per gene set from 316 sets (79 donors × 4 time points), so the total number of enrichGO tests subjected to correction equals 316
- 'positive regulation of cytokine production' increases in frequency at 40-h relative to earlier time points, consistent with CD4+ T cell activation mechanisms

## Limitations

- BH controls the expected false discovery rate, not the family-wise error rate, so individual false-positive terms can still appear among the top-15 frequency tables
- The frequency ranking depends on upstream decisions (Dozer network construction, CRank ranking of the single top community per donor per time point); different community-selection choices change which terms pass FDR 0.05
- The context provides no numeric ARI or enrichment output values, so downstream agreement can only be validated structurally (term presence at 16-h/40-h), not quantitatively against reported statistics

## Evidence

- [other] Run clusterProfiler enrichGO over-representation analysis on the biological-process ontology for each of the 316 gene sets, calling terms significant at Benjamini-Hochberg adjusted FDR 0.05.: [evidence span withheld — anchor status: not_found]
- [abstract] Our results illustrate how personalized dynamic community detection can aid in the exploration of subject-specific biological processes that vary across time: "Our results illustrate how personalized dynamic community detection can aid in the exploration of subject-specific biological processes that vary across time"
- [readme] MuDCoD (Multi-subject Dynamic Community Detection) provides robust community detection in time-varying personalized networks modules.: "MuDCoD (Multi-subject Dynamic Community Detection) provides robust community detection in time-varying personalized networks modules."
