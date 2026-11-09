---
name: zero-inflated-negative-binomial-regression-implementation
description: Use when a task requires implementing or refitting a zero-inflated negative
  binomial regression to distinguish true fusion candidates from background technical
  chimeric-read noise in per-cell scRNA-seq candidate feature tables with expression
  and GC-content covariates.
when_to_use_negative:
- Input data are 10X droplet-based scRNA-seq reads — scFusion's pipeline (and thus
  this statistical step) targets Smart-Seq data with *_1.fastq/*_2.fastq files
- Input is already a scored candidate list with p-values/FDR, or bulk RNA-seq data
  where the per-cell zero-inflation (dropout) component and joint multi-cell analysis
  do not apply
- The goal is candidate fusion calling from raw reads (alignment, chimera detection)
  rather than statistical significance testing of an existing candidate feature table
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3308
tools:
- name: scFusion
  role: Source pipeline whose statistical model (Methods equations 1-4) defines the
    ZINB regression, resampling p-value, and FDR procedures being reimplemented
  repo: https://github.com/ZijieJin/scFusion
- name: Python 3
  role: Implementation environment for fitting the ZINB model and computing resampling-based
    p-values
- name: R >= 3.5
  role: Runtime used within the scFusion pipeline, including stringr, from which the
    statistical testing step draws covariates and writes candidate tables
- name: pyensembl
  role: Provides Ensembl gene/transcript annotation used to derive gene-expression
    covariates for the regression
  repo: https://github.com/openvax/pyensembl
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:2ac14691788123419be86bc8ee9f1a247d02d49d8f1b0384e7854447e741f618
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
  - Python
  - Python 3
  - R
  - R >= 3.5
  - pyensembl
  - scFusion
  - stringr
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# zero-inflated-negative-binomial-regression-implementation

## Summary

Reconstruction of scFusion's statistical significance model for scRNA-seq gene fusion candidates: a zero-inflated negative binomial regression in which supporting chimeric read counts are modeled with a logit link for the zero-inflation (dropout) component and a log link for the count-mean component, using linear gene-expression covariates and spline terms for local GC content, followed by resampling-based p-values and FDR control.

## When to use

Use when a task requires implementing or refitting a zero-inflated negative binomial regression to distinguish true fusion candidates from background technical chimeric-read noise in per-cell scRNA-seq candidate feature tables with expression and GC-content covariates.

## When NOT to use

- Input data are 10X droplet-based scRNA-seq reads — scFusion's pipeline (and thus this statistical step) targets Smart-Seq data with *_1.fastq/*_2.fastq files
- Input is already a scored candidate list with p-values/FDR, or bulk RNA-seq data where the per-cell zero-inflation (dropout) component and joint multi-cell analysis do not apply
- The goal is candidate fusion calling from raw reads (alignment, chimera detection) rather than statistical significance testing of an existing candidate feature table

## Inputs

- Per-cell fusion candidate feature table with supporting chimeric read counts for each candidate
- Partner gene-expression covariates for each fusion candidate
- GC-content values measured near the fusion breakpoints
- GTF annotation (via pyensembl) and reference genome FASTA used to derive gene-expression and GC covariates in the scFusion pipeline

## Outputs

- fusion_zinb_pvalues.csv with candidate identifiers, raw resampling-based p-values, and FDR estimates
- Fitted ZINB model parameters (logit zero-inflation component and log count-mean component with expression and GC spline terms)

## How to apply

Load the per-cell fusion candidate feature table containing chimeric supporting read counts per cell, partner gene-expression covariates, and GC-content values near the breakpoints. Fit the zero-inflated negative binomial regression per Methods equations 1-4: a logit link models the zero-inflation (dropout) component and a log link models the count-mean component, with linear terms for gene expression and nonlinear spline terms for GC content, capturing the heavy amplification artifacts that generate artificial chimeric reads. Compute resampling-based p-values by repeatedly refitting the model on resampled/permuted data and comparing each candidate's observed statistic against the resampling null distribution. Apply false discovery rate control across all candidate p-values to obtain adjusted FDR estimates, then write candidate identifiers, raw p-values, and FDR estimates to fusion_zinb_pvalues.csv. In scFusion this statistical model works alongside a deep-learning model and ad hoc/mappability filters to control false positives across jointly analyzed single cells.

## Related tools

- **scFusion** (Source pipeline whose statistical model (Methods equations 1-4) defines the ZINB regression, resampling p-value, and FDR procedures being reimplemented) — https://github.com/ZijieJin/scFusion
- **Python 3** (Implementation environment for fitting the ZINB model and computing resampling-based p-values)
- **R >= 3.5** (Runtime used within the scFusion pipeline, including stringr, from which the statistical testing step draws covariates and writes candidate tables)
- **pyensembl** (Provides Ensembl gene/transcript annotation used to derive gene-expression covariates for the regression) — https://github.com/openvax/pyensembl

## Examples

```
from statsmodels.discrete.count_model import ZeroInflatedNegativeBinomialP; zinb = ZeroInflatedNegativeBinomialP(counts, X_expr_gc, exog_infl=X_infl, inflation='logit').fit(method='nm', maxiter=500)
```

## Evaluation signals

- Every test candidate receives a raw p-value in [0,1] and a corresponding FDR estimate in [0,1]
- FDR estimates are monotonically consistent with the ordering of raw p-values (adjusted p-values never smaller than raw ones for the same candidate)
- The fitted model uses the correct link structure: logit link for the zero-inflation component and log link for the count-mean component, per Methods equations 1-4
- The resampling null distribution is built from repeated refits on resampled/permuted data, and observed statistics are compared against it rather than against an asymptotic approximation
- On simulated scRNA-seq data built from real datasets, detected true fusions show low false discovery rate, consistent with scFusion's reported sensitivity/FDR performance

## Limitations

- scFusion has reduced sensitivity for fusions with low expression, as is the case for all RNA-based methods
- Detection power is limited for rare fusions in highly heterogeneous tumor samples; overcoming this requires sequencing more cells and/or deeper sequencing per cell
- The heavy amplification step of scRNA-seq can generate artificial chimeric reads that inflate the background-noise distribution the model must control for

## Evidence

- [intro] scFusion employs a statistical model and a deep-learning model to control for false positives.: "scFusion employs a statistical model and a deep-learning model to control for false positives."
- [intro] Heavy amplification generates false candidates: "the heavy amplification step may generate artificial chimeric reads, leading to false-positive fusion candidates"
- [intro] Low FDR on simulated data: "we first simulated single-cell data based on real scRNA-seq data and found that scFusion achieved high sensitivity while maintaining a low false discovery rate"
- [abstract] Efficient, sensitive detection with low FDR: "We evaluate the performance of scFusion using simulated and five real scRNA-seq datasets and find that scFusion can efficiently and sensitively detect fusions with a low false discovery rate."
- [discussion] Joint analysis of single cells: "Another important difference compared to existing bulk methods is that scFusion takes advantage of the fact that multiple cells from a sample should contain the same fusion."
- [readme] Python/R runtime for the pipeline: "- python 3
- R >= 3.5 
- R package: stringr
- python module: [pyensembl](https://github.com/openvax/pyensembl)"
