---
name: log-rank-hypothesis-testing
description: Use when a cell-type or gene signature discovered in single-cell data
  needs independent validation in bulk transcriptome cohorts that have matched overall
  survival time/status and radiologic response metadata.
when_to_use_negative:
- The goal is detecting perturbed cell types in single-cell RNA-seq data itself —
  that requires scDist differential state analysis, not bulk survival validation
- Bulk cohorts lack matched survival time/status or response metadata, so no KM strata
  or log-rank comparison can be constructed
- Signature scores are continuous predictors and you need multivariable covariate
  adjustment rather than a two-stratum median split
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: scDist
  role: Source of the NK-2 signature genes derived from FCER1G+ NK cells, whose bulk
    RNA-seq score is survival-tested
  repo: https://github.com/phillipnicol/scDist
- name: ggplot2
  role: Renders the Kaplan-Meier survival curves for the visualization output
- name: ggpubr
  role: Annotates KM curves with log-rank p-values and produces the responder-vs-non-responder
    association plot
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:8ba3412848f32409be6c396014a4546e4718da3dd6842758df9143748bd53e61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/phillipnicol/scDist
  tools:
  - ggplot2
  - ggpubr
  - scDist
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# log-rank-hypothesis-testing

## Summary

Validates single-cell-derived signatures (e.g., the NK-2 signature from scDist analysis of FCER1G+ NK cells) in independent bulk RNA-seq cohorts by computing per-patient Spearman-correlation-based signature scores, dichotomizing at the median, and testing Kaplan-Meier survival differences with log-rank tests.

## When to use

Use when a cell-type or gene signature discovered in single-cell data needs independent validation in bulk transcriptome cohorts that have matched overall survival time/status and radiologic response metadata.

## When NOT to use

- The goal is detecting perturbed cell types in single-cell RNA-seq data itself — that requires scDist differential state analysis, not bulk survival validation
- Bulk cohorts lack matched survival time/status or response metadata, so no KM strata or log-rank comparison can be constructed
- Signature scores are continuous predictors and you need multivariable covariate adjustment rather than a two-stratum median split

## Inputs

- Bulk RNA-seq expression matrices (gene x patient) for each validation cohort
- Clinical metadata with overall survival time and status per patient
- Radiologic response annotations (partial/complete response vs non-responder)
- NK-2 signature gene list from scDist analysis of FCER1G+ NK cells

## Outputs

- Per-patient signature scores and median-based strata assignments (nk2_signature_survival_results.csv)
- Per-cohort log-rank p-values comparing high vs low signature groups
- Kaplan-Meier curves annotated with log-rank p-values and responder-vs-non-responder association plots (nk2_km_response_plots.pdf)

## How to apply

Load bulk RNA-seq expression matrices with matched clinical metadata (overall survival time/status, radiologic response) for the validation cohorts — in the source work, seven cancer cohorts totaling 789 patients. Compute the signature score per patient by aggregating expression of the signature genes (here the NK-2 signature derived from scDist analysis of FCER1G+ NK cells) using a Spearman-correlation-based signature score per ref. 39. Within each cohort, dichotomize patients at the median score into high- and low-signature groups, fit Kaplan-Meier curves per stratum, and compute log-rank p-values comparing the groups. Render KM curves annotated with log-rank p-values using ggplot2/ggpubr, and separately compare signature scores between radiologic responders (partial/complete) and non-responders. Write per-patient scores, strata assignments, and log-rank p-values to an output CSV (nk2_signature_survival_results.csv) and combined figures to nk2_km_response_plots.pdf.

## Related tools

- **scDist** (Source of the NK-2 signature genes derived from FCER1G+ NK cells, whose bulk RNA-seq score is survival-tested) — https://github.com/phillipnicol/scDist
- **ggplot2** (Renders the Kaplan-Meier survival curves for the visualization output)
- **ggpubr** (Annotates KM curves with log-rank p-values and produces the responder-vs-non-responder association plot)

## Examples

```
Rscript nk2_km_response.R --expr bulk_matrix.rds --clin clinical_meta.csv --genes nk2_signature.txt --out nk2_signature_survival_results.csv --plots nk2_km_response_plots.pdf
```

## Evaluation signals

- Total scored patients across the seven cohorts equals 789
- Each cohort's output file exists and contains per-patient scores, strata assignments, and log-rank p-values
- Log-rank p-values qualitatively reproduce the significance pattern reported in the article for high vs low NK-2 groups
- Radiologic responders (partial/complete) show significantly different signature scores than non-responders, matching the reported association
- Strata split is balanced by construction (median dichotomization within each cohort)

## Limitations

- Software/data availability statements and accession numbers for the seven bulk RNA-seq cohorts are not provided in the article, complicating data acquisition
- Median dichotomization is a coarse two-group test; it does not model signature score as a continuous or covariate-adjusted predictor
- Validation is qualitative reproduction of significance patterns, not an exact numeric match to the published p-values

## Evidence

- [intro] Bulk validation of NK-2 signature via KM/log-rank: "jointly inferring information from five single-cell immunotherapy cohorts, revealing significant differences in a subpopulation of NK cells between immunotherapy responders and non-responders"
- [results] Article finding of bulk validation in 789 patients: [evidence span withheld — anchor status: not_found]
- [readme] Install/usage of scDist grounding the toolchain: "From the R console, `devtools::install_github("phillipnicol/scDist")`"
