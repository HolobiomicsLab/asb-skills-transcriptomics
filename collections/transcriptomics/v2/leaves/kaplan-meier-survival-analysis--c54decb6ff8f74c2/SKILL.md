---
name: kaplan-meier-survival-analysis
description: Use when a cell-type or gene signature identified in single-cell data
  must be validated for association with overall survival and radiologic response
  in independent bulk RNA-seq cohorts with clinical metadata.
when_to_use_negative:
- The goal is differential expression or distance estimation on single-cell data itself
  — use scDist directly on normalized counts with fixed/random effects instead
- Clinical metadata (survival time/status or radiologic response) is missing for the
  bulk cohorts, making KM/log-rank testing impossible
- A cohort has fewer patients than needed for a median-split dichotomization to be
  meaningful in a log-rank comparison
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3308
tools:
- name: scDist
  role: Upstream tool that derives the NK-2 signature from FCER1G+ NK cells via mixed-effects
    distance estimation in the single-cell immunotherapy cohorts; its signature genes
    are the input scores tested in bulk
  repo: github.com/phillipnicol/scDist
- name: ggplot2
  role: Renders the Kaplan-Meier survival curves annotated with log-rank p-values
    for each cohort
- name: ggpubr
  role: Produces the responder-vs-non-responder signature score association plot and
    supports annotated survival figure assembly
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:4ece2f86bf3014bd20dd312f30d36f8a253d6ee57365674c04b60afccce56171
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dplyr
  - ggplot2
  - ggpubr
  - scDist
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# kaplan-meier-survival-analysis

## Summary

Validates a single-cell-derived cell-type signature (here, the scDist-derived NK-2 signature from FCER1G+ NK cells) in independent bulk RNA-seq cohorts by computing per-patient Spearman-based signature scores, dichotomizing at the median, and testing Kaplan-Meier survival differences with log-rank tests across 7 immunotherapy cohorts totaling 789 patients.

## When to use

Use when a cell-type or gene signature identified in single-cell data must be validated for association with overall survival and radiologic response in independent bulk RNA-seq cohorts with clinical metadata.

## When NOT to use

- The goal is differential expression or distance estimation on single-cell data itself — use scDist directly on normalized counts with fixed/random effects instead
- Clinical metadata (survival time/status or radiologic response) is missing for the bulk cohorts, making KM/log-rank testing impossible
- A cohort has fewer patients than needed for a median-split dichotomization to be meaningful in a log-rank comparison

## Inputs

- Bulk RNA-seq expression matrices for 7 cancer cohorts (789 patients total)
- Clinical metadata per patient: overall survival time and status
- Radiologic response annotations (partial/complete responders vs non-responders)
- NK-2 signature gene list derived from the scDist analysis of FCER1G+ NK cells

## Outputs

- Per-patient NK-2 signature scores, high/low strata assignments, and per-cohort log-rank p-values written to nk2_signature_survival_results.csv
- Combined Kaplan-Meier curves (annotated with log-rank p-values) and responder-vs-non-responder association plots written to nk2_km_response_plots.pdf

## How to apply

Load each bulk RNA-seq expression matrix with its matching clinical metadata (overall survival time/status and radiologic response) for the seven cancer cohorts covering 789 patients. Compute the NK-2 signature score per patient by aggregating expression of the NK-2 genes derived from the scDist analysis of FCER1G+ NK cells, using the Spearman-correlation-based signature score per ref. 39. Dichotomize patients within each cohort at the median signature score into high- and low-NK2 strata, fit Kaplan-Meier curves per stratum, and compute log-rank p-values comparing the strata. Render KM curves annotated with log-rank p-values via ggplot2/ggpubr, and separately compare signature scores between radiologic responders (partial/complete) and non-responders. The rationale is that transcriptomic perturbations scDist detected in single-cell immunotherapy cohorts should translate into measurable survival and response differences in independent bulk transcriptomes.

## Related tools

- **scDist** (Upstream tool that derives the NK-2 signature from FCER1G+ NK cells via mixed-effects distance estimation in the single-cell immunotherapy cohorts; its signature genes are the input scores tested in bulk) — github.com/phillipnicol/scDist
- **ggplot2** (Renders the Kaplan-Meier survival curves annotated with log-rank p-values for each cohort)
- **ggpubr** (Produces the responder-vs-non-responder signature score association plot and supports annotated survival figure assembly)

## Examples

```
devtools::install_github("phillipnicol/scDist")  # derive NK-2 signature genes, then fit per-cohort Kaplan-Meier strata and log-rank tests in R
```

## Evaluation signals

- Output file nk2_signature_survival_results.csv exists and contains exactly 789 scored patients summed across the seven cohorts
- Each of the 7 per-cohort outputs is present with per-patient signature scores and high/low strata assignments
- Log-rank p-values and responder-vs-non-responder contrasts qualitatively reproduce the significance pattern reported in the article (significant NK subpopulation differences between responders and non-responders)
- Dichotomization splits each cohort at its median NK-2 signature score (balanced high/low strata within cohort)
- PDF of figures renders KM curves with per-cohort log-rank p-value annotations

## Limitations

- Significant errors in clustering and annotation of the single-cell data could bias the upstream scDist signature that this bulk validation relies on
- Results are sensitive to the number of principal components used by scDist to estimate the distance (fixed heuristic of 20 or 50, no adaptive selection)
- scDist's assumption of spherical-normal (shared) variance, if violated, can yield miscalibrated standard errors and hypothesis tests for the derived signature

## Evidence

- [intro] Bulk validation rationale: "jointly inferring information from five single-cell immunotherapy cohorts, revealing significant differences in a subpopulation of NK cells between immunotherapy responders and non-responders"
- [abstract] Mixed-model distance estimation underpinning the signature: "we propose a computational tool, scDist, based on a mixed-effects model that provides a statistically rigorous and computationally efficient approach for detecting transcriptomic differences"
- [other] KM/log-rank strata testing procedure: [evidence span withheld — anchor status: not_found]
- [other] Signature scoring method: [evidence span withheld — anchor status: not_found]
- [discussion] Recommended upstream normalization: "We recommend using scTransform13 to normalize, although the method is compatible with any normalization approach."
