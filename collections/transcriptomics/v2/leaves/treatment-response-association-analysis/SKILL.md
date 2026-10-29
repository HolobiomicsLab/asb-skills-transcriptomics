---
name: treatment-response-association-analysis
description: Use when a cell-type perturbation signature discovered in single-cell
  immunotherapy cohorts (e.g., scDist FCER1G+ NK cells) must be tested against survival
  and radiologic response in independent bulk RNA-seq cohorts with clinical metadata.
when_to_use_negative:
- The goal is detecting perturbed cell types in single-cell data itself — use scDist
  differential-state analysis instead of bulk signature validation
- No matched survival or radiologic response metadata exists for the bulk cohorts,
  so neither the KM/log-rank nor responder contrast can be computed
- Differential abundance (cell-proportion differences) is the question of interest
  — tools like DA-seq, Milo, or Meld address that, not transcriptomic signature-score
  association testing
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scDist
  role: Derives the cell-type perturbation signature (FCER1G+ NK-cell / NK-2 genes)
    that is scored in bulk transcriptomes
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
  source_content_hash: sha256:be1b76b3d06734a756f00a2e54a653bebdb21b3c1af3b68d15ab2f52d8cf8504
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ggplot2
  - ggpubr
  - scDist
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# treatment-response-association-analysis

## Summary

Validates single-cell-derived treatment-response signatures in independent bulk transcriptomes: compute a per-patient signature score (e.g., the NK-2/FCER1G+ NK-cell signature from scDist), dichotomize at the cohort median, and test association with overall survival via Kaplan-Meier/log-rank and with radiologic response via responder-vs-non-responder contrasts.

## When to use

Use when a cell-type perturbation signature discovered in single-cell immunotherapy cohorts (e.g., scDist FCER1G+ NK cells) must be tested against survival and radiologic response in independent bulk RNA-seq cohorts with clinical metadata.

## When NOT to use

- The goal is detecting perturbed cell types in single-cell data itself — use scDist differential-state analysis instead of bulk signature validation
- No matched survival or radiologic response metadata exists for the bulk cohorts, so neither the KM/log-rank nor responder contrast can be computed
- Differential abundance (cell-proportion differences) is the question of interest — tools like DA-seq, Milo, or Meld address that, not transcriptomic signature-score association testing

## Inputs

- Bulk RNA-seq expression matrices (one per cancer cohort)
- Clinical metadata per patient: overall survival time/status and radiologic response (partial/complete vs progressive/stable disease)
- Gene signature list derived from the single-cell scDist analysis (e.g., NK-2 signature genes from FCER1G+ NK cells)

## Outputs

- Per-patient signature scores and high/low median-split strata assignments (nk2_signature_survival_results.csv)
- Log-rank p-values per cohort
- Kaplan-Meier curves annotated with log-rank p-values and responder-vs-non-responder association plots (nk2_km_response_plots.pdf)

## How to apply

Load bulk RNA-seq expression matrices plus clinical metadata (overall survival time/status and radiologic response) for each cohort — the reference analysis spans 7 cohorts and 789 patients. Compute the signature score per patient by aggregating expression of the signature genes derived from the scDist analysis (Spearman-correlation-based signature score). Dichotomize patients within each cohort at the median score into high- and low-signature strata, then fit Kaplan-Meier curves per stratum and compute log-rank p-values comparing the strata. Separately, compare signature scores between radiologic responders (partial/complete) and non-responders. Render annotated Kaplan-Meier curves and response-association plots with ggplot2/ggpubr, and write per-patient scores, strata assignments, and p-values to tabular output (e.g., nk2_signature_survival_results.csv) with combined figures to a PDF (e.g., nk2_km_response_plots.pdf).

## Related tools

- **scDist** (Derives the cell-type perturbation signature (FCER1G+ NK-cell / NK-2 genes) that is scored in bulk transcriptomes) — github.com/phillipnicol/scDist
- **ggplot2** (Renders the Kaplan-Meier survival curves annotated with log-rank p-values)
- **ggpubr** (Produces the responder-vs-non-responder signature score association plot)

## Examples

```
devtools::install_github("phillipnicol/scDist"); out <- scDist(sim$Y, sim$meta.data, fixed.effects = "response", random.effects = "patient", clusters = "clusters")
```

## Evaluation signals

- Each cohort's output file exists and the union of cohorts contains exactly 789 scored patients
- Patients within each cohort are split into two strata of approximately equal size at the median signature score
- Per-cohort log-rank p-values and the responder-vs-non-responder contrast qualitatively reproduce the significance pattern reported in the article
- Kaplan-Meier figures carry the cohort label, both strata, and the computed log-rank p-value annotation

## Limitations

- Software/data availability statements and accession numbers for the four immunotherapy scRNA cohorts and seven bulk RNA-seq cohorts are not provided in the article
- scDist results are sensitive to the number of principal components used, and no adaptive procedure for selecting K exists (fixed heuristic of 20 or 50), which affects the signature the bulk validation depends on
- Significant clustering/annotation errors could cause unavoidable bias, and a cluster-free extension of scDist does not yet exist

## Evidence

- [intro] NK-cell subpopulation validated in bulk: "jointly inferring information from five single-cell immunotherapy cohorts, revealing significant differences in a subpopulation of NK cells between immunotherapy responders and non-responders"
- [readme] scDist invocation deriving the signature: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response", random.effects="patient", clusters="clusters")"
- [methods] interpreting per-cell-type results: "The results data frame gives a summary of the estimated distance and uncertainty for each cell type"
