---
name: false-discovery-rate-control
description: Use when a table of per-cell candidate fusion supporting-read counts
  must be tested against a background-noise model, and multiple-testing-adjusted (FDR)
  significance calls are required to control false positives.
when_to_use_negative:
- Input is bulk RNA-seq or 10X scRNA-seq data — scFusion is designed for Smart-Seq
  single-cell data only
- Candidate calls already carry adjusted p-values from another framework, so no ZINB
  background-noise model is needed
- The goal is fusion read mapping/candidate enumeration rather than statistical significance
  testing — the alignment stage (STAR/samtools/bedtools) should be used instead
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3361
tools:
- name: scFusion
  role: Implements the ZINB regression, resampling-based p-value computation, and
    FDR control for fusion candidates
  repo: ZijieJin/scFusion
- name: Python 3
  role: Runtime for the scFusion statistical model (alongside R >= 3.5, stringr, and
    pyensembl)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:5b6f274f97632f243ba0fe5bf78beb104e5ba6972a174344a6da51c80a437034
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ZijieJin/scFusion
  tools:
  - Python
  - Python 3
  - R
  - pyensembl
  - scFusion
  - stringr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# false-discovery-rate-control

## Summary

Statistical significance testing of single-cell fusion candidates via a zero-inflated negative binomial (ZINB) regression followed by resampling-based p-values and false discovery rate control, as implemented in scFusion to separate true gene fusions from background technical noise in supporting chimeric read counts.

## When to use

Use when a table of per-cell candidate fusion supporting-read counts must be tested against a background-noise model, and multiple-testing-adjusted (FDR) significance calls are required to control false positives.

## When NOT to use

- Input is bulk RNA-seq or 10X scRNA-seq data — scFusion is designed for Smart-Seq single-cell data only
- Candidate calls already carry adjusted p-values from another framework, so no ZINB background-noise model is needed
- The goal is fusion read mapping/candidate enumeration rather than statistical significance testing — the alignment stage (STAR/samtools/bedtools) should be used instead

## Inputs

- Test candidate fusion feature table with per-cell fusion candidate counts, gene-expression covariates, and GC-content values near the breakpoints for each candidate
- scRNA-seq data from the Smart-Seq protocol (*_1.fastq, *_2.fastq) processed through the scFusion pipeline (STAR, samtools, bedtools)
- Reference genome FASTA (*.fa) and GTF annotation (*.gtf), plus optional hg19 mappability file for the mappability filter

## Outputs

- fusion_zinb_pvalues.csv containing candidate identifiers, raw p-values, and FDR estimates
- Per-candidate significance calls identifying fusions exceeding the background-noise distribution

## How to apply

Fit a zero-inflated negative binomial regression to the per-cell supporting chimeric read counts for each fusion candidate: a logit link models the zero-inflation (dropout) component and a log link models the count-mean component, with linear terms for the partner genes' expression covariates and nonlinear spline terms for local GC content near the breakpoints (Methods equations 1-4). Compute resampling-based p-values by repeatedly refitting the model on resampled/permuted data and comparing each candidate's observed statistic against the resampling null distribution. Apply an FDR control procedure across all candidate p-values to produce adjusted FDR estimates, then write candidate identifiers, raw p-values, and FDR estimates to fusion_zinb_pvalues.csv. This statistical model works together with scFusion's deep-learning model and ad hoc filters (e.g. the mappability filter) to keep the false discovery rate low while retaining sensitivity.

## Related tools

- **scFusion** (Implements the ZINB regression, resampling-based p-value computation, and FDR control for fusion candidates) — ZijieJin/scFusion
- **Python 3** (Runtime for the scFusion statistical model (alongside R >= 3.5, stringr, and pyensembl))

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- Every test candidate receives a raw p-value in [0,1]
- Every test candidate receives a corresponding FDR estimate in [0,1]
- Adjusted FDR values are consistent with (i.e., not below) the raw p-values across candidates
- On simulated single-cell data derived from real scRNA-seq, sensitivity remains high while the false discovery rate stays low
- fusion_zinb_pvalues.csv is written with all three required columns (candidate ID, raw p-value, FDR) populated

## Limitations

- Reduced sensitivity for fusions with low expression, common to all RNA-based methods
- Detection power is limited for rare fusions in highly heterogeneous tumor samples; overcoming this requires sequencing more cells and/or sequencing each cell deeper
- Choosing appropriate thresholds to define distinct cell populations (e.g. by marker gene expression or transcriptional profile similarity) remains difficult

## Evidence

- [intro] scFusion employs a statistical model and a deep-learning model to control for false positives.: "scFusion employs a statistical model and a deep-learning model to control for false positives."
- [abstract] scFusion can efficiently and sensitively detect fusions with a low false discovery rate: "scFusion can efficiently and sensitively detect fusions with a low false discovery rate."
- [readme] scFusion is a computational pipeline for detecting gene fusions at single-cell resolution (For Smart-Seq data rather than 10X).: [evidence span withheld — anchor status: external_doc]
