---
name: bulk-rnaseq-signature-scoring
description: Use when single-cell-derived cell-type signatures (e.g. from scDist differential-state
  analysis) must be validated in bulk transcriptome cohorts with matched survival
  and radiologic-response metadata.
when_to_use_negative:
- The validation target is single-cell data and the goal is detecting perturbed cell
  types — scDist's differential-state model on normalized single-cell counts is the
  appropriate tool, not bulk signature scoring
- Expression data are raw counts without normalization or a defined gene signature
  — the score is Spearman-correlation-based on signature gene expression and requires
  processed expression matrices
- Cohorts lack matched survival or response metadata, so neither KM/log-rank testing
  nor responder-vs-non-responder contrasts can be performed
edam_operation: http://edamontology.org/operation_0313
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: scDist
  role: Source of the NK-2 signature genes via differential-state analysis of FCER1G+
    NK cells between immunotherapy responders and non-responders
  repo: github.com/phillipnicol/scDist
- name: ggplot2
  role: Renders the Kaplan-Meier survival curves annotated with log-rank p-values
- name: ggpubr
  role: Produces the responder vs non-responder signature-score association plot
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:57784ecd0ed7410278397294b1daf446e72522361ad42692f1dc1744458bee0a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ggplot2
  - ggpubr
  - scDist
  edam_operation: http://edamontology.org/operation_0313
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# bulk-rnaseq-signature-scoring

## Summary

Score independent bulk RNA-seq cohorts against a cell-type signature (here the NK-2 signature derived from FCER1G+ NK cells via scDist) and test whether the signature separates outcomes, e.g. immunotherapy responders vs non-responders and overall survival via Kaplan-Meier/log-rank.

## When to use

Use when single-cell-derived cell-type signatures (e.g. from scDist differential-state analysis) must be validated in bulk transcriptome cohorts with matched survival and radiologic-response metadata.

## When NOT to use

- The validation target is single-cell data and the goal is detecting perturbed cell types — scDist's differential-state model on normalized single-cell counts is the appropriate tool, not bulk signature scoring
- Expression data are raw counts without normalization or a defined gene signature — the score is Spearman-correlation-based on signature gene expression and requires processed expression matrices
- Cohorts lack matched survival or response metadata, so neither KM/log-rank testing nor responder-vs-non-responder contrasts can be performed

## Inputs

- Bulk RNA-seq expression matrices for seven cancer cohorts
- Matching clinical metadata: overall survival time/status and radiologic response per patient
- NK-2 signature gene list derived from the scDist analysis of FCER1G+ NK cells

## Outputs

- Per-patient NK-2 signature scores and high/low strata assignments per cohort
- Per-cohort log-rank p-values comparing high vs low signature groups
- nk2_signature_survival_results.csv (scores, strata, p-values)
- nk2_km_response_plots.pdf (Kaplan-Meier curves with log-rank annotations; responder vs non-responder association plot)

## How to apply

Load bulk RNA-seq expression matrices plus clinical metadata (overall survival time/status, radiologic response) for each cohort (seven cohorts, 789 patients). Compute the NK-2 signature score per patient by aggregating expression of the signature genes derived from the scDist analysis of FCER1G+ NK cells, using a Spearman-correlation-based signature score per ref. 39. Dichotomize patients within each cohort at the median signature score into high- and low-NK2 strata, then fit Kaplan-Meier curves per stratum and compute log-rank p-values comparing high vs low groups. Separately compare signature scores between radiologic responders (partial/complete) and non-responders. Render curves annotated with log-rank p-values using ggplot2/ggpubr and export per-patient scores, strata assignments, and p-values.

## Related tools

- **scDist** (Source of the NK-2 signature genes via differential-state analysis of FCER1G+ NK cells between immunotherapy responders and non-responders) — github.com/phillipnicol/scDist
- **ggplot2** (Renders the Kaplan-Meier survival curves annotated with log-rank p-values)
- **ggpubr** (Produces the responder vs non-responder signature-score association plot)

## Examples

```
devtools::install_github("phillipnicol/scDist"); out <- scDist(sim$Y, sim$meta.data, fixed.effects="response", random.effects="patient", clusters="clusters")
```

## Evaluation signals

- Each cohort's output file exists and the combined output contains 789 total scored patients across the seven cohorts
- Log-rank p-values and responder-vs-non-responder contrasts qualitatively reproduce the significance pattern reported in the article
- Every patient falls in exactly one high/low stratum, split at the cohort-specific median signature score
- Kaplan-Meier figures include log-rank p-value annotations for each cohort
- Signature scores are computable for all signature genes present in each cohort's expression matrix with no missing-value failures

## Limitations

- Results are sensitive to analyst choices such as the median-dichotomization cut point and the Spearman-correlation-based scoring scheme; the article's qualitative significance pattern, not exact p-values, is the reproduction target
- The upstream scDist signature depends on normalized single-cell input; the article notes performance can suffer if data are normalized by a sub-optimal approach other than Pearson residuals
- Software/data availability statements and accession numbers for the seven bulk RNA-seq cohorts are not provided in the article, so cohort acquisition must be resolved independently

## Evidence

- [abstract] scDist uncovers transcriptomic perturbations in FCER1G+ NK cells that inform immunotherapy response validation: "scDist uncovers transcriptomic perturbations in dendritic cells, plasmacytoid dendritic cells, and FCER1G+NK cells, that provide new insights into disease mechanisms and treatment responses"
- [intro] Five single-cell immunotherapy cohorts reveal NK subpopulation differences validated in bulk transcriptomes from 789 patients: "jointly inferring information from five single-cell immunotherapy cohorts, revealing significant differences in a subpopulation of NK cells between immunotherapy responders and non-responders"
- [discussion] scDist is the tool that derives the cell-type signature subsequently scored in bulk data: "We present scDist, a statistically rigorous and computationally fast method for detecting cell-type specific differences across multiple groups or conditions."
- [methods] Install scDist from GitHub to reproduce the upstream signature derivation: "From the R console, `devtools::install_github("phillipnicol/scDist")`"
