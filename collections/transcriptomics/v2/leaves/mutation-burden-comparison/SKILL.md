---
name: mutation-burden-comparison
description: Use when you have per-sample CMS subtype assignments plus molecular features
  (total mutation counts, CNA counts, BRAF status) and need to test whether alteration
  burden differs significantly between subtypes.
when_to_use_negative:
- Samples have not been assigned CMS subtypes by CMScaller — subtype-comparison tests
  require per-sample predictions as the grouping variable
- Data are single-cell RNA-seq or primary tumor biopsies with human stromal content,
  where CMScaller predictions are not recommended
- The dataset is small (fewer than ~40 samples) where CMScaller's centered-and-scaled
  input introduces prediction uncertainty, weakening subtype grouping for burden tests
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: CMScaller
  role: Provides the consensus molecular subtype predictions used to group samples
    before burden comparison; core algorithm is Nearest Template Prediction
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Statistical environment running fisher.test() and wilcox.test() to compare
    mutation, CNA, and MSI associations across subtypes
- name: Biobase
  role: Bioconductor base package loaded in the CMScaller workflow for handling expression
    data containers
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:7d74a612d508ecb6748b4bdad683f7c5214ba25fe06826417f54de846af4cfeb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# mutation-burden-comparison

## Summary

Compares genomic alteration burden (total mutation counts, copy number aberration counts, BRAF mutation status) across CMScaller consensus molecular subtype groups in pre-clinical colorectal cancer models, using Mann-Whitney U and Fisher's exact tests in R.

## When to use

Use when you have per-sample CMS subtype assignments plus molecular features (total mutation counts, CNA counts, BRAF status) and need to test whether alteration burden differs significantly between subtypes.

## When NOT to use

- Samples have not been assigned CMS subtypes by CMScaller — subtype-comparison tests require per-sample predictions as the grouping variable
- Data are single-cell RNA-seq or primary tumor biopsies with human stromal content, where CMScaller predictions are not recommended
- The dataset is small (fewer than ~40 samples) where CMScaller's centered-and-scaled input introduces prediction uncertainty, weakening subtype grouping for burden tests

## Inputs

- CMScaller CMS subtype assignments per sample (prediction table for a pre-clinical model cohort, e.g. Fujii et al. organoids or Gao et al. PDXs)
- Per-sample total mutation counts (from whole-exome or targeted sequencing)
- Per-sample copy number aberration (CNA) counts
- Per-sample BRAF mutation status (mutant/wildtype)
- MSI status annotations (MSI-positive/MSI-negative) for organoid samples

## Outputs

- statistical_association_tests.csv — table of test names, group sizes/statistics, and p-values
- Contingency tables (e.g. CMS1 vs non-CMS1 by MSI status)
- Median burden values per subtype (e.g. CMS2 CNA median 216 versus 112)

## How to apply

Load the CMScaller subtype assignments for each cohort (e.g., the Gao et al. PDX cohort) together with per-sample total mutation counts, BRAF mutation status, and copy number aberration (CNA) counts. Apply the two-sided Mann-Whitney U test (wilcox.test in R) to compare total mutation counts between CMS groups (reported p = 9.9e-4) and to compare CNA burden between subtypes (CMS2 median 216 versus 112, p = 5.9e-3). Apply Fisher's exact test (fisher.test in R) to test enrichment of BRAF-mutant versus BRAF-wildtype samples across subtypes (reported p = 4e-4). For MSI associations, build a 2x2 contingency table of CMS1 versus non-CMS1 by MSI-positive versus MSI-negative (in the Fujii et al. organoid cohort, 5/6 MSI organoids were predicted CMS1-immune, p = 1.6e-4). Compile all test names, group sizes/statistics, and p-values into a single results table such as statistical_association_tests.csv so each reported association is traceable to a concrete statistical test.

## Related tools

- **CMScaller** (Provides the consensus molecular subtype predictions used to group samples before burden comparison; core algorithm is Nearest Template Prediction) — https://github.com/Lothelab/CMScaller
- **R** (Statistical environment running fisher.test() and wilcox.test() to compare mutation, CNA, and MSI associations across subtypes)
- **Biobase** (Bioconductor base package loaded in the CMScaller workflow for handling expression data containers)

## Examples

```
library(CMScaller); res <- CMScaller(emat, RNAseq=TRUE); fisher.test(table(CMS1=res$prediction=='CMS1', MSI=msi_status)); wilcox.test(mut_counts ~ (res$prediction=='CMS2'))
```

## Evaluation signals

- Fisher's exact test on Fujii organoid CMS1-by-MSI contingency table reproduces 5/6 MSI samples as CMS1 with p ≈ 1.6e-4
- Mann-Whitney U comparison of total mutation counts across CMS groups in Gao PDXs yields p ≈ 9.9e-4
- CNA burden comparison shows CMS2 median ≈ 216 versus 112 with p ≈ 5.9e-3
- Fisher's exact test of BRAF-mutant vs BRAF-wildtype enrichment across subtypes gives p ≈ 4e-4
- statistical_associations_tests.csv contains one row per test with test name, group sizes/statistics, and p-value, all traceable to the loaded cohort data

## Limitations

- CMScaller in its current implementation is not recommended for samples with a different human stromal component than primary CRCs, including metastatic CRC biopsies
- Input gene expression data must be centered and scaled, introducing prediction uncertainty for small datasets (fewer than approximately forty samples)
- Caution warranted in highly selected datasets where molecular distributions deviate from primary CRC cohorts
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic) may explain prediction errors at class boundaries

## Evidence

- [results] CMS prediction via NTP plus mutation/CNA/BRAF burden tests: [evidence span withheld — anchor status: not_found]
- [methods] CNA burden comparison across subtypes: [evidence span withheld — anchor status: not_found]
- [abstract] CMScaller classification requirement: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [discussion] Small-dataset prediction uncertainty: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [readme] Installation from Bioconductor-compatible R package: "devtools::install_github("Lothelab/CMScaller")"
