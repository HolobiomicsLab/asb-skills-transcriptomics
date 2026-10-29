---
name: log-rank-hazard-ratio-estimation
description: Use when two patient subtype calls (e.g., basal-like vs classical from
  signature scoring of 150 TCGA PAAD tumors) must be validated against overall survival
  with KM curves, a log-rank P value, and a hazard ratio.
when_to_use_negative:
- Input is a single-cell clustering result without patient-level survival metadata
  — use silhouette-based subclustering (ComputeSilhouetteScores) instead of survival
  testing
- No clinical follow-up data (OS time/event) are available for the tumor samples,
  making KM/log-rank analysis impossible
- More than two prognostic groups need simultaneous comparison — this skill is defined
  for the two-signature, two-subtype (basal-like vs classical) setting
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: R
  role: Environment in which the subtype assignments, Kaplan-Meier fits, log-rank
    test, and hazard ratio are computed
- name: SCISSORS
  role: Source of the basal-like and classical tumor gene signatures (top-10 fold-change-ranked)
    used to assign the subtypes being survival-tested
  repo: https://github.com/jr-leary7/SCISSORS
- name: Seurat
  role: Upstream single-cell framework from which the SCISSORS signatures are derived
    before de novo application to bulk TCGA PAAD data
  repo: https://github.com/jr-leary7/SCISSORS
- name: ggplot2
  role: Plotting library for rendering the Kaplan-Meier survival curves
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad449
    title: <u>S</u>ub-<u>C</u>luster <u>I</u>dentification through <u>S</u>emi-<u>S</u>upervised
      <u>O</u>ptimization of <u>R</u>are-Cell <u>S</u>ilhouettes (SCISSORS) in single-cell
      RNA-sequencing
  source_content_hash: sha256:29e86d26d00012512e37ea73176500fe42c741edc5102c9d2dfe3604934c5362
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jr-leary7/SCISSORS
  tools:
  - R
  - SCISSORS
  - Seurat
  - dplyr
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# log-rank-hazard-ratio-estimation

## Summary

Survival-analysis validation skill that fits Kaplan-Meier curves for two tumor subtypes (e.g., basal-like vs classical) and evaluates separation with a log-rank test, reporting the hazard ratio. It converts a subtype-calling step (such as SCISSORS-gene signature scoring on TCGA PAAD bulk RNA-seq) into a clinically interpretable survival finding.

## When to use

Use when two patient subtype calls (e.g., basal-like vs classical from signature scoring of 150 TCGA PAAD tumors) must be validated against overall survival with KM curves, a log-rank P value, and a hazard ratio.

## When NOT to use

- Input is a single-cell clustering result without patient-level survival metadata — use silhouette-based subclustering (ComputeSilhouetteScores) instead of survival testing
- No clinical follow-up data (OS time/event) are available for the tumor samples, making KM/log-rank analysis impossible
- More than two prognostic groups need simultaneous comparison — this skill is defined for the two-signature, two-subtype (basal-like vs classical) setting

## Inputs

- Per-tumor subtype calls (basal-like / classical) for 150 TCGA PAAD bulk RNA-seq tumors
- Patient-level overall survival table (time-to-event and censoring status per TCGA PAAD barcode)
- Optional reference schema calls (Moffitt basal-like / classical) for concordance checking

## Outputs

- Kaplan-Meier survival curves stratified by subtype (basal-like vs classical)
- Log-rank test statistic and P value (expected P = .025)
- Hazard ratio with direction of effect (expected HR = 1.723, shorter basal-like survival)
- Subtype-by-schema concordance table (target 139/150 consistent calls)

## How to apply

First assign each tumor to a subtype by scoring it against both gene signatures and taking the higher-scoring one (here, top-10 fold-change-ranked SCISSORS basal-like and classical signatures anchored by their overlap with Moffitt genes), producing per-tumor calls such as 33 basal-like / 106 classical out of 150 TCGA PAAD tumors. Then, with overall-survival time and event status per patient in R, fit Kaplan-Meier survival curves stratified by the two subtype calls and evaluate curve separation with the log-rank test. Report the log-rank P value and the Cox-style hazard ratio comparing the aggressive subtype to the reference (expected in the source analysis: P = .025, hazard ratio = 1.723, shorter survival for basal-like). Decision points: subtype calls must first be checked for concordance with a published schema (target 139-of-150 Moffitt-consistent calls) before survival testing, and significance should be reported alongside the effect size rather than the P value alone.

## Related tools

- **R** (Environment in which the subtype assignments, Kaplan-Meier fits, log-rank test, and hazard ratio are computed)
- **SCISSORS** (Source of the basal-like and classical tumor gene signatures (top-10 fold-change-ranked) used to assign the subtypes being survival-tested) — https://github.com/jr-leary7/SCISSORS
- **Seurat** (Upstream single-cell framework from which the SCISSORS signatures are derived before de novo application to bulk TCGA PAAD data) — https://github.com/jr-leary7/SCISSORS
- **ggplot2** (Plotting library for rendering the Kaplan-Meier survival curves)

## Examples

```
library(survival); survdiff(Surv(OS_time, OS_event) ~ subtype, data = tcga_paad); coxph(Surv(OS_time, OS_event) ~ subtype, data = tcga_paad)
```

## Evaluation signals

- Subtype calls reproduce 139/150 concordance with the Moffitt schema (33 basal-like, 106 classical) before survival testing
- Log-rank test on the KM curves yields P = .025 for basal-like vs classical separation
- Estimated hazard ratio equals ~1.723 with survival worse in the basal-like group (direction must match)
- All 150 TCGA PAAD tumors receive exactly one subtype call with no missing survival annotations
- KM curves are stratified on the SCISSORS-signature-derived calls, not the published Moffitt calls, to demonstrate de novo reproducibility

## Limitations

- The analysis covers only 150 TCGA PAAD tumors, so the log-rank P = .025 and HR = 1.723 rest on a modest cohort size
- Subtype assignments come from signature scoring whose parameters (gene count, fold-change ranking) can shift calls; as the article notes, 'even small differences in analysis and parameter choosing can greatly alter clustering results'
- The expected statistics (P = .025, HR = 1.723) were reported for a specific figure (Fig. 5k) and may not transfer to other cohorts or signature sizes

## Evidence

- [other] Basal-like survival significance: "patients with basal-like tumors subtyped by SCISSORS genes showed significantly shorter survival (P = .025, hazard ratio = 1.723, and log-rank test) (Fig. 5k)"
- [other] Workflow survival step: [evidence span withheld — anchor status: not_found]
- [abstract] Parameter sensitivity caveat: "even small differences in analysis and parameter choosing can greatly alter clustering results and thus impose great influence on which cell types are identified"
- [readme] SCISSORS purpose from README: "This package enables researchers to efficiently, accurately, and reproducibly uncover rare celltypes and other small subgroups using an iterative reclustering optimization"
