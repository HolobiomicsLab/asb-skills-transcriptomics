---
name: log-rank-significance-testing
description: Use when a patient cohort has overall-survival metadata plus a continuous
  risk/enrichment score (e.g., ssGSEA signature score) and you must test high-vs-low
  group separation by survival.
when_to_use_negative:
- The cohort lacks survival time/event metadata, so no Kaplan-Meier or log-rank comparison
  is possible
- You need to identify which cells are malignant (use Cancer-Finder or CNV-based tools
  instead) — this skill only tests prognostic association of an already-computed score
- The gene set is not fixed a priori; deriving a new signature requires the Cancer-Finder
  saliency/interpretability workflow first
tools:
- name: Cancer-Finder
  role: Source of the 10-gene prognostic signature whose ssGSEA score is tested by
    the log-rank analysis
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: ssGSEA
  role: Computes per-patient single-sample enrichment scores for the 10-gene signature
    from TCGA bulk RNA-seq before median-split grouping
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:d134f21b8cb5f135172176f2686bb53ce60bbe2ee7a67b56aa23aab990f028e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - ssGSEA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# log-rank-significance-testing

## Summary

Two-sided log-rank testing of Kaplan-Meier survival curves to assess whether a gene-signature score (here the Cancer-Finder 10-gene ccRCC signature) stratifies patients into high- and low-risk groups with significantly different overall survival.

## When to use

Use when a patient cohort has overall-survival metadata plus a continuous risk/enrichment score (e.g., ssGSEA signature score) and you must test high-vs-low group separation by survival.

## When NOT to use

- The cohort lacks survival time/event metadata, so no Kaplan-Meier or log-rank comparison is possible
- You need to identify which cells are malignant (use Cancer-Finder or CNV-based tools instead) — this skill only tests prognostic association of an already-computed score
- The gene set is not fixed a priori; deriving a new signature requires the Cancer-Finder saliency/interpretability workflow first

## Inputs

- TCGA ccRCC (KIRC) bulk RNA-seq expression matrix with matched overall-survival metadata for 530 patients
- Per-patient ssGSEA enrichment scores for the 10-gene Cancer-Finder signature (NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19)

## Outputs

- km_survival_results.csv with per-patient signature scores, median-split high/low group assignments, and log-rank statistics
- km_curve.png Kaplan-Meier survival plot reproducing Fig. 5f
- Two-sided log-rank p-value (expected: 0.0012, significant at p < 0.01)

## How to apply

Start from per-patient signature scores computed on the 530 TCGA ccRCC (KIRC) bulk RNA-seq samples that have both expression and overall-survival information; scores are derived via ssGSEA over the 10-gene signature (NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19). Split the cohort into high- and low-signature groups at the median score value (median split avoids arbitrary cutoff tuning). Fit Kaplan-Meier curves per group and apply a two-sided log-rank test to compare survival distributions. Declare a significant prognostic association when p < 0.01; the reference result is p = 0.0012 at n = 530, where the Cancer-Finder signature outperformed the Hallmark EMT pathway and the ccRCC EMT program. Persist per-patient scores, median-split assignments, and log-rank statistics (km_survival_results.csv) and render the Kaplan-Meier curve (km_curve.png, reproducing Fig. 5f).

## Related tools

- **Cancer-Finder** (Source of the 10-gene prognostic signature whose ssGSEA score is tested by the log-rank analysis) — https://github.com/Patchouli-M/SequencingCancerFinder
- **ssGSEA** (Computes per-patient single-sample enrichment scores for the 10-gene signature from TCGA bulk RNA-seq before median-split grouping)

## Examples

```
survdiff(Surv(OS_time, OS_status) ~ signature_group, data = km_survival_results)
```

## Evaluation signals

- Reproduce the two-sided log-rank p = 0.0012 (< 0.01) at n = 530 for the Cancer-Finder signature
- The median split yields two groups of approximately equal size (~265 per arm) from the 530-patient cohort
- km_survival_results.csv contains a score, group label, and survival fields for every patient with both RNA-seq and survival data
- The rendered km_curve.png visually separates the high- and low-signature Kaplan-Meier curves, matching Fig. 5f
- The signature's log-rank p-value is lower (better separation) than that of the Hallmark EMT pathway and ccRCC EMT program comparators

## Limitations

- The 10-gene signature was identified from only 5 ccRCC spatial transcriptomic slides, so generalization beyond ccRCC should be verified per cohort
- Median-split dichotomization discards score magnitude; results can be sensitive to the choice of cutoff

## Evidence

- [abstract] abstract: "identifies a gene signature consisting of 10 genes that are significantly co-localized and enriched at the tumor-normal interface and have a strong correlation with the prognosis of clear cell renal"
- [other] other: [evidence span withheld — anchor status: not_found]
