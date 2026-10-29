---
name: geo-accession-data-retrieval
description: 'Use when a GEO-deposited bulk RNA-seq cohort (e.g., GSE23649: 8 DBD
  EAD vs 8 DBD non-EAD liver transplant samples) must be retrieved in R to validate
  single-cell signatures.'
when_to_use_negative:
- The count matrix and metadata are already available locally (e.g., a processed matrix
  or Seurat object) — no GEO retrieval step is needed.
- The analysis target is single-cell cell-cell communication inference (LR-pair enrichment
  via scCrossTalk's find_lrpairs/rev_gene workflow), not bulk-cohort retrieval and
  signature validation.
- The cohort has no GEO accession or lacks the paired pre-cold-perfusion / 2h post-portal-reperfusion
  biopsy design matching the transplant time course.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Environment used to retrieve the GSE23649 count matrix and sample metadata
    from GEO, normalize samples to a median of 1000 with log transformation, and run
    Welch's t-test on niche scores
- name: ggplot2
  role: Renders the EAD vs non-EAD comparison boxplot of per-sample niche signature
    scores written to niche_ead_validation.csv
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:a8e6901dcdc7bb199ea3f99ee796da97ac68d97f18cb3fe29c76cd5f599c215f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/multitalk/scCrossTalk
  tools:
  - R
  - ggplot2
  - scCrossTalk
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# geo-accession-data-retrieval

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Retrieves bulk RNA-seq count matrices and sample metadata for a GEO accession (here GSE23649: 8 DBD EAD vs 8 DBD non-EAD liver transplant recipients) using R. This enables validation of single-cell-derived signatures, such as the MAIT / GZMB+ GZMK+ NK / S100A12+ neutrophil EAD immune niche, in an independent bulk cohort.

## When to use

Use when a GEO-deposited bulk RNA-seq cohort (e.g., GSE23649: 8 DBD EAD vs 8 DBD non-EAD liver transplant samples) must be retrieved in R to validate single-cell signatures.

## When NOT to use

- The count matrix and metadata are already available locally (e.g., a processed matrix or Seurat object) — no GEO retrieval step is needed.
- The analysis target is single-cell cell-cell communication inference (LR-pair enrichment via scCrossTalk's find_lrpairs/rev_gene workflow), not bulk-cohort retrieval and signature validation.
- The cohort has no GEO accession or lacks the paired pre-cold-perfusion / 2h post-portal-reperfusion biopsy design matching the transplant time course.

## Inputs

- GEO series accession identifier (GSE23649) for the bulk RNA-seq liver allograft cohort
- Marker-gene lists for the niche cell populations (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) defined in the single-cell landscape

## Outputs

- Raw bulk RNA-seq count matrix for the 16 DBD samples (8 EAD, 8 non-EAD) retrieved from GEO
- Sample metadata with EAD status and perfusion time point (Sample A pre-cold-perfusion, Sample B 2h post-portal-reperfusion)
- niche_ead_validation.csv containing per-sample niche signature scores plus the Welch's t-test statistic and p-value
- EAD vs non-EAD niche-score comparison boxplot rendered with ggplot2

## How to apply

Retrieve the GSE23649 bulk RNA-seq count matrix and sample metadata from GEO in R, confirming the cohort resolves to 8 DBD EAD vs 8 DBD non-EAD recipients, each with two biopsies (Sample A pre-cold-perfusion, Sample B 2h post-portal-reperfusion). Normalize the count matrix by scaling each sample to a median value of 1000 followed by a log transformation so per-sample scores are comparable. Compute a per-sample immune niche signature score from the marker genes of the niche cell populations defined in the single-cell landscape (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils). Compare niche signature scores between EAD and non-EAD groups with Welch's t-test, recording the test statistic and p-value. Write the 16-sample score table and group-comparison statistics to niche_ead_validation.csv and render an EAD vs non-EAD comparison boxplot with ggplot2. Validation succeeds when the table is complete and Welch's test shows a significant difference with EAD samples scoring higher, matching the reported direction.

## Related tools

- **R** (Environment used to retrieve the GSE23649 count matrix and sample metadata from GEO, normalize samples to a median of 1000 with log transformation, and run Welch's t-test on niche scores)
- **ggplot2** (Renders the EAD vs non-EAD comparison boxplot of per-sample niche signature scores written to niche_ead_validation.csv)

## Examples

```
write.csv(niche_ead_validation, "niche_ead_validation.csv"); ggplot(niche_ead_validation, aes(group, niche_score)) + geom_boxplot()
```

## Evaluation signals

- The retrieved sample table resolves to exactly 16 DBD samples: 8 EAD vs 8 non-EAD, each contributing a Sample A (pre-cold-perfusion) and a Sample B (2h post-portal-reperfusion) biopsy.
- Post-normalization check: every sample's counts have been scaled to a median value of 1000 and log-transformed before niche scoring.
- niche_ead_validation.csv contains one row per sample for all 16 samples with the niche score plus the Welch's t-test statistic and p-value.
- Direction-of-effect check: Welch's test shows a significant niche-score difference with EAD samples scoring higher, the same direction as the single-cell report.

## Limitations

- The GSE23649 validation cohort is limited to 8 DBD EAD vs 8 DBD non-EAD recipients with paired pre-cold-perfusion and 2h post-portal-reperfusion biopsies; other graft types or sampling times are out of scope.
- The niche readout is a bulk per-sample marker-gene score and cannot resolve the individual MAIT, GZMB+ GZMK+ NK, and S100A12+ neutrophil populations captured in the 75,231-cell single-cell landscape.

## Evidence

- [other] Retrieve the GSE23649 bulk RNA-seq count matrix and sample metadata (8 DBD EAD vs 8 DBD non-EAD; Sample A pre-cold-perfusion, Sample B 2h post-portal-reperfusion) from GEO using R.: [evidence span withheld — anchor status: not_found]
- [other] Normalize the count matrix by scaling each sample to a median value of 1000 followed by a log transformation.: [evidence span withheld — anchor status: not_found]
- [other] Compute a per-sample immune niche signature score using the marker genes of the niche cell populations (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils): [evidence span withheld — anchor status: not_found]
