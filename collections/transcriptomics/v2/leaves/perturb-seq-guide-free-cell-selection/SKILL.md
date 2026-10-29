---
name: perturb-seq-guide-free-cell-selection
description: Use when a Perturb-seq/CRISPRi dataset needs guide-unassigned cells as
  a null cell population for evaluating null P-value calibration of single-cell DE
  methods.
when_to_use_negative:
- Cells with assigned guides are the target — guide-free cells only provide a null
  population and exclude gRNA trans-effect analyses
- Full-length (non-UMI) scRNA-seq platforms — the paper only validated nonlinear covariate
  removal on UMI-based datasets
- Hypothesis testing of arbitrary nonlinear associations, which Normalisr is explicitly
  not designed for
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Normalisr
  role: Provides qc_reads and qc_outlier for cell QC and the de module for the downstream
    null differential expression on guide-free cells
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:351e9de7e34e2b27f52150855b9aba912652c1ac495484cb0e85879dee2bfc47
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# perturb-seq-guide-free-cell-selection

## Summary

Selects gRNA-free (guide-unassigned) cells from a Perturb-seq K562 UPR study, after Normalisr QC, to build a null differential-expression benchmark on which uniformity of P values across expression bins can be tested.

## When to use

Use when a Perturb-seq/CRISPRi dataset needs guide-unassigned cells as a null cell population for evaluating null P-value calibration of single-cell DE methods.

## When NOT to use

- Cells with assigned guides are the target — guide-free cells only provide a null population and exclude gRNA trans-effect analyses
- Full-length (non-UMI) scRNA-seq platforms — the paper only validated nonlinear covariate removal on UMI-based datasets
- Hypothesis testing of arbitrary nonlinear associations, which Normalisr is explicitly not designed for

## Inputs

- UMI read count matrix (tsv or sparse Cell Ranger mtx format, optionally .gz) from the UPR Perturb-seq K562 study via GEO
- gRNA assignment matrix (guide-to-cell assignments, tsv) from the same GEO deposition

## Outputs

- Set of gRNA-free (guide-unassigned) QC-passed cells (4622 cells in the K562 UPR study)
- Partitions of these cells into two random groups across 100 splits, feeding normalisr.de null DE P values

## How to apply

Download the UMI read count and gRNA assignment matrices from GEO for the UPR Perturb-seq K562 study. Apply Normalisr QC (qc_reads) restricting to cells with at least 500 reads and 100 genes with non-zero reads, then qc_outlier removing low-variance outlier cells with a prior outlier-proportion bound of 2% and Bonferroni P ≤ 1e-10. From the QC-passed cells, use the gRNA assignment matrix to retain only cells with no assigned guide (the 4622 guide-free cells in this study). These cells are then repeatedly partitioned into two random groups (100 splits) and analyzed with normalisr.de to generate null P values for every gene, which are stratified into 10 expression-level bins and tested against the uniform distribution with two-sided Kolmogorov-Smirnov tests.

## Related tools

- **Normalisr** (Provides qc_reads and qc_outlier for cell QC and the de module for the downstream null differential expression on guide-free cells) — https://github.com/lingfeiwang/normalisr

## Examples

```
pip install normalisr; normalisr qc_reads -h; normalisr qc_outlier -h
```

## Evaluation signals

- All retained cells have no guide assignment in the gRNA matrix (4622 cells in the K562 run)
- QC thresholds met: ≥ 500 reads and ≥ 100 genes with non-zero reads per cell; outliers removed with prior bound 2% and Bonferroni P ≤ 1e-10
- Null DE P values across 100 two-group random splits yield two-sided KS tests not rejecting uniformity within each of the 10 expression bins
- Aggregated ks_uniformity_results.csv contains KS statistics and P values for all 10 bins across all 100 splits

## Limitations

- Validated only on UMI-based datasets; full-length platforms may require revalidation of nonlinear cellular summary covariates
- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations
- Batch effects in multi-dataset integrative studies are not addressed by Normalisr

## Evidence

- [discussion] prior outlier-proportion bound and Bonferroni cutoff: "With the prior bound for outlier proportion as 2%, low-variance outlier cells with Bonferroni P ≤ 10−10 were removed"
- [discussion] read-count QC floor: "we restricted our analyses to cells with at least 500 reads and 100 genes with non-zero reads"
- [readme] Normalisr scope over CRISPR screen analyses: "unifies single-cell differential expression, co-expression, and pooled single-cell CRISPR screen analyses with linear models"
- [methods] QC submodules: "qc_reads
	qc_outlier"
