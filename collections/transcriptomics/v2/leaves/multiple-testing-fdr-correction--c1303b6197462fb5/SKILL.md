---
name: multiple-testing-fdr-correction
description: Use when a genome- or transcriptome-wide set of per-gene hypothesis tests
  (e.g., allelic burst-correlation logOR tests from SCALE) must be corrected for multiplicity
  before declaring significant genes at a fixed FDR level.
when_to_use_negative:
- The tests are not genome-wide/per-gene multiple comparisons (e.g., a single pre-specified
  hypothesis test), where BH correction would be unnecessarily conservative
- Input is raw allelic read counts rather than model-derived per-gene test statistics
  — run scBASE count estimation and SCALE fitting first
- The analysis goal is differential burst kinetics between two conditions rather than
  within-gene allelic independence testing
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SCALE
  role: Fits the allelic bursting model to scBASE-estimated counts and produces per-gene
    burst-correlation logOR statistics and independence-test p-values that are then
    FDR-corrected
- name: scBASE
  role: Supplies the four allelic count matrices (unique-read, weighted-allocation,
    with/without partial pooling) whose results are compared after FDR correction
  repo: github.com/churchill-lab/scBASE
- name: Stan
  role: Probabilistic programming language in which the Bayesian bursting/independence
    models underlying the per-gene tests are implemented
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:00bb47ebcd50de24cf0f782cc75b6262d836ce63b23b2a2f7a6e017dd93cb453
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCALE
  - Stan
  - scBASE
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# multiple-testing-fdr-correction

## Summary

Applies Benjamini-Hochberg false discovery rate correction to per-gene p-values from allelic-bursting independence tests (SCALE logOR tests on 8,290 genes across 122 blastocyst cells), then tabulates significant genes (logOR > 0) and large-effect genes (|logOR| > 2) at a 5% FDR threshold per scBASE count-estimation method.

## When to use

Use when a genome- or transcriptome-wide set of per-gene hypothesis tests (e.g., allelic burst-correlation logOR tests from SCALE) must be corrected for multiplicity before declaring significant genes at a fixed FDR level.

## When NOT to use

- The tests are not genome-wide/per-gene multiple comparisons (e.g., a single pre-specified hypothesis test), where BH correction would be unnecessarily conservative
- Input is raw allelic read counts rather than model-derived per-gene test statistics — run scBASE count estimation and SCALE fitting first
- The analysis goal is differential burst kinetics between two conditions rather than within-gene allelic independence testing

## Inputs

- Per-gene p-values from SCALE allelic-bursting independence tests (one set per scBASE count matrix: unique-read, weighted-allocation, unique+partial-pooling, weighted+partial-pooling)
- Per-gene burst-correlation log odds ratios (logOR) for the same 8,290 genes
- Alpha threshold: 5% FDR (Benjamini-Hochberg) and |logOR| > 2 effect-size cutoff

## Outputs

- scale_independence_results.csv: per-scBASE-method counts of significant genes (logOR > 0) at 5% FDR and genes with |logOR| > 2, before and after partial pooling
- FDR-adjusted p-value vector and significance calls per gene
- Summary comparison table (Fig 5 / Supplementary Fig 2 reproduction)

## How to apply

Fit the SCALE bursting model to each of the four scBASE allelic count matrices (unique-read, weighted-allocation, and each with partial pooling) for the 8,290 analyzed genes, obtaining a per-gene burst-correlation log odds ratio (logOR) and a p-value for deviation from independence. Apply Benjamini-Hochberg correction across all 8,290 genes simultaneously and call genes significant at a 5% FDR threshold; separately count genes with |logOR| > 2 as large-effect cases. Compare counts across the four scBASE methods to quantify how partial pooling changes the inference — pooling increases significant-gene counts from 3,381 (unique reads) and 4,815 (weighted allocation) to 6,068 and 6,761 respectively, because pooling reduces sampling variability without sacrificing cell-to-cell heterogeneity. Tabulate all counts per method into scale_independence_results.csv and check against the published validation values.

## Related tools

- **SCALE** (Fits the allelic bursting model to scBASE-estimated counts and produces per-gene burst-correlation logOR statistics and independence-test p-values that are then FDR-corrected)
- **scBASE** (Supplies the four allelic count matrices (unique-read, weighted-allocation, with/without partial pooling) whose results are compared after FDR correction) — github.com/churchill-lab/scBASE
- **Stan** (Probabilistic programming language in which the Bayesian bursting/independence models underlying the per-gene tests are implemented)

## Examples

```
from statsmodels.stats.multitest import multipletests; reject, p_adj, _, _ = multipletests(pvals, alpha=0.05, method='fdr_bh')
```

## Evaluation signals

- Significant-gene counts at 5% FDR must reproduce the published values exactly: 3,381 (unique reads), 4,815 (weighted allocation), 6,068 (unique+partial pooling), 6,761 (weighted+partial pooling) out of 8,290 genes
- |logOR| > 2 counts must equal 2,845/3,763 before pooling and 5,622/6,209 after pooling out of 8,290 genes
- The corrected calls must show logOR > 0 for the majority of significant genes, consistent with positive correlation rather than either perfect independence or perfect dependence
- Partial-pooling methods must yield equal or greater significant-gene counts than their non-pooled counterparts, reflecting reduced sampling variability
- FDR-adjusted p-values must be monotonically non-decreasing relative to raw p-values and bounded at 1.0

## Limitations

- The majority of genes show statistical dependence, so declaring 'independence' is rejected for most genes under all four scBASE methods — the correction does not rescue results from systematic allelic dependence
- Partial pooling has the biggest impact when read coverage is low and the number of cells is large; FDR-corrected counts will differ substantially with coverage regimes
- Discarding multi-mapping reads (unique-reads method) increases sampling zeros, which changes both the underlying tests and the FDR-corrected significance counts relative to weighted allocation

## Evidence

- [discussion] Allelic bursting is statistically dependent and positively correlated for the majority of genes: [evidence span withheld — anchor status: not_found]
- [abstract] Partial pooling reduces sampling variability, driving the FDR-corrected count differences across methods: "combining information across cells using a hierarchical mixture model reduces sampling variability without sacrificing cell-to-cell heterogeneity"
- [intro] The independence testing context in which the FDR correction is applied: [evidence span withheld — anchor status: not_found]
- [readme] scBASE's soft zero-and-one inflated model motivates the count matrices feeding the tests: "We propose a new method, **scBASE**, a “soft” zero-and-one inflated model for cellular allelic proportions in which we tackle data sparsity and variability by harnessing information derived from"
