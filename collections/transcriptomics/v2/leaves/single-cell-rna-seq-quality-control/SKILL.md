---
name: single-cell-rna-seq-quality-control
description: Use when raw UMI count matrices (e.g., GSE123139 MARS-seq melanoma or
  GSE120861 Perturb-seq) are the input to Normalisr differential-expression or co-expression
  analyses and low-read/low-gene cells, sparsely detected genes, or outlier cells
  must be filtered first.
when_to_use_negative:
- Data are from full-length (non-UMI) platforms — the article states only UMI-based
  datasets were analyzed and the nonlinear cellular summary covariates may need revalidation
  there
- Integrative studies spanning multiple datasets that require explicit batch-effect
  handling, which Normalisr does not address
- Hypothesis testing of arbitrary nonlinear associations between genes, which Normalisr
  is explicitly not designed for
edam_operation: http://edamontology.org/operation_2930
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Normalisr
  role: Provides the QC submodules qc_reads (cell read/gene thresholds) and qc_outlier
    (iterative outlier removal), plus the normalization functions (lcpm, normcov,
    scaling_factor, compute_var, normvar) that consume the QC-filtered matrix
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:85cbb1ffbe25962689c7bc19091a70f01a9bc4e40ffe1e9227fc542f96bf641d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - BioServices
  - GOATOOLS
  - Normalisr
  edam_operation: http://edamontology.org/operation_2930
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-rna-seq-quality-control

## Summary

Normalisr-based quality control for UMI-based scRNA-seq: restrict cells to ≥500 reads and ≥100 detected genes, restrict genes by detection frequency (≥500 cells for MARS-seq, or ≥50 cells and ≥2% of cells for 10x), then iteratively remove low-variance outlier cells via Normalisr's qc_outlier before normalization and association testing.

## When to use

Use when raw UMI count matrices (e.g., GSE123139 MARS-seq melanoma or GSE120861 Perturb-seq) are the input to Normalisr differential-expression or co-expression analyses and low-read/low-gene cells, sparsely detected genes, or outlier cells must be filtered first.

## When NOT to use

- Data are from full-length (non-UMI) platforms — the article states only UMI-based datasets were analyzed and the nonlinear cellular summary covariates may need revalidation there
- Integrative studies spanning multiple datasets that require explicit batch-effect handling, which Normalisr does not address
- Hypothesis testing of arbitrary nonlinear associations between genes, which Normalisr is explicitly not designed for

## Inputs

- Raw UMI read count matrix in tsv format (or Cell Ranger sparse mtx), with gene names one per line and cell names one per line in text files
- Technology flag distinguishing 10x vs MARS-seq-type (plate-based) data, which selects the gene detection threshold

## Outputs

- QC-filtered cell-by-gene UMI count matrix in tsv format suitable for Normalisr normalization (lcpm, normcov, compute_var, normvar)
- Set of retained cells after iterative outlier removal at Bonferroni P ≤ 10−10 with 2% prior bound

## How to apply

First apply cell-level QC on the raw UMI matrix, retaining cells with at least 500 reads and 100 genes with non-zero reads. Then filter genes with non-zero reads in at least 500 cells for MARS-seq-type data (or at least 50 cells and at least 2% of all cells for 10x data). Next run Normalisr's iterative outlier removal (qc_outlier): inverse of the estimated cell variance is modeled with a normal distribution, using a prior outlier proportion bound of 2%, and low-variance outlier cells with Bonferroni P ≤ 10−10 are removed iteratively. Normalisr is non-parametric, so these QC thresholds and final association cutoffs (e.g., co-expression Q ≤ 10−15) are the only adjustable parameters; feed the filtered tsv count matrix into lcpm/normcov/normvar normalization downstream.

## Related tools

- **Normalisr** (Provides the QC submodules qc_reads (cell read/gene thresholds) and qc_outlier (iterative outlier removal), plus the normalization functions (lcpm, normcov, scaling_factor, compute_var, normvar) that consume the QC-filtered matrix) — https://github.com/lingfeiwang/normalisr

## Examples

```
import normalisr.normalisr as norm; ?norm.qc_outlier
```

## Evaluation signals

- All retained cells have ≥500 reads and ≥100 genes with non-zero reads; all retained genes have non-zero reads in ≥500 cells (MARS-seq) or ≥50 cells and ≥2% of cells (10x)
- Removed outlier fraction is consistent with the 2% prior bound and each removed cell has outlier Bonferroni P ≤ 10−10
- Output matrix is in tsv format with row/column name text files listing cells and genes one per line, recognized by Normalisr (gzipped files accepted via '.gz' suffix)
- Downstream co-expression P-values are unbiased (no inflation), since QC thresholds are the only adjustable parameters in the non-parametric pipeline
- Re-running the pipeline is deterministic: Normalisr has no tuning parameters other than QC thresholds and final association cutoffs

## Limitations

- Only UMI-based datasets were analyzed; full-length platforms may produce different read-count distributions and the nonlinear cellular summary covariates require revalidation
- Normalisr is robust against read count distribution distortions only with enough (> 100) cells per analysis
- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations, nor for causal network reconstruction or multi-dataset batch-effect correction

## Evidence

- [discussion] cell read/gene minimum thresholds: "we restricted our analyses to cells with at least 500 reads and 100 genes with non-zero reads"
- [discussion] gene detection threshold by technology: "genes with non-zero reads both in at least 50 cells and in at least 2% of all cells for 10× technologies, or in at least 500 cells"
- [discussion] outlier removal parameters: "With the prior bound for outlier proportion as 2%, low-variance outlier cells with Bonferroni P ≤ 10−10 were removed"
- [discussion] outlier model: "For outlier removal, inverse of (estimated) cell variance was modeled with normal distribution"
- [readme] input/output file formats: [evidence span withheld — anchor status: external_doc]
