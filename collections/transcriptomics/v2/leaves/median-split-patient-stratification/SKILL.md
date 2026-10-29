---
name: median-split-patient-stratification
description: Use when bulk RNA-seq expression matrices with clinical metadata (survival
  time/status, radiologic response) are available and a single-cell-derived gene signature
  (e.g., NK-2 from scDist) must be validated for association with outcome.
when_to_use_negative:
- When the aim is to detect which cell types are perturbed between conditions from
  single-cell data — that is scDist's differential state analysis, not the downstream
  bulk-cohort stratification
- When patient-level survival or response metadata are unavailable, since median-split
  strata cannot be linked to outcomes
- When the cohort is too small for a meaningful median split — the article's validation
  relies on 789 patients across 7 cohorts
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scDist
  role: Upstream mixed-effects method whose FCER1G+ NK cell perturbation defines the
    NK-2 signature genes being validated in bulk cohorts
  repo: github.com/phillipnicol/scDist
- name: ggplot2
  role: Renders the Kaplan-Meier survival curves annotated with log-rank p-values
- name: ggpubr
  role: Produces the responder-vs-non-responder signature score association plot
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:6f991c9517044459b25850f2001fb4a06f2398d2ace1004cbe535b37b1f7db7e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ggplot2
  - ggpubr
  - scDist
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Median-split patient stratification for signature-based survival validation

## Summary

Dichotomizes bulk-RNA-seq patients at the cohort-median of a single-cell-derived signature score (here the NK-2 signature from FCER1G+ NK cells found by scDist) and tests high- vs low-score strata with Kaplan-Meier/log-rank survival analysis and responder-vs-non-responder contrasts, reproducing the paper's validation across 7 cohorts and 789 immunotherapy patients.

## When to use

Use when bulk RNA-seq expression matrices with clinical metadata (survival time/status, radiologic response) are available and a single-cell-derived gene signature (e.g., NK-2 from scDist) must be validated for association with outcome.

## When NOT to use

- When the aim is to detect which cell types are perturbed between conditions from single-cell data — that is scDist's differential state analysis, not the downstream bulk-cohort stratification
- When patient-level survival or response metadata are unavailable, since median-split strata cannot be linked to outcomes
- When the cohort is too small for a meaningful median split — the article's validation relies on 789 patients across 7 cohorts

## Inputs

- Bulk RNA-seq expression matrices for seven cancer cohorts (789 patients total)
- Matching clinical metadata: overall survival time/status and radiologic response (partial/complete vs non-responder)
- NK-2 signature gene list derived from scDist analysis of FCER1G+ NK cells in single-cell immunotherapy cohorts

## Outputs

- Per-patient NK-2 signature scores, high/low strata assignments, and per-cohort log-rank p-values (nk2_signature_survival_results.csv)
- Kaplan-Meier curves annotated with log-rank p-values and responder-vs-non-responder association plots (nk2_km_response_plots.pdf)

## How to apply

Compute the per-patient signature score by aggregating expression of the NK-2 signature genes derived from scDist analysis of FCER1G+ NK cells, using a Spearman-correlation-based signature score. Within each cohort, dichotomize patients at the median score into high- and low-NK2 groups, then fit Kaplan-Meier survival curves per stratum and compute log-rank p-values comparing the two groups. Separately, compare signature scores between radiologic responders (partial/complete) and non-responders. Render Kaplan-Meier curves annotated with log-rank p-values using ggplot2/ggpubr, and write per-patient scores, strata assignments, and log-rank p-values to nk2_signature_survival_results.csv with combined figures in nk2_km_response_plots.pdf. The rationale is that transcriptomic perturbations detected by scDist in single-cell immunotherapy cohorts should reproduce as outcome-associated contrasts in independent bulk transcriptomes (789 patients) if the finding is robust.

## Related tools

- **scDist** (Upstream mixed-effects method whose FCER1G+ NK cell perturbation defines the NK-2 signature genes being validated in bulk cohorts) — github.com/phillipnicol/scDist
- **ggplot2** (Renders the Kaplan-Meier survival curves annotated with log-rank p-values)
- **ggpubr** (Produces the responder-vs-non-responder signature score association plot)

## Evaluation signals

- Each cohort's output file exists and the combined scored patients total 789
- Per-patient rows include signature score, strata assignment, and log-rank p-value columns (nk2_signature_survival_results.csv schema check)
- Log-rank p-values and responder-vs-non-responder contrasts qualitatively reproduce the significance pattern reported in the article
- Median split yields approximately balanced high/low-NK2 groups within each cohort
- Figures in nk2_km_response_plots.pdf show KM curves per stratum annotated with log-rank p-values

## Limitations

- Median-split dichotomization discards within-group variation in signature score; significance depends on cohort size and stratum balance
- The upstream scDist result is sensitive to over-clustering (median distance decays toward 0 as cluster number increases) and to the number of principal components used (fixed heuristic of 20 or 50), which can change the signature genes
- Naive approaches that ignore individual-to-individual and cohort-to-cohort variability lead to false positives, so only scDist-derived signatures should be propagated to the bulk validation

## Evidence

- [intro] NK-2 signature validation rationale: "jointly inferring information from five single-cell immunotherapy cohorts, revealing significant differences in a subpopulation of NK cells between immunotherapy responders and non-responders"
- [other] Median-split workflow step: [evidence span withheld — anchor status: not_found]
- [other] Survival testing step: [evidence span withheld — anchor status: not_found]
- [readme] Upstream variability correction by scDist: "When there are multiple replicates (i.e., patients), `scDist` uses linear-mixed effects models to correct for sample-to-sample variability."
