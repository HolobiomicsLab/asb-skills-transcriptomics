---
name: prognostic-signature-benchmarking
description: Use when a candidate gene signature identified from single-cell or spatial
  transcriptomics must be validated as a prognostic indicator in a bulk RNA-seq cohort
  with survival metadata (e.g., TCGA KIRC overall survival).
when_to_use_negative:
- Input is single-cell or spatial data without matched bulk cohort survival metadata
  — the skill requires bulk RNA-seq plus overall-survival information per patient
- The cohort lacks enough patients in both median-split groups to support a meaningful
  log-rank comparison
- Evaluating hematologic tumor signatures via the upstream Cancer-Finder identification
  step — the article states it is not recommended for hematologic tumor data (survival
  validation itself is unaffected, but the signature source is)
tools:
- name: Cancer-Finder
  role: Source of the 10-gene prognostic signature (identified via saliency-map interpretability
    module on ccRCC spatial transcriptomic slides) that is validated by survival analysis
    in this skill
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: ssGSEA
  role: Single-sample Gene Set Enrichment Analysis algorithm used to compute per-patient
    enrichment scores for the 10-gene signature from bulk RNA-seq
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:f181c0218140091a00014f849954a9be00a0aff749c60f41f2c28c5317cae340
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
yes_rate: 0.9090909090909091
n_claims: 11
---

# prognostic-signature-benchmarking

## Summary

Evaluates whether a gene signature (here the 10-gene ccRCC interface signature: NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19) predicts patient survival by scoring bulk RNA-seq cohorts with ssGSEA, median-splitting patients, and comparing Kaplan-Meier curves with a two-sided log-rank test, optionally benchmarking against competing signatures such as Hallmark EMT.

## When to use

Use when a candidate gene signature identified from single-cell or spatial transcriptomics must be validated as a prognostic indicator in a bulk RNA-seq cohort with survival metadata (e.g., TCGA KIRC overall survival).

## When NOT to use

- Input is single-cell or spatial data without matched bulk cohort survival metadata — the skill requires bulk RNA-seq plus overall-survival information per patient
- The cohort lacks enough patients in both median-split groups to support a meaningful log-rank comparison
- Evaluating hematologic tumor signatures via the upstream Cancer-Finder identification step — the article states it is not recommended for hematologic tumor data (survival validation itself is unaffected, but the signature source is)

## Inputs

- TCGA KIRC bulk RNA-seq expression matrix (counts/TPM) for 530 patients
- Overall survival metadata (time, event status) matched to the same patients
- Candidate gene signature list (10 genes: NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19)
- Comparator gene sets for benchmarking (Hallmark EMT pathway, ccRCC EMT program)

## Outputs

- Per-patient ssGSEA signature scores (data frame)
- Median-split high/low group assignments per patient
- Two-sided log-rank test statistics and p-value (km_survival_results.csv)
- Kaplan-Meier survival curve image (km_curve.png, reproducing Fig. 5f)

## How to apply

Retrieve TCGA ccRCC (KIRC) bulk RNA-seq expression together with overall-survival metadata and retain only patients having both (n=530). Compute per-patient ssGSEA enrichment scores for the signature genes, then split the cohort into high- and low-signature groups at the median score. Fit Kaplan-Meier curves per group and test curve separation with a two-sided log-rank test; significance threshold used in the paper is p < 0.01. Benchmark the signature against alternative gene sets (e.g., Hallmark EMT pathway and the ccRCC EMT program) with the identical ssGSEA/median-split/log-rank procedure to show superior prognostic separation. Persist per-patient scores, group assignments, and log-rank statistics, and render the Kaplan-Meier curve reproducing Fig. 5f.

## Related tools

- **Cancer-Finder** (Source of the 10-gene prognostic signature (identified via saliency-map interpretability module on ccRCC spatial transcriptomic slides) that is validated by survival analysis in this skill) — https://github.com/Patchouli-M/SequencingCancerFinder
- **ssGSEA** (Single-sample Gene Set Enrichment Analysis algorithm used to compute per-patient enrichment scores for the 10-gene signature from bulk RNA-seq)

## Examples

```
from lifelines.statistics import logrank_test; p = logrank_test(T_high, T_low, E_high, E_low).p_value
```

## Evaluation signals

- Two-sided log-rank p-value on the 530-patient TCGA ccRCC cohort reproduces the reported p = 0.0012 (< 0.01) for the Cancer-Finder signature, matching Fig. 5f
- Patient count retained in the analysis equals 530 (patients with both RNA-seq and survival metadata)
- km_survival_results.csv contains per-patient ssGSEA scores, median-split group labels, and log-rank statistics with no missing survival records
- The signature's log-rank p-value is smaller than that of the Hallmark EMT pathway and ccRCC EMT program computed with the identical procedure
- Kaplan-Meier curve splits exactly at the median signature score, yielding two groups of 265 patients each

## Limitations

- Cancer-Finder is not recommended for hematologic tumor data, so signatures derived from such tumors via the upstream tool may not be trustworthy
- Cancer-Finder overlooks the spatial relationships among spots in ST data, which could bias interface-derived signatures
- The validation is confined to cohorts with overall-survival metadata; signatures cannot be prognostically benchmarked on datasets lacking survival information

## Evidence

- [abstract] 10-gene signature correlated with prognosis: "identifies a gene signature consisting of 10 genes that are significantly co-localized and enriched at the tumor-normal interface and have a strong correlation with the prognosis of clear cell renal"
- [intro] signature usable as prognostic indicator: "may be associated with the formation of an invasive tumor"
- [readme] saliency map identifies prognosis-related genes: "Cancer-Finder integrated an interpretability module, which can utilizes a saliency map to identified important genes related to prognosis and tumor microenvironment."
- [other] task workflow specifying ssGSEA and log-rank on TCGA ccRCC: [evidence span withheld — anchor status: not_found]
