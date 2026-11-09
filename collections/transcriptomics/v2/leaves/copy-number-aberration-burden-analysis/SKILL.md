---
name: copy-number-aberration-burden-analysis
description: Use when CMScaller CMS subtype assignments exist for a pre-clinical CRC
  cohort (e.g. Gao PDXs, Fujii organoids) and you need to test whether mutation burden,
  CNA burden, BRAF mutation status, or MSI is enriched in specific subtypes.
when_to_use_negative:
- Input samples are primary CRC tumors or metastatic biopsies with a human stromal
  component — the original CMS classifier, not CMScaller, applies, and CMScaller is
  explicitly not recommended for metastatic CRC biopsies.
- Cohort has fewer than approximately forty samples — centered/scaled input introduces
  prediction uncertainty for such small datasets.
- Data is single-cell RNA-sequencing — CMScaller is not optimized for such exceedingly
  noisy data.
tools:
- name: CMScaller
  role: Assigns CMS1-CMS4 subtypes to pre-clinical model gene expression data, providing
    the grouping variable for burden and enrichment tests
  repo: https://github.com/Lothelab/CMScaller
- name: R
  role: Environment for running fisher.test() on the MSI-CMS1 contingency table and
    two-sided wilcox.test() (Mann-Whitney U) on mutation and CNA burdens
- name: Biobase
  role: Loaded to handle expression data containers when running CMScaller predictions
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-16747-x
    title: 'CMScaller: an R package for consensus molecular subtyping of colorectal
      cancer pre-clinical models'
  source_content_hash: sha256:1f77ba7719898f6aaf3e523ffc9eff8d75957466e7837095d792974e560718af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Lothelab/CMScaller
  tools:
  - Biobase
  - CMScaller
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# copy-number-aberration-burden-analysis

## Summary

Statistically associates CMScaller-derived CMS subtypes of pre-clinical colorectal cancer models (PDXs, organoids) with molecular features such as copy number aberration (CNA) burden, total mutation counts, BRAF status, and MSI, using two-sided Mann-Whitney U tests and Fisher's exact tests in R.

## When to use

Use when CMScaller CMS subtype assignments exist for a pre-clinical CRC cohort (e.g. Gao PDXs, Fujii organoids) and you need to test whether mutation burden, CNA burden, BRAF mutation status, or MSI is enriched in specific subtypes.

## When NOT to use

- Input samples are primary CRC tumors or metastatic biopsies with a human stromal component — the original CMS classifier, not CMScaller, applies, and CMScaller is explicitly not recommended for metastatic CRC biopsies.
- Cohort has fewer than approximately forty samples — centered/scaled input introduces prediction uncertainty for such small datasets.
- Data is single-cell RNA-sequencing — CMScaller is not optimized for such exceedingly noisy data.

## Inputs

- CMScaller CMS subtype assignment table for the cohort (e.g. Gao et al. PDXs, Fujii et al. organoids)
- Per-sample total mutation counts
- Per-sample BRAF mutation status (mutant vs wildtype)
- Per-sample copy number aberration (CNA) counts
- Per-sample microsatellite instability (MSI) status annotations (organoid cohort)

## Outputs

- statistical_association_tests.csv containing test names, group sizes/statistics, and p-values
- 2x2 contingency table of CMS1 vs non-CMS1 by MSI-positive vs MSI-negative
- Reported statistics: mutation burden p=9.9e-4, CNA burden CMS2 median 216 vs 112 (p=5.9e-3), BRAF enrichment p=4e-4, MSI-CMS1 p=1.6e-4

## How to apply

Run CMScaller (Nearest Template Prediction-based, available as an R package using Biobase ExpressionSet-style input matrices) to assign CMS1–CMS4 classes to the gene expression matrix of the cohort. Load per-sample total mutation counts, BRAF mutation status, CNA counts, and (for organoids) MSI status annotations. Build a 2x2 contingency table (CMS1 vs non-CMS1 by MSI+ vs MSI-) and apply fisher.test() in R; the Fujii organoid validation reported 5/6 MSI organoids predicted as CMS1-immune, p = 1.6e-4. For the Gao PDX cohort, apply two-sided wilcox.test (Mann-Whitney U) comparing total mutation counts between CMS groups (reported p = 9.9e-4) and CNA burden between subtypes (CMS2 median 216 vs 112, p = 5.9e-3), and fisher.test() for BRAF-mutant vs BRAF-wildtype enrichment across subtypes (p = 4e-4). Compile all test names, group sizes/statistics, and p-values into a single results table such as statistical_association_tests.csv.

## Related tools

- **CMScaller** (Assigns CMS1-CMS4 subtypes to pre-clinical model gene expression data, providing the grouping variable for burden and enrichment tests) — https://github.com/Lothelab/CMScaller
- **R** (Environment for running fisher.test() on the MSI-CMS1 contingency table and two-sided wilcox.test() (Mann-Whitney U) on mutation and CNA burdens)
- **Biobase** (Loaded to handle expression data containers when running CMScaller predictions)

## Examples

```
res <- CMScaller(emat, RNAseq=TRUE); wilcox.test(total_mutations ~ cms_class, data=gao_pdx, alternative="two.sided")
```

## Evaluation signals

- The Fisher's exact test on the organoid 2x2 table reproduces 5/6 MSI organoids predicted CMS1 with p = 1.6e-4.
- Mann-Whitney U test on Gao PDX total mutation counts between CMS groups yields p = 9.9e-4.
- CNA burden comparison yields CMS2 median 216 versus 112 with p = 5.9e-3.
- BRAF-mutant vs BRAF-wildtype Fisher's exact test across subtypes yields p = 4e-4.
- Output CSV contains one row per test with test name, group sizes/statistics, and p-value, and all reported p-values fall below 0.01.

## Limitations

- Input gene expression data must be centered and scaled, which introduces prediction uncertainty for small datasets (fewer than approximately forty samples).
- Caution is warranted for highly selected datasets where molecular distributions deviate from primary CRC cohorts, including metastatic samples.
- Intra-tumor heterogeneity (immune/stromal infiltration and cancer-cell intrinsic) may explain prediction errors at class boundaries.

## Evidence

- [discussion] CMScaller is platform independent and available as an R package.: "CMScaller is platform independent and available as an R package."
- [abstract] This new classifier recapitulated subtypes in 551 models: "This new classifier, referred to as CMScaller, recapitulated the subtypes in both in vitro and in vivo models (551 in total)."
- [readme] README description of CMScaller scope: "The `CMScaller` package provides Consensus Molecular Subtype (CMS) classification of colorectal cancer pre-clinical models"
- [abstract] Cancer-cell-intrinsic marker rationale: [evidence span withheld — anchor status: not_found]
