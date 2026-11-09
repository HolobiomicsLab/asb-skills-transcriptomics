---
name: kolmogorov-smirnov-null-uniformity-testing
description: Use when validating that a DE method (e.g. Normalisr linear models) produces
  uniformly distributed null P values across gene expression levels, using replicate
  null groupings such as gRNA-free cells split randomly.
when_to_use_negative:
- Hypothesis testing of arbitrary nonlinear associations — Normalisr explicitly states
  it is not designed for this
- Datasets with fewer than ~100 cells per group, where Normalisr's robustness against
  read count distribution distortions no longer holds
- Full-length (non-UMI) scRNA-seq platforms, which the paper did not validate for
  its nonlinear cellular summary covariates
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Normalisr
  role: Generates the null DE P values (normalisr.de) on gRNA-free cell splits after
    lcpm/normcov/compute_var/normvar normalization, which are then tested for uniformity
    with KS
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:d29305296ce4a4ac01f5da71fce22d12d9a7911a1104dbff9fb86e63aff1c4f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# kolmogorov-smirnov-null-uniformity-testing

## Summary

Two-sided Kolmogorov–Smirnov test of null differential-expression P values against the uniform distribution, stratified by expression-level bins, to verify that Normalisr yields unbiased P-value estimation in single-cell DE.

## When to use

Use when validating that a DE method (e.g. Normalisr linear models) produces uniformly distributed null P values across gene expression levels, using replicate null groupings such as gRNA-free cells split randomly.

## When NOT to use

- Hypothesis testing of arbitrary nonlinear associations — Normalisr explicitly states it is not designed for this
- Datasets with fewer than ~100 cells per group, where Normalisr's robustness against read count distribution distortions no longer holds
- Full-length (non-UMI) scRNA-seq platforms, which the paper did not validate for its nonlinear cellular summary covariates

## Inputs

- UMI read count matrix (tsv or sparse mtx) from the UPR Perturb-seq K562 study (downloaded from GEO)
- gRNA assignment matrix used to select gRNA-free (guide-unassigned) cells
- Null DE P values per gene from normalisr.de on random two-group splits
- Expression-level values used to define 10 gene bins

## Outputs

- Per-bin, per-split two-sided KS statistics and P values of null P values versus the uniform distribution
- Aggregated summary table, e.g. ks_uniformity_results.csv

## How to apply

After Normalisr QC (qc_reads, qc_outlier with prior outlier-proportion bound 2% and Bonferroni P <= 1e-10) and normalization (lcpm, normcov, compute_var, normvar), select the guide-unassigned cells (e.g. 4622 gRNA-free cells of the UPR Perturb-seq K562 dataset). Repeatedly partition these cells into two random groups (100 splits) and run normalisr.de on each split to obtain null P values for every gene. Stratify genes into 10 bins ordered by expression level, then apply a two-sided Kolmogorov–Smirnov test of each bin's null P values against the uniform distribution. Aggregate KS statistics and P values across bins and splits (e.g. into ks_uniformity_results.csv); uniformity in every bin indicates unbiased P-value estimation across expression levels.

## Related tools

- **Normalisr** (Generates the null DE P values (normalisr.de) on gRNA-free cell splits after lcpm/normcov/compute_var/normvar normalization, which are then tested for uniformity with KS) — https://github.com/lingfeiwang/normalisr

## Examples

```
import normalisr.normalisr as norm; p = norm.de(umi_counts, groups)  # then scipy.stats.kstool-free: scipy.stats.kstist/p via scipy.stats.kstest(bin_pvalues, 'uniform')
```

## Evaluation signals

- Two-sided KS test of null P values against Uniform(0,1) is not significant within each of the 10 expression bins and across all 100 splits
- Null P-value histograms are flat (no enrichment near 0 or 1) at both low- and high-expression bins
- QC retained cells per stated thresholds: outlier removal with prior outlier-proportion bound 2% and Bonferroni P <= 1e-10
- Exactly 100 random two-group partitions of the 4622 gRNA-free cells were tested and aggregated into ks_uniformity_results.csv

## Limitations

- Normalisr was only validated on UMI-based datasets; full-length platforms may distort read count distributions and require revalidation of the nonlinear covariates
- Uniformity holds for linear-model null DE under random groupings; it does not certify the method for arbitrary nonlinear association testing
- Batch effects in multi-dataset integrative studies are not addressed and could bias null P values

## Evidence

- [results] Unbiased P-value estimation across expression bins: [evidence span withheld — anchor status: not_found]
- [readme] Unbiased p-value estimation via nonlinear confounder removal: "Normalisr achieves high sensitivity, specificity, speed, and generalizability across multiple scRNA-seq protocols and experimental conditions with unbiased p-value estimation."
- [discussion] Outlier removal thresholds: "With the prior bound for outlier proportion as 2%, low-variance outlier cells with Bonferroni P ≤ 10−10 were removed"
- [readme] Python interface for the DE function: "import normalisr.normalisr as norm
		?norm.de"
