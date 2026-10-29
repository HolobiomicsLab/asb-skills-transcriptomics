---
name: cms-subtype-molecular-association-analysis
description: Use when CMS subtype calls from CMScaller must be cross-validated against
  per-sample molecular features (MSI/MSS status, mutation counts, BRAF status, CNA
  burden) to confirm biologically expected subtype associations.
when_to_use_negative:
- Samples are metastatic CRC biopsies or otherwise have a different human stromal
  component than primary CRCs, where CMScaller predictions themselves are not recommended
- The expression dataset contains fewer than approximately forty samples, where centering/scaling
  input data introduces substantial prediction uncertainty before any association
  testing
- Input is single-cell RNA-sequencing data, which CMScaller is not optimized for because
  it is exceedingly noisy
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0203
tools:
- name: CMScaller
  role: Produces the CMS subtype predictions (CMS1–CMS4) for the pre-clinical models
    whose molecular associations are being tested
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Environment for running fisher.test() and wilcox.test() on the contingency
    tables and burden comparisons
- name: Biobase
  role: Loaded in R to handle the gene expression data structures (ExpressionSet)
    feeding CMScaller predictions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:d3c217c74aea91447906791b30596cf03ee6f29dc8673308709b528ee7911b4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# cms-subtype-molecular-association-analysis

## Summary

Statistically validates CMScaller consensus molecular subtype (CMS) assignments of colorectal cancer pre-clinical models (organoids, PDXs) against molecular annotations — MSI status, mutation burden, BRAF mutation, and copy-number aberration (CNA) burden — using Fisher's exact and Mann–Whitney U tests in R.

## When to use

Use when CMS subtype calls from CMScaller must be cross-validated against per-sample molecular features (MSI/MSS status, mutation counts, BRAF status, CNA burden) to confirm biologically expected subtype associations.

## When NOT to use

- Samples are metastatic CRC biopsies or otherwise have a different human stromal component than primary CRCs, where CMScaller predictions themselves are not recommended
- The expression dataset contains fewer than approximately forty samples, where centering/scaling input data introduces substantial prediction uncertainty before any association testing
- Input is single-cell RNA-sequencing data, which CMScaller is not optimized for because it is exceedingly noisy

## Inputs

- CMScaller CMS subtype assignment table for the cohort (per-sample predicted CMS class, e.g. from res$prediction)
- Per-sample MSI/MSS status annotations (e.g. Fujii et al. organoid cohort)
- Per-sample total mutation counts (Gao et al. PDX cohort)
- Per-sample BRAF mutation status (mutant vs wildtype)
- Per-sample copy number aberration (CNA) counts

## Outputs

- 2x2 contingency tables for categorical tests (CMS1 vs non-CMS1 by MSI-positive vs MSI-negative)
- Fisher's exact test p-values for CMS1–MSI and BRAF enrichment associations (e.g. p = 1.6e-4, p = 4e-4)
- Mann–Whitney U test p-values and group medians for mutation and CNA burden comparisons (e.g. CMS2 median 216 vs 112, p = 5.9e-3)
- Consolidated results table statistical_association_tests.csv listing test names, group sizes/statistics, and p-values

## How to apply

Load CMScaller CMS predictions for the cohort (e.g. Fujii et al. organoids or Gao et al. PDXs) together with per-sample molecular annotations. For categorical associations such as CMS1-MSI concordance, build a 2x2 contingency table (CMS1 vs non-CMS1 by MSI-positive vs MSI-negative) and apply two-sided Fisher's exact test via fisher.test() in R; the Fujii organoid cohort yielded 5 of 6 MSI samples predicted CMS1-immune, p = 1.6e-4. For continuous molecular features in the Gao PDX cohort, compare total mutation counts between CMS groups with a two-sided Mann–Whitney U test (wilcox.test, reported p = 9.9e-4) and CNA burden between subtypes (CMS2 median 216 vs 112, p = 5.9e-3); test BRAF-mutant vs BRAF-wildtype enrichment across subtypes with Fisher's exact test (p = 4e-4). Compile all test names, group sizes/statistics, and p-values into a single results table (statistical_association_tests.csv) so every association is auditable. Rationale: CMScaller relies on cancer cell-intrinsic markers, so molecular features like MSI and BRAF mutation should track with specific subtypes (CMS1-immune), providing an independent correctness check on the classifications.

## Related tools

- **CMScaller** (Produces the CMS subtype predictions (CMS1–CMS4) for the pre-clinical models whose molecular associations are being tested) — https://github.com/Lothelab/CMScaller
- **R** (Environment for running fisher.test() and wilcox.test() on the contingency tables and burden comparisons)
- **Biobase** (Loaded in R to handle the gene expression data structures (ExpressionSet) feeding CMScaller predictions)

## Examples

```
res <- CMScaller(emat, RNAseq=TRUE); tab <- table(CMS1=res$prediction=="CMS1", MSI=msi_status); fisher.test(tab); wilcox.test(mutation_count ~ prediction, data=pdx)
```

## Evaluation signals

- The CMS1-MSI contingency table in the Fujii organoid cohort reproduces 5 of 6 MSI samples predicted as CMS1-immune with Fisher's exact test p = 1.6e-4
- Mann–Whitney U comparison of total mutation counts between CMS groups in Gao PDXs yields p = 9.9e-4
- CNA burden comparison shows CMS2 median 216 versus 112 with p = 5.9e-3
- Fisher's exact test for BRAF-mutant versus BRAF-wildtype enrichment across subtypes gives p = 4e-4
- statistical_association_tests.csv contains one row per test with test name, group sizes/statistics, and p-value, and all p-values match the reported values

## Limitations

- CMScaller in its current implementation is not recommended for samples with a different human stromal component than primary CRCs, e.g. metastatic CRC biopsies
- Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples)
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic) may explain prediction errors at class boundaries, which could weaken subtype–molecular associations

## Evidence

- [results] CMS1–MSI concordance in organoids: [evidence span withheld — anchor status: not_found]
- [abstract] NTP-based classifier scope: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [abstract] Package availability: "CMScaller is available as an R package."
- [readme] Core algorithm: [evidence span withheld — anchor status: not_found]
- [discussion] Small-dataset caveat: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
