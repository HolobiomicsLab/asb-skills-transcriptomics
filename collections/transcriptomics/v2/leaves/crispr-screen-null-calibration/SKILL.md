---
name: crispr-screen-null-calibration
description: Use when you need to verify that a differential-expression pipeline (e.g.,
  Normalisr on a Perturb-seq/CRISPR screen dataset) produces uniformly distributed
  null P values across gene expression levels before trusting its significance calls.
when_to_use_negative:
- The goal is inference of actual guide (gRNA) trans-effects or CRISPR screen hits
  — this skill uses only gRNA-free cells and provides no guide-effect sensitivity
  estimates
- The dataset is full-length (non-UMI) scRNA-seq — the article only focused on UMI-based
  datasets and nonlinear covariates may need revalidation on full-length platforms
- You need hypothesis testing of arbitrary nonlinear associations — Normalisr is explicitly
  not designed for that
tools:
- name: Normalisr
  role: Provides QC (qc_reads, qc_outlier), nonlinear mean/variance normalization
    (lcpm, normcov, compute_var, normvar), and the linear-model null DE function whose
    P values are being calibrated; installed via pip install normalisr
  repo: github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:d6baaa4a747c10750ef49eb2d71ddd5be63bf79a41692b0974b7ec6e76ef7800
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Normalisr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# crispr-screen-null-calibration

## Summary

A null-calibration procedure that validates unbiased P-value estimation of Normalisr differential expression by repeatedly running null DE between random two-group partitions of gRNA-free cells and testing uniformity of the resulting P values across ten expression bins with Kolmogorov–Smirnov tests.

## When to use

Use when you need to verify that a differential-expression pipeline (e.g., Normalisr on a Perturb-seq/CRISPR screen dataset) produces uniformly distributed null P values across gene expression levels before trusting its significance calls.

## When NOT to use

- The goal is inference of actual guide (gRNA) trans-effects or CRISPR screen hits — this skill uses only gRNA-free cells and provides no guide-effect sensitivity estimates
- The dataset is full-length (non-UMI) scRNA-seq — the article only focused on UMI-based datasets and nonlinear covariates may need revalidation on full-length platforms
- You need hypothesis testing of arbitrary nonlinear associations — Normalisr is explicitly not designed for that

## Inputs

- UMI read count matrix (tsv or Cell Ranger sparse mtx) for the Perturb-seq K562 study
- gRNA assignment matrix for identifying guide-unassigned (gRNA-free) cells
- Cell and gene name text files (one per line, tsv row/column conventions)

## Outputs

- Per-split matrices of null DE P values for every gene
- KS statistics and P values against uniformity, stratified by 10 expression bins and 100 splits
- Summary table ks_uniformity_results.csv

## How to apply

Download the UMI read count and gRNA assignment matrices of the Perturb-seq K562 (UPR) study from GEO and apply Normalisr QC (qc_reads requiring at least 500 reads and 100 non-zero genes per cell; qc_outlier with prior outlier-proportion bound 2% and Bonferroni P ≤ 1e-10; genes must have non-zero reads in at least 50 cells and at least 2% of cells). Normalize raw counts with the Normalisr chain lcpm → normcov → compute_var → normvar to remove nonlinear library-size confounders at mean and variance levels, then select the gRNA-free (guide-unassigned) cells — 4622 in the reference run. Partition them into two random groups over 100 splits, running the Normalisr linear-model differential expression function (normalisr de / norm.de) per split to obtain null P values for every gene. Stratify genes into 10 bins ordered by expression level and, for each bin in each split, apply a two-sided Kolmogorov–Smirnov test of the P values against the uniform distribution. Aggregate KS statistics and P values across bins and splits into ks_uniformity_results.csv; uniformity at all expression levels indicates unbiased P-value estimation.

## Related tools

- **Normalisr** (Provides QC (qc_reads, qc_outlier), nonlinear mean/variance normalization (lcpm, normcov, compute_var, normvar), and the linear-model null DE function whose P values are being calibrated; installed via pip install normalisr) — github.com/lingfeiwang/normalisr

## Examples

```
normalisr de -h
```

## Evaluation signals

- KS statistics per expression bin and split are small (P values consistent with uniformity), with no monotonic trend across the 10 expression-ordered bins
- The run uses exactly 100 random two-group splits of the 4622 gRNA-free K562 cells, with QC applied: ≥500 reads and ≥100 non-zero genes per cell, outlier removal bound 2% at Bonferroni P ≤ 1e-10
- Normalization chain order is correct (lcpm → normcov → compute_var → normvar) before running differential expression
- ks_uniformity_results.csv exists and contains per-bin, per-split rows of KS statistics and P values
- Group sizes per split stay above the >100 cells needed for robustness against read-count distribution distortions

## Limitations

- Only UMI-based datasets were analyzed; full-length platforms may produce a different read-count distribution and require revalidating the nonlinear cellular summary covariates
- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations
- Robustness against read count distribution distortions requires enough (> 100) cells per comparison group
- Batch effects in multi-dataset integrative studies are not addressed by Normalisr and may need additional consideration

## Evidence

- [discussion] With the prior bound for outlier proportion as 2%, low-variance outlier cells with Bonferroni P ≤ 10−10 were removed: "With the prior bound for outlier proportion as 2%, low-variance outlier cells with Bonferroni P ≤ 10−10 were removed"
- [discussion] Minimum cell QC thresholds: "we restricted our analyses to cells with at least 500 reads and 100 genes with non-zero reads"
- [abstract] Unbiased P-value estimation claim: "Normalisr achieves high sensitivity, specificity, speed, and generalizability"
- [readme] Input/output matrix format conventions: [evidence span withheld — anchor status: external_doc]
- [readme] Cell-count robustness requirement for linear-model DE: "robustness against read count distribution distortions with enough (> 100) cells"
