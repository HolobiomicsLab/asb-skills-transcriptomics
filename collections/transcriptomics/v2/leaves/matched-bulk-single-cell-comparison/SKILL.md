---
name: matched-bulk-single-cell-comparison
description: Use when matched single-cell and bulk RNA-Seq expression matrices are
  available for the same cell population and the goal is to test whether bulk signalling
  entropy exceeds what random single-cell sampling would explain, i.e. regulated heterogeneity.
when_to_use_negative:
- Only a bulk-average profile is available with no matched single cells — the bulk-average
  control itself is expected to be non-significant (P approximately 0.30), so MRH
  significance cannot be assessed
- The goal is ordering cells by differentiation potency in pseudo-time; SCENT alone
  (or alternatives like Monocle, MPath, StemID, SLICE) is the appropriate framing
  rather than a bulk-vs-single-cell MRH test
- Single-cell and bulk data are not matched to the same cell population, since the
  z-score is defined against the matched population's single-cell SR distribution
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCENT
  role: Computes network-based signalling entropy (SR) for each single cell and each
    bulk sample; its package underpins the MRH z-score test
  repo: https://github.com/aet21/SCENT
- name: R
  role: Environment in which the SCENT package is installed via devtools and the matched
    bulk-vs-single-cell MRH analysis is run
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms15599
    title: Single-cell entropy for accurate estimation of differentiation potency
      from a cell’s transcriptome
  source_content_hash: sha256:87b55a71b7478f1c3805295a344c86157a4ef002407f31f7e3ca89bb29d7dcd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/SCENT
  tools:
  - R
  - SCENT
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# matched-bulk-single-cell-comparison

## Summary

Quantifies regulated single-cell expression heterogeneity (the measure of regulated heterogeneity, MRH) by testing whether the SCENT signalling entropy (SR) of a bulk RNA-Seq sample deviates significantly above the single-cell SR distribution computed from matched individual cells.

## When to use

Use when matched single-cell and bulk RNA-Seq expression matrices are available for the same cell population and the goal is to test whether bulk signalling entropy exceeds what random single-cell sampling would explain, i.e. regulated heterogeneity.

## When NOT to use

- Only a bulk-average profile is available with no matched single cells — the bulk-average control itself is expected to be non-significant (P approximately 0.30), so MRH significance cannot be assessed
- The goal is ordering cells by differentiation potency in pseudo-time; SCENT alone (or alternatives like Monocle, MPath, StemID, SLICE) is the appropriate framing rather than a bulk-vs-single-cell MRH test
- Single-cell and bulk data are not matched to the same cell population, since the z-score is defined against the matched population's single-cell SR distribution

## Inputs

- Matched single-cell RNA-Seq expression matrix for the population of interest
- Matched bulk RNA-Seq expression matrix for the same population
- Protein interaction network used by SCENT for entropy estimation

## Outputs

- Per-cell signalling entropy (SR) reference distribution
- Bulk-sample SR values and MRH z-scores with empirical/permutation P-values
- Negative-control values (bulk-average P-value; population SR vs maximum single-cell SR)
- Results table, e.g. mrh_zscore_results.csv

## How to apply

Load the matched single-cell and bulk expression matrices into R and map genes onto the protein interaction network required by SCENT. Compute the network-based signalling entropy (SR) of every individual cell with SCENT to build the reference single-cell SR distribution, then compute the SR of each matched bulk sample and derive its MRH z-score as the standardized deviation of the bulk SR from that single-cell distribution. Assign an empirical/permutation P-value to each bulk-sample z-score (e.g. melanoma T-cell, B-cell and cancer-cell bulk samples, combined via Fisher's test, gave P<1e-6). Always run the negative controls: a bulk-average profile should be non-significant (P approximately 0.30), and a truly heterogeneous population such as AML yields SR=0.88 exceeding the maximum single-cell SR=0.82. Write all z-scores, P-values and control values (e.g. to mrh_zscore_results.csv) and confirm melanoma/cancer samples are significant while the bulk-average control is not.

## Related tools

- **SCENT** (Computes network-based signalling entropy (SR) for each single cell and each bulk sample; its package underpins the MRH z-score test) — https://github.com/aet21/SCENT
- **R** (Environment in which the SCENT package is installed via devtools and the matched bulk-vs-single-cell MRH analysis is run)

## Examples

```
devtools::install_github("aet21/SCENT"); library(SCENT)  # then compute per-cell SR and bulk SR, z-scores, and permutation P-values
```

## Evaluation signals

- Melanoma cancer cells from each of the 12 patients yield significant MRH z-scores with combined Fisher test P<1e-6
- The bulk-average negative control is non-significant with P approximately 0.30
- A genuinely heterogeneous population (AML) has population SR=0.88 exceeding the maximum single-cell SR=0.82
- All z-scores, P-values and control values are present in the output file (e.g. mrh_zscore_results.csv)
- Single-cell SR values are computable after genes are mapped onto the SCENT protein interaction network (no cells dropped to failure)

## Limitations

- No existing assay can measure all single cells in a population, which limits direct validation of the bulk-vs-single-cell synergy
- The precise relation between single-cell signalling entropy and transcriptional entropy as defined by Richard et al. is non-trivial and not established
- Criticality hallmarks such as an SI peak before a cell-fate transition remain untested pending high-resolution time-course and genome-wide scRNA-Seq data

## Evidence

- [abstract] Expression heterogeneity within single-cell populations is regulated: "It further reveals that expression heterogeneity within single-cell populations is regulated."
- [abstract] signalling entropy of bulk and single cells: "signalling entropy allows in silico estimation of the differentiation potency and plasticity of single cells and bulk samples"
- [readme] SCENT estimates potency from a cell's transcriptome: "estimating the differentiation potency of single cells using SCENT and CCAT"
- [discussion] no existing assay can measure all single cells: "While no existing assay can measure all single cells in a population"
