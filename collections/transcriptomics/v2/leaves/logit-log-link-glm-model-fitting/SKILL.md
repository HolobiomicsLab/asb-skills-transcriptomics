---
name: logit-log-link-glm-model-fitting
description: Use when you need to distinguish true gene fusions from artificial chimeric-read
  noise by regressing per-cell fusion supporting-read counts on gene-expression covariates
  and GC-content splines, then testing candidates for significance.
when_to_use_negative:
- Input is raw FASTQ reads rather than an already-extracted candidate fusion feature
  table (candidate counting must happen first in the scFusion pipeline)
- The goal is filtering alignment artifacts such as low-mappability regions — use
  the mappability/ad hoc filters instead of the ZINB significance model
- Count data are not zero-inflated single-cell counts; a standard NB or Poisson GLM
  without the logit zero-inflation component is more appropriate
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scFusion
  role: Pipeline whose statistical model implements the ZINB logit/log-link regression
    for fusion significance testing
  repo: https://github.com/ZijieJin/scFusion
- name: Python 3
  role: Implementation environment for fitting the ZINB regression and resampling-based
    p-value computation
- name: R
  role: R >= 3.5 used within the scFusion pipeline (with stringr) for supporting statistical
    scripting
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28661-6
    title: Single-cell gene fusion detection by scFusion
  source_content_hash: sha256:e1a9292bdb659f6022cb06cf7d30db985dca625d7ea6f093729dbc9aedbba155
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
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Logit-log-link GLM model fitting

## Summary

Fits a zero-inflated negative binomial (ZINB) regression with a logit link for the dropout (zero-inflation) component and a log link for the count-mean component, as used in scFusion to model per-cell chimeric read counts of fusion candidates against background technical noise.

## When to use

Use when you need to distinguish true gene fusions from artificial chimeric-read noise by regressing per-cell fusion supporting-read counts on gene-expression covariates and GC-content splines, then testing candidates for significance.

## When NOT to use

- Input is raw FASTQ reads rather than an already-extracted candidate fusion feature table (candidate counting must happen first in the scFusion pipeline)
- The goal is filtering alignment artifacts such as low-mappability regions — use the mappability/ad hoc filters instead of the ZINB significance model
- Count data are not zero-inflated single-cell counts; a standard NB or Poisson GLM without the logit zero-inflation component is more appropriate

## Inputs

- Test candidate fusion feature table with per-cell fusion candidate counts (chimeric supporting reads)
- Partner gene expression covariates per candidate
- Local GC content values near each candidate's breakpoints

## Outputs

- fusion_zinb_pvalues.csv containing candidate identifiers, raw p-values, and FDR estimates
- Fitted ZINB regression model (logit zero-inflation component + log count-mean component)

## How to apply

Load a candidate fusion feature table containing per-cell fusion candidate counts, partner-gene expression covariates, and local GC-content values near the breakpoints. Fit the ZINB regression per Methods equations 1-4: the zero-inflation (dropout) component uses a logit link and the count-mean component uses a log link, with linear gene-expression covariates and spline (nonlinear) terms for GC content. Compute resampling-based p-values for each candidate by repeatedly refitting the model on resampled/permuted data and comparing the observed statistic against the resampling null distribution. Apply false discovery rate control across all candidate p-values to obtain adjusted FDR estimates. The rationale is that scFusion employs a statistical model (this ZINB GLM) together with a deep-learning model to control for false positives arising from the heavy amplification step that generates artificial chimeric reads.

## Related tools

- **scFusion** (Pipeline whose statistical model implements the ZINB logit/log-link regression for fusion significance testing) — https://github.com/ZijieJin/scFusion
- **Python 3** (Implementation environment for fitting the ZINB regression and resampling-based p-value computation)
- **R** (R >= 3.5 used within the scFusion pipeline (with stringr) for supporting statistical scripting)

## Examples

```
docker run -v XXX:/data --rm jzj2035198/scfusion python -u /usr/local/src/scFusion/scFusion.py [commands]
```

## Evaluation signals

- Every test candidate receives a raw p-value in [0,1]
- Every test candidate receives a corresponding FDR estimate in [0,1]
- The fitted model includes both a logit-link zero-inflation term and a log-link count-mean term with spline GC-content covariates, consistent with Methods equations 1-4
- Resampling null distribution contains enough permuted refits to yield stable per-candidate p-values
- Low false discovery rate is maintained on simulated validation data where known true fusions are planted

## Limitations

- scFusion has reduced sensitivity for fusions with low expression, as with all RNA-based methods
- Detection power is limited for rare fusions in highly heterogeneous tumor samples
- Limitations can only be overcome by sequencing more cells and/or sequencing each cell deeper

## Evidence

- [intro] ZINB model rationale: "scFusion employs a statistical model and a deep-learning model to control for false positives."
- [intro] Amplification noise motivation: "the heavy amplification step may generate artificial chimeric reads, leading to false-positive fusion candidates"
- [methods] Pipeline entry point: "Download all scripts, unzip the hg19mappability file in the data folder, and run scFusion.py."
- [abstract] Low FDR evaluation claim: "scFusion can efficiently and sensitively detect fusions with a low false discovery rate."
- [discussion] Low-expression sensitivity limitation: "scFusion has reduced sensitivity for fusions with low expression (as shown in the simulation), just as is the case for all RNA-based methods"
