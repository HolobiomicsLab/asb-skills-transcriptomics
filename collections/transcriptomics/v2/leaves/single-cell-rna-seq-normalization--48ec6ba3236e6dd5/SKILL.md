---
name: single-cell-rna-seq-normalization
description: Use when raw scRNA-seq UMI count matrices must be normalized before association
  testing (DE, co-expression, or CRISPR screen) and unbiased, uniformly distributed
  null P values across gene expression levels are required.
when_to_use_negative:
- Input is full-length (non-UMI) scRNA-seq data — the study only focused on UMI-based
  datasets, and nonlinear covariates may need revalidation on full-length platforms
- The goal is hypothesis testing of arbitrary nonlinear associations, not linear DE/co-expression/CRISPR
  screen inference
- An integrative analysis spanning multiple datasets requires batch-effect correction,
  which Normalisr does not address
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Normalisr
  role: Provides the entire normalization pipeline (qc_reads, qc_outlier, lcpm, normcov,
    scaling_factor, compute_var, normvar) via CLI or Python functional interface
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:aa61c4044bad177636482250c55237fb0684077890a023509d3e1b2064ecadf4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - BioServices
  - GOATOOLS
  - Normalisr
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# single-cell-rna-seq-normalization

## Summary

Normalizes raw scRNA-seq UMI read counts with the Normalisr normalization module (lcpm, normcov, scaling_factor, compute_var, normvar) so that nonlinear library-size confounders are removed at both mean and variance levels, yielding unbiased P values for downstream differential expression, co-expression, and CRISPR screen testing.

## When to use

Use when raw scRNA-seq UMI count matrices must be normalized before association testing (DE, co-expression, or CRISPR screen) and unbiased, uniformly distributed null P values across gene expression levels are required.

## When NOT to use

- Input is full-length (non-UMI) scRNA-seq data — the study only focused on UMI-based datasets, and nonlinear covariates may need revalidation on full-length platforms
- The goal is hypothesis testing of arbitrary nonlinear associations, not linear DE/co-expression/CRISPR screen inference
- An integrative analysis spanning multiple datasets requires batch-effect correction, which Normalisr does not address

## Inputs

- Raw scRNA-seq UMI read count matrix (tsv, text, or Cell Ranger sparse mtx; .gz suffix auto-recognized)
- gRNA assignment matrix (for Perturb-seq / CRISPR screen analyses)
- Row/column name text files listing cells and genes, one per line

## Outputs

- Normalized expression matrix with nonlinear library-size confounders removed at mean and variance levels
- Estimated cell variances and per-gene variance-normalized values (from compute_var and normvar)
- QC-filtered cell and gene sets (qc_reads, qc_outlier outputs)

## How to apply

Start from a raw UMI read-count matrix (tsv, text, or Cell Ranger mtx; gzipped files auto-recognized) and apply Normalisr QC first: qc_reads to restrict to cells with at least 500 reads and 100 genes with non-zero reads, and qc_outlier with a prior outlier-proportion bound of 2% and Bonferroni P ≤ 10−10 to drop low-variance outlier cells. Then run the normalization sequence lcpm, normcov, scaling_factor, compute_var, normvar, which estimates pre-measurement mRNA frequencies and regresses out the nonlinear effect of library size via a restricted forward stepwise selection of nonlinear cellular summary covariates. Normalisr is parameter-free by design, so beyond QC and final significance cutoffs (e.g., co-expression Q ≤ 10−15 for dysfunctional T cell networks) no tunable parameters are adjusted. The rationale is that normalization methods designed for clustering leave sensitivity, specificity, and effect size estimation in association testing uncharted, whereas Normalisr's mean-and-variance-level correction yields uniform null P values across expression bins. Output normalized matrices feed directly into normalisr.de (differential expression), normalisr.coex (conditional Pearson co-expression), or CRISPR screen linear-model analyses.

## Related tools

- **Normalisr** (Provides the entire normalization pipeline (qc_reads, qc_outlier, lcpm, normcov, scaling_factor, compute_var, normvar) via CLI or Python functional interface) — https://github.com/lingfeiwang/normalisr

## Examples

```
import normalisr.normalisr as norm; ?norm.de
```

## Evaluation signals

- Null P values from downstream tests (e.g., 100 random group splits of 4622 gRNA-free K562 cells with normalisr.de) stratified into 10 expression bins should pass two-sided Kolmogorov-Smirnov tests against the uniform [0,1] distribution with no significant departure
- The normalization sequence must include both mean-level (lcpm, normcov, scaling_factor) and variance-level (compute_var, normvar) steps; skipping normvar leaves variance-level confounding uncorrected
- qc_outlier parameters must be prior outlier-proportion bound 2% and Bonferroni P ≤ 10−10; cells retained should satisfy ≥500 reads and ≥100 detected genes
- Normalized output matrix should be in tsv format compatible with normalisr.de and normalisr.coex downstream modules
- Per-bin KS statistics and P values (e.g., bin_pair_ks_results.csv or ks_uniformity_results.csv) should show uniformity across all 55 bin pairs or 10 expression bins, respectively

## Limitations

- Validated only on UMI-based datasets; full-length platforms may have a different read-count distribution requiring revalidation of the nonlinear cellular summary covariates
- Not designed for hypothesis testing of arbitrary nonlinear associations
- Batch effects in multi-dataset integrative studies require additional consideration outside Normalisr
- Does not reconstruct causal regulatory networks (v-structures, regularized regression) or Granger causality with longitudinal data

## Evidence

- [readme] Normalisr first removes confounding technical noises from raw read counts to recover the biological variations.: "Normalisr first removes confounding technical noises from raw read counts to recover the biological variations."
- [abstract] Nonlinear library-size confounder removal at mean and variance levels: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity, specificity, speed, and generalizability"
- [discussion] Outlier removal parameters: "With the prior bound for outlier proportion as 2%, low-variance outlier cells with Bonferroni P ≤ 10−10 were removed"
- [methods] Normalization function inventory: "Normalization
==============
.. autosummary::
	:toctree:
	:nosignatures:

	lcpm
	normcov
	scaling_factor
	compute_var
	normvar"
- [intro] Nonlinear library-size effect regression: "The normalization step estimates the pre-measurement mRNA frequencies from the scRNA-seq read counts"
