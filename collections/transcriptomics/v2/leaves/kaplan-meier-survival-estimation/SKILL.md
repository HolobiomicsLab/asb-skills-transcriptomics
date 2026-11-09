---
name: kaplan-meier-survival-estimation
description: Use when a per-sample continuous score (e.g., ssGSEA enrichment of a
  gene signature in TCGA bulk RNA-seq) must be validated as a prognostic indicator
  against patient overall-survival metadata.
when_to_use_negative:
- Cancer-Finder is not recommended for hematologic tumor data, so signature/score
  derivation from such datasets should not feed this survival workflow
- Input lacks per-patient survival time/event metadata or the sample set is too small
  for a two-group median-split comparison
- The goal is classifying malignant cells rather than validating a score's prognostic
  association (use Cancer-Finder/CopyKAT-style annotation instead)
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
tools:
- name: Cancer-Finder
  role: Source of the 10-gene prognostic signature identified from ccRCC spatial transcriptomics
    slides; its genes define the score tested by the survival analysis
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: ssGSEA
  role: Computes per-patient single-sample enrichment scores for the 10-gene signature
    from TCGA KIRC bulk RNA-seq, producing the continuous variable split into high/low
    groups
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:98162751c0c78e98feb1dcccd79f70cd252875304439fd5c1fc2617ab8e7b826
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - ssGSEA
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# kaplan-meier-survival-estimation

## Summary

Kaplan-Meier survival analysis of a continuous molecular score (here the 10-gene ccRCC signature ssGSEA score) split at the median into high/low groups, tested with a two-sided log-rank test, to establish prognostic significance.

## When to use

Use when a per-sample continuous score (e.g., ssGSEA enrichment of a gene signature in TCGA bulk RNA-seq) must be validated as a prognostic indicator against patient overall-survival metadata.

## When NOT to use

- Cancer-Finder is not recommended for hematologic tumor data, so signature/score derivation from such datasets should not feed this survival workflow
- Input lacks per-patient survival time/event metadata or the sample set is too small for a two-group median-split comparison
- The goal is classifying malignant cells rather than validating a score's prognostic association (use Cancer-Finder/CopyKAT-style annotation instead)

## Inputs

- TCGA KIRC bulk RNA-seq expression matrix for patients with overall-survival metadata (n=530)
- Per-patient overall-survival time and event/censoring status table
- Gene set: 10-gene signature (NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19)

## Outputs

- Per-patient signature ssGSEA scores, median-split group assignments, and log-rank statistics saved to km_survival_results.csv
- Kaplan-Meier survival curve rendered as km_curve.png (reproducing Fig. 5f)
- Two-sided log-rank p-value (target: p = 0.0012)

## How to apply

Retrieve TCGA ccRCC (KIRC) bulk RNA-seq expression plus overall-survival metadata and retain the 530 patients having both. Compute per-patient ssGSEA enrichment scores for the 10-gene signature (NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19). Split the cohort into high- and low-signature groups at the median score (median split avoids distributional assumptions about the score). Fit Kaplan-Meier curves per group and test curve separation with a two-sided log-rank test (n=530). Reproduction is judged by recovering the reported p = 0.0012 (< 0.01), which outperformed the Hallmark EMT pathway and ccRCC EMT program comparisons.

## Related tools

- **Cancer-Finder** (Source of the 10-gene prognostic signature identified from ccRCC spatial transcriptomics slides; its genes define the score tested by the survival analysis) — https://github.com/Patchouli-M/SequencingCancerFinder
- **ssGSEA** (Computes per-patient single-sample enrichment scores for the 10-gene signature from TCGA KIRC bulk RNA-seq, producing the continuous variable split into high/low groups)

## Examples

```
ssGSEA_scores <- ssgseaParam(expr_kirc, list(sig10=c('NDRG1','TAGLN','MALAT1','IGKC','IGHA1','IGHG4','IGLC2','IGHG3','SOD2','KRT19'))); survdiff(Surv(time,status) ~ group, data=kirc)
```

## Evaluation signals

- Two-sided log-rank p-value reproduces the reported p = 0.0012 (must be < 0.01) at n = 530
- Cohort size after joining RNA-seq and survival metadata equals 530 KIRC patients
- km_survival_results.csv contains per-patient ssGSEA scores, median-split group labels, and log-rank statistics with no missing survival annotations
- Kaplan-Meier plot (km_curve.png) visually matches Fig. 5f of the article
- Signature performance exceeds that of the Hallmark EMT pathway and ccRCC EMT program comparators

## Limitations

- Result depends on the median-split cutoff choice; alternate thresholds may alter the log-rank p-value
- The 10-gene signature was derived from only 5 ccRCC spatial transcriptomic slides, so its generalization beyond TCGA KIRC is untested here
- ssGSEA scores in bulk RNA-seq reflect tumor-microenvironment composition and deconvolution assumptions, not pure tumor-cell expression

## Evidence

- [abstract] identifies a gene signature consisting of 10 genes that are significantly co-localized and enriched at the tumor-normal interface and have a strong correlation with the prognosis of clear cell renal: "identifies a gene signature consisting of 10 genes that are significantly co-localized and enriched at the tumor-normal interface and have a strong correlation with the prognosis of clear cell renal"
- [intro] we successfully identify a gene signature consisting of ten genes that tends to be enriched at the interface between tumor and normal tissue, may be associated with the formation of an invasive tumor: "we successfully identify a gene signature consisting of ten genes that tends to be enriched at the interface between tumor and normal tissue, may be associated with the formation of an invasive tumor"
- [results] Survival analysis of the Cancer-Finder gene signature ssGSEA score on 530 TCGA ccRCC samples, using a two-sided log-rank test, yielded a significant prognostic association with p = 0.0012 < 0.01: [evidence span withheld — anchor status: not_found]
