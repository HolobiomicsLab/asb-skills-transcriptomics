---
name: bayesian-log-expression-mmse-estimation
description: Use when the input is a raw scRNA-seq UMI read count matrix (e.g. GSE120861
  highmoi, Cell Ranger sparse mtx or tsv) and normalized log expression is needed
  as the basis for Normalisr's downstream association testing.
when_to_use_negative:
- Input is full-length (non-UMI) scRNA-seq — Normalisr was validated only on UMI-based
  datasets, and the nonlinear covariates may need revalidation there
- You need hypothesis testing of arbitrary nonlinear associations or causal network
  reconstruction (v-structures, regularized regression, Granger causality) — Normalisr
  explicitly does not support these
- Analysis spans multiple datasets requiring batch-effect correction — integrative
  studies may need additional consideration for batch effects beyond Normalisr
tools:
- name: Normalisr
  role: Provides the parameter-free lcpm and scaling_factor functions that implement
    the MMSE (digamma-based) Bayesian posterior estimator for log relative gene expression
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:2751cd38ab80ed1cc233ee1441a7193b00e55cf05e24a21a57c3e0815acbbfbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# bayesian-log-expression-mmse-estimation

## Summary

Estimate per-cell, per-gene log relative expression from raw UMI read counts using Normalisr's Minimum Mean Square Error (MMSE) digamma-based Bayesian posterior estimator — the 'Bayesian log expression' — which recovers pre-measurement mRNA frequencies before nonlinear covariate regression and variance normalization.

## When to use

Use when the input is a raw scRNA-seq UMI read count matrix (e.g. GSE120861 highmoi, Cell Ranger sparse mtx or tsv) and normalized log expression is needed as the basis for Normalisr's downstream association testing.

## When NOT to use

- Input is full-length (non-UMI) scRNA-seq — Normalisr was validated only on UMI-based datasets, and the nonlinear covariates may need revalidation there
- You need hypothesis testing of arbitrary nonlinear associations or causal network reconstruction (v-structures, regularized regression, Granger causality) — Normalisr explicitly does not support these
- Analysis spans multiple datasets requiring batch-effect correction — integrative studies may need additional consideration for batch effects beyond Normalisr

## Inputs

- Raw scRNA-seq UMI read count matrix in tsv or Cell Ranger sparse mtx format (genes × cells); gzipped files recognized by '.gz' suffix
- Optional: cell QC filters applied upstream (≥500 reads, ≥100 genes with non-zero reads per cell)

## Outputs

- Bayesian log expression matrix (MMSE posterior estimates of log relative gene expression, one value per gene per cell) in tsv format

## How to apply

Load the raw UMI count matrix (tsv/sparse mtx, gzipped inputs auto-detected) with genes as rows and cells as columns. Apply Normalisr's normalization functions in order: scaling_factor computes the scaling factors from total read counts per cell, lcpm converts counts into Bayesian log expression via the MMSE posterior estimator yij≡E[ln g~ij]. Optionally run quality control first (qc_reads, qc_outlier) restricting to cells with at least 500 reads and 100 genes with non-zero reads, and genes with non-zero reads in at least 50 cells and at least 2% of cells for 10x technologies. The MMSE estimator is parameter-free, so no tuning is required at this stage; subsequent nonlinear library-size confounder removal happens in the separate normcov/compute_var/normvar steps, not here.

## Related tools

- **Normalisr** (Provides the parameter-free lcpm and scaling_factor functions that implement the MMSE (digamma-based) Bayesian posterior estimator for log relative gene expression) — https://github.com/lingfeiwang/normalisr

## Examples

```
import normalisr.normalisr as norm; ?norm.lcpm
```

## Evaluation signals

- Output matrix is on the log-relative-expression scale (values ≪ raw UMI counts), consistent with lcpm (log counts per million with posterior correction)
- Pipeline order is correct: scaling_factor/lcpm precede normcov, then compute_var, then normvar
- Cells passing upstream QC match the ≥500 reads / ≥100 genes-with-non-zero-reads threshold and genes meet the ≥50 cells and ≥2% threshold for 10x data
- No tuning parameters were introduced — Normalisr is non-parametric and only QC and final cutoffs are adjustable
- Row/column names preserved as one gene/cell per line in tsv output as Normalisr's format requires

## Limitations

- Validated only on UMI-based datasets; full-length platforms produce different read-count distributions and would need revalidation of the covariates
- Bayesian log expression still carries mean-level library-size confounding at this stage — nonlinear covariate regression (normcov) and variance normalization (normvar) are separate downstream steps
- Outlier cells should be removed before/alongside this step (inverse cell variance modeled with a normal distribution; prior outlier proportion bound 2%, Bonferroni P ≤ 10−10)

## Evidence

- [discussion] MMSE estimator defines Bayesian log expression: [evidence span withheld — anchor status: not_found]
- [intro] Normalization estimates pre-measurement mRNA frequencies from UMI counts: "The normalization step estimates the pre-measurement mRNA frequencies from the scRNA-seq read counts (e.g., unique molecular identifiers (UMIs)) and regresses out the nonlinear effect of library size"
- [methods] Normalization function list including lcpm: "Normalization
==============
.. autosummary::
	:toctree:
	:nosignatures:

	lcpm
	normcov
	scaling_factor
	compute_var
	normvar"
- [readme] README on removing technical noise from raw counts: "Normalisr first removes confounding technical noises from raw read counts to recover the biological variations."
- [discussion] Variance-level confounding beyond the mean: "Covariates may confound (Bayesian log) expression at the variance level in addition to the mean"
