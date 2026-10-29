---
name: mann-whitney-u-group-comparison
description: Use when comparing a continuous, non-normally distributed variable (e.g.,
  mutation counts, CNA burden) between two independent sample groups, such as CMS1
  vs CMS2 colorectal cancer PDX subtypes.
when_to_use_negative:
- Input is already a per-group summary (only means/medians, no raw per-sample values),
  since wilcox.test requires individual observations
- Samples lack CMScaller subtype assignments and are primary CRCs with intact stroma,
  where immune/stromal-dependent classifiers and other grouping strategies apply
- Data are single-cell RNA-seq or metastatic biopsy samples, where CMScaller-based
  grouping is explicitly not recommended
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_0622
tools:
- name: R
  role: Statistical environment in which wilcox.test() and fisher.test() are run on
    CMS-assigned PDX and organoid sample groups
- name: CMScaller
  role: Produces the CMS subtype assignments (CMS1-immune, CMS2, etc.) that define
    the groups being compared for mutation and CNA burden
  repo: https://github.com/Lothelab/CMScaller
- name: Biobase
  role: Loaded in the R workflow to handle the gene expression data objects used for
    CMS prediction prior to group testing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:9d6f37449aaf49372e8581e9cdf8b3365c21542177092c975c1f06c84e2664ff
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# Mann-Whitney U group comparison

## Summary

Two-sided non-parametric comparison of a continuous variable (e.g., total mutation counts or copy-number aberration burden) between two independent groups, used in the CMScaller paper to test whether Gao et al. PDX CMS subtypes differ in mutational load and CNA burden.

## When to use

Use when comparing a continuous, non-normally distributed variable (e.g., mutation counts, CNA burden) between two independent sample groups, such as CMS1 vs CMS2 colorectal cancer PDX subtypes.

## When NOT to use

- Input is already a per-group summary (only means/medians, no raw per-sample values), since wilcox.test requires individual observations
- Samples lack CMScaller subtype assignments and are primary CRCs with intact stroma, where immune/stromal-dependent classifiers and other grouping strategies apply
- Data are single-cell RNA-seq or metastatic biopsy samples, where CMScaller-based grouping is explicitly not recommended

## Inputs

- Per-sample total mutation counts for the Gao et al. PDX cohort
- Per-sample copy number aberration (CNA) counts for the Gao et al. PDX cohort
- CMScaller CMS subtype assignments per sample (res$prediction from the CMScaller R call)
- BRAF mutation status annotations (mutant vs wildtype) per sample
- MSI status annotations for the Fujii et al. organoid cohort

## Outputs

- Two-sided Mann-Whitney U test p-values for mutation counts between CMS groups (reported p = 9.9e-4)
- Mann-Whitney U test p-value and medians for CNA burden by subtype (CMS2 median 216 vs 112, p = 5.9e-3)
- Fisher's exact test p-values for BRAF enrichment (p = 4e-4) and CMS1-MSI association (p = 1.6e-4)
- A compiled results table of test names, group sizes/statistics, and p-values saved as statistical_association_tests.csv

## How to apply

Assign each sample to a group using CMScaller subtype predictions (generated in R with the CMScaller package, which runs Nearest Template Prediction on centered and scaled gene expression data). In the Gao et al. PDX cohort, run a two-sided Mann-Whitney U test via wilcox.test() in R comparing total mutation counts between CMS groups (reported p = 9.9e-4) and CNA burden between subtypes (CMS2 median 216 vs 112, p = 5.9e-3). For categorical pairings in the same workflow, pair the Mann-Whitney test with Fisher's exact test: BRAF-mutant vs BRAF-wildtype enrichment across subtypes (p = 4e-4) and CMS1-vs-non-CMS1 by MSI status in Fujii organoids, where 5/6 MSI organoids were CMS1-immune (p = 1.6e-4). Compile test names, group sizes/statistics, and p-values into a single results table (statistical_association_tests.csv). The rationale is that mutation/CNA burden distributions are skewed and small-n, so a rank-based non-parametric test avoids normality assumptions.

## Related tools

- **R** (Statistical environment in which wilcox.test() and fisher.test() are run on CMS-assigned PDX and organoid sample groups)
- **CMScaller** (Produces the CMS subtype assignments (CMS1-immune, CMS2, etc.) that define the groups being compared for mutation and CNA burden) — https://github.com/Lothelab/CMScaller
- **Biobase** (Loaded in the R workflow to handle the gene expression data objects used for CMS prediction prior to group testing)

## Examples

```
wilcox.test(total_mutations ~ CMS_class, data = gao_pdx, alternative = "two.sided")
```

## Evaluation signals

- wilcox.test is called with alternative = 'two.sided' and no paired argument, matching the reported two-sided design
- Reported p-values fall within expected ranges: mutation-count comparison p = 9.9e-4 and CNA-burden comparison p = 5.9e-3 reproduce the paper's results
- Group sizes and medians are recorded alongside p-values (e.g., CMS2 CNA median 216 vs 112) in the output table
- statistical_association_tests.csv contains one row per test with test name, group sizes/statistics, and p-value columns
- Categorical tests in the same workflow use fisher.test() on a 2x2 contingency table (e.g., 5/6 MSI organoids predicted CMS1, p = 1.6e-4)

## Limitations

- The grouping depends on CMScaller predictions, which require input gene expression data to be centered and scaled, introducing prediction uncertainty for small datasets (fewer than approximately forty samples)
- CMScaller is not recommended for samples with a different human stromal component than primary CRCs, e.g., metastatic CRC biopsies, so group labels for such samples are unreliable
- Caution is warranted in highly selected datasets where molecular distributions deviate from primary CRC cohorts, which can bias subtype group membership and hence test results

## Evidence

- [abstract] CMScaller recapitulated subtypes in models: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [discussion] CMScaller availability in R: "CMScaller is platform independent and available as an R package."
- [discussion] Input scaling caveat: "input gene expression data should be centered and scaled. The implication is that small datasets inescapably introduce prediction uncertainty"
- [readme] Core algorithm from README: "The core algorithm is *[Nearest Template Prediction (NTP)](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0015543)* algorithm proposed by Yujin Hoshida"
