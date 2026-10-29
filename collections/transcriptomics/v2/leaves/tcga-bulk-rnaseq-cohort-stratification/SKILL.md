---
name: tcga-bulk-rnaseq-cohort-stratification
description: Use when validating whether a gene signature discovered in single-cell/spatial
  data (e.g. the Cancer-Finder 10-gene ccRCC signature) has prognostic value in an
  independent bulk RNA-seq cohort with survival metadata.
when_to_use_negative:
- Cohort is a hematologic tumor or the signature was derived from hematologic malignancy
  data, where the source method is explicitly not recommended
- Patients lack matched survival metadata, so no Kaplan-Meier/log-rank outcome analysis
  is possible
- The gene signature is not measurable in the bulk RNA-seq panel (signature genes
  absent from the expression matrix)
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_3518
tools:
- name: Cancer-Finder
  role: Source of the 10-gene prognostic signature (NDRG1, TAGLN, MALAT1, IGKC, IGHA1,
    IGHG4, IGLC2, IGHG3, SOD2, KRT19) identified from ccRCC spatial transcriptomic
    slides; its interpretability/saliency module surfaces prognosis-related genes
    that this survival workflow then validates in TCGA bulk RNA-seq.
  repo: https://github.com/Patchouli-M/SequencingCancerFinder
- name: ssGSEA
  role: Computes per-patient single-sample gene set enrichment scores for the signature
    from the TCGA bulk expression matrix, enabling median-split stratification of
    the 530-patient cohort.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-46413-6
    title: Domain generalization enables general cancer cell annotation in single-cell
      and spatial transcriptomics
  source_content_hash: sha256:5c1cfdb36cb3a770e1a6b40fa4577ba00bdb1c47d101f11f454c56d756fba715
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Patchouli-M/SequencingCancerFinder
  tools:
  - Cancer-Finder
  - ssGSEA
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# tcga-bulk-rnaseq-cohort-stratification

## Summary

Stratifies a TCGA bulk RNA-seq patient cohort (e.g. 530 KIRC ccRCC samples with overall-survival metadata) into high- vs low-signature groups by median-splitting per-patient ssGSEA scores of a gene signature, then tests prognostic separation with a two-sided log-rank Kaplan-Meier analysis to validate the signature as a prognostic indicator.

## When to use

Use when validating whether a gene signature discovered in single-cell/spatial data (e.g. the Cancer-Finder 10-gene ccRCC signature) has prognostic value in an independent bulk RNA-seq cohort with survival metadata.

## When NOT to use

- Cohort is a hematologic tumor or the signature was derived from hematologic malignancy data, where the source method is explicitly not recommended
- Patients lack matched survival metadata, so no Kaplan-Meier/log-rank outcome analysis is possible
- The gene signature is not measurable in the bulk RNA-seq panel (signature genes absent from the expression matrix)

## Inputs

- TCGA KIRC bulk RNA-seq expression matrix (counts/TPM per gene per patient)
- TCGA overall-survival metadata (time and event status per patient)
- Gene signature list (10 genes: NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19)

## Outputs

- Per-patient ssGSEA signature scores
- Median-split high/low group assignments per patient
- Log-rank test statistics and p-value
- Kaplan-Meier survival curve image (km_curve.png)
- Results table (km_survival_results.csv)

## How to apply

Retrieve TCGA ccRCC (KIRC) bulk RNA-seq expression data together with overall-survival metadata, retaining the 530 patients that have both RNA-seq data and survival information. Compute per-patient ssGSEA enrichment scores for the 10-gene signature (NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19) using the single-sample Gene Set Enrichment Analysis algorithm. Split the cohort into high- and low-signature groups at the median value of the signature score, fit Kaplan-Meier survival curves for the two groups, and test separation with a two-sided log-rank test (n=530). Save per-patient signature scores, median-split group assignments, and log-rank statistics (e.g. to km_survival_results.csv) and render the Kaplan-Meier curve (km_curve.png), here reproducing Fig. 5f. Rationale: median-split dichotomization plus log-rank testing is the standard, reproducible way to ask whether interface-enriched malignant-cell signature expression correlates with patient outcome, and the signature should outperform generic benchmarks such as the Hallmark EMT pathway and the ccRCC EMT program.

## Related tools

- **Cancer-Finder** (Source of the 10-gene prognostic signature (NDRG1, TAGLN, MALAT1, IGKC, IGHA1, IGHG4, IGLC2, IGHG3, SOD2, KRT19) identified from ccRCC spatial transcriptomic slides; its interpretability/saliency module surfaces prognosis-related genes that this survival workflow then validates in TCGA bulk RNA-seq.) — https://github.com/Patchouli-M/SequencingCancerFinder
- **ssGSEA** (Computes per-patient single-sample gene set enrichment scores for the signature from the TCGA bulk expression matrix, enabling median-split stratification of the 530-patient cohort.)

## Examples

```
ssgsea_scores = ssgsea(expr_matrix, gene_sets={'CF_signature': ['NDRG1','TAGLN','MALAT1','IGKC','IGHA1','IGHG4','IGLC2','IGHG3','SOD2','KRT19']}); groups = ssgsea_scores > ssgsea_scores.median(); logrank = multivariate_logrank_test(...); kmf.fit(times[high], events[high])
```

## Evaluation signals

- Two-sided log-rank p-value reproduces the reported p = 0.0012 (< 0.01) at n = 530 for the Cancer-Finder gene signature, matching Fig. 5f
- Cohort size after filtering equals 530 KIRC patients with both RNA-seq data and survival information
- Median-split yields two groups of approximately equal size (~265 high, ~265 low signature score)
- Signature survival separation outperforms benchmark gene sets (Hallmark EMT pathway and ccRCC EMT program)
- Output artifacts contain all required fields: per-patient ssGSEA scores, group assignments, and log-rank statistics in km_survival_results.csv plus km_curve.png

## Limitations

- A significant median-split log-rank result demonstrates association, not causation, and is sensitive to the median cutoff choice
- ssGSEA scores in bulk RNA-seq reflect aggregate tumor-microenvironment composition, so the signature signal may partly stem from cell-type proportions rather than malignant-cell-intrinsic expression
- Results depend on cohort completeness; patients missing survival metadata or expression data must be excluded, changing n from the full KIRC set

## Evidence

- [abstract] 10-gene signature co-localized and enriched at the tumor-normal interface and correlated with patient prognosis: "identifies a gene signature consisting of 10 genes that are significantly co-localized and enriched at the tumor-normal interface and have a strong correlation with the prognosis of clear cell renal"
- [other] Survival analysis workflow: 530 TCGA ccRCC patients, ssGSEA, median split, log-rank p=0.0012: [evidence span withheld — anchor status: not_found]
- [readme] Cancer-Finder interpretability module identifies prognosis-related genes: "Cancer-Finder integrated an interpretability module, which can utilizes a saliency map to identified important genes related to prognosis and tumor microenvironment."
