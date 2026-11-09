---
name: permutation-significance-testing
description: Use when you must attach an empirical significance level to an entropy-
  or potency-derived statistic — e.g. testing whether a bulk-sample signalling entropy
  (SR) significantly exceeds a matched single-cell SR distribution — without assuming
  a parametric null.
when_to_use_negative:
- The observed statistic falls outside the single-cell reference range entirely (e.g.
  AML population SR=0.88 > max single-cell SR=0.82), where an exceedance comparison
  rather than resampling within the reference is the appropriate argument
- The statistic is a bulk-average profile with no regulated heterogeneity, which the
  control shows is correctly non-significant (P≈0.30) — do not force significance
  interpretation
- Entropies are transcriptomic entropies not based on network information (e.g. as
  in StemID/SLICE); the SCENT-network-based SR framework underpinning this test would
  not apply
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3572
tools:
- name: SCENT
  role: Computes the network-based signalling entropy (SR) of single cells and bulk
    samples whose deviation (MRH z-score) is assessed for significance by permutation
  repo: aet21/SCENT
- name: R
  role: Environment in which single-cell and bulk expression matrices are loaded,
    genes mapped to the interaction network, and empirical P-values and Fisher combined
    tests are computed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:2a62e004942a698449c860b4a2a6359a959c1dcb2cb36fa55e47282eca29afc7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - R
  - SCENT
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3572
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# permutation-significance-testing

## Summary

Assess statistical significance of signalling-entropy-based quantities (e.g. the MRH z-score of a bulk sample's deviation from a matched single-cell SR distribution) by empirical/permutation P-values, complemented by combined Fisher tests and negative controls, as done in the SCENT-based regulated heterogeneity analysis.

## When to use

Use when you must attach an empirical significance level to an entropy- or potency-derived statistic — e.g. testing whether a bulk-sample signalling entropy (SR) significantly exceeds a matched single-cell SR distribution — without assuming a parametric null.

## When NOT to use

- The observed statistic falls outside the single-cell reference range entirely (e.g. AML population SR=0.88 > max single-cell SR=0.82), where an exceedance comparison rather than resampling within the reference is the appropriate argument
- The statistic is a bulk-average profile with no regulated heterogeneity, which the control shows is correctly non-significant (P≈0.30) — do not force significance interpretation
- Entropies are transcriptomic entropies not based on network information (e.g. as in StemID/SLICE); the SCENT-network-based SR framework underpinning this test would not apply

## Inputs

- Matched single-cell and bulk RNA-Seq expression matrices mapped onto the protein interaction network required by SCENT
- Single-cell SR reference distribution per cell type (computed with SCENT)
- Observed bulk-sample SR values or MRH z-scores per patient/cell type
- Negative-control quantities (bulk-average profile; population SR vs maximum single-cell SR)

## Outputs

- Per-bulk-sample empirical/permutation P-values for each cell type (melanoma cancer, T, B cells)
- Fisher combined P-value across patients (e.g. P<1e-6 for melanoma cancer cells)
- Negative-control results (bulk-average P≈0.30; AML population SR=0.88 vs max single-cell SR=0.82)
- Results table (e.g. mrh_zscore_results.csv) with z-scores, P-values and control values

## How to apply

Compute the observed statistic (here, the MRH z-score: the standardized deviation of a bulk sample's SCENT signalling entropy from the matched single-cell SR distribution) and derive an empirical/permutation P-value by resampling from the single-cell SR reference distribution for each patient's bulk sample (melanoma cancer cells, T cells, B cells). Combine per-patient P-values with a Fisher combined test (melanoma cancer cells: combined P<1e-6). Validate the test's calibration with negative controls: (a) a bulk-average profile should yield a non-significant P≈0.30, and (b) an AML population SR=0.88 exceeding the maximum single-cell SR=0.82 demonstrates when the statistic falls outside the reference range entirely. Write all z-scores, P-values and control values (e.g. mrh_zscore_results.csv) so the significance assessment is reproducible.

## Related tools

- **SCENT** (Computes the network-based signalling entropy (SR) of single cells and bulk samples whose deviation (MRH z-score) is assessed for significance by permutation) — aet21/SCENT
- **R** (Environment in which single-cell and bulk expression matrices are loaded, genes mapped to the interaction network, and empirical P-values and Fisher combined tests are computed)

## Examples

```
devtools::install_github("aet21/SCENT"); library(SCENT)  # then compute per-cell SR, derive MRH z-scores vs the single-cell SR distribution, and write mrh_zscore_results.csv
```

## Evaluation signals

- Melanoma cancer-cell bulk samples from all 12 patients yield significant MRH z-scores with Fisher combined P<1e-6
- The bulk-average negative control is non-significant (P approximately 0.30), confirming no inflation of the test statistic without regulated heterogeneity
- AML population SR=0.88 exceeds the maximum single-cell SR=0.82, as expected for a genuinely heterogeneous population outside the single-cell reference
- Bulk entropies are much higher than expected from the single-cell SR distributions in T cells and B cells, matching the reported finding
- All z-scores, P-values and control values are written to mrh_zscore_results.csv

## Limitations

- No existing assay can measure all single cells in a population, limiting validation of bulk-vs-single-cell synergy
- The precise relation between single-cell signalling entropy and transcriptional entropy (as defined by Richard et al.) is non-trivial and not established
- Criticality hallmarks (e.g. an SI peak before a cell-fate transition) remain untested, pending upcoming high-resolution time-course and genome-wide scRNA-Seq data

## Evidence

- [abstract] It further reveals that expression heterogeneity within single-cell populations is regulated.: "It further reveals that expression heterogeneity within single-cell populations is regulated."
- [abstract] signalling entropy allows in silico estimation of potency of bulk samples: "signalling entropy allows in silico estimation of the differentiation potency and plasticity of single cells and bulk samples, providing a means to identify normal and cancer stem-cell phenotypes"
- [readme] The main purpose of the SCENT package is to provide a means of estimating the differentiation potency of single cells: "The main purpose of the `SCENT` package is to provide a means of estimating the differentiation potency of single cells without the need to assume prior biological knowledge"
- [readme] devtools::install_github("aet21/SCENT"): "devtools::install_github("aet21/SCENT")"
- [discussion] no existing assay can measure all single cells in a population: "While no existing assay can measure all single cells in a population"
