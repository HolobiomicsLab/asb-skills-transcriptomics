---
name: immune-niche-signature-scoring
description: Use when an independent bulk RNA-seq (e.g. GSE23649) or scRNA-seq cohort
  (e.g. SRP313633) of liver transplant recipients with EAD vs non-EAD labels must
  be scored for the niche cell populations and tested for association with EAD occurrence.
when_to_use_negative:
- The goal is to infer ligand-receptor-mediated cell-cell communications from scRNA-seq
  data — use the scCrossTalk find_lrpairs/enrichment workflow instead of signature
  scoring.
- The cohort lacks EAD vs non-EAD clinical labels or lacks Sample A (pre-cold-perfusion)
  / Sample B (2h post-portal-reperfusion) stage metadata, so the niche-occurrence
  association cannot be tested.
- Input is already a processed niche score table or feature-level summary rather than
  a count matrix or annotated single-cell object.
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Environment for cohort retrieval, sample normalization to median 1000 plus
    log transformation, niche score computation, and Welch's t-test
  repo: https://github.com/multitalk/scCrossTalk
- name: Seurat
  role: Preprocessing, dimensionality reduction, clustering, and annotation of the
    SRP313633 scRNA-seq cohort to identify MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+
    neutrophils
  repo: https://github.com/multitalk/scCrossTalk
- name: scCrossTalk
  role: Source of the marker definitions and cell-cell communication context for the
    niche cell populations defined in the manuscript
  repo: https://github.com/multitalk/scCrossTalk
- name: ggplot2
  role: Rendering the EAD vs non-EAD niche-score comparison boxplot
  repo: https://github.com/multitalk/scCrossTalk
provenance:
  source_task_ids:
  - task_008
  - task_007
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:53864c28dd1582c07cc1f8367e4372180ceecb1bc0a2a47ed157d28aa3a3915d
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/multitalk/scCrossTalk
  repo_url: https://github.com/multitalk/scCrossTalk
  tools:
  - R
  - Seurat
  - ggplot2
  - scCrossTalk
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# immune-niche-signature-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes a per-sample score reflecting the abundance of the EAD-associated immune niche identified in the single-cell landscape of human transplanted livers (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) and tests whether it discriminates EAD from non-EAD samples in independent validation cohorts.

## When to use

Use when an independent bulk RNA-seq (e.g. GSE23649) or scRNA-seq cohort (e.g. SRP313633) of liver transplant recipients with EAD vs non-EAD labels must be scored for the niche cell populations and tested for association with EAD occurrence.

## When NOT to use

- The goal is to infer ligand-receptor-mediated cell-cell communications from scRNA-seq data — use the scCrossTalk find_lrpairs/enrichment workflow instead of signature scoring.
- The cohort lacks EAD vs non-EAD clinical labels or lacks Sample A (pre-cold-perfusion) / Sample B (2h post-portal-reperfusion) stage metadata, so the niche-occurrence association cannot be tested.
- Input is already a processed niche score table or feature-level summary rather than a count matrix or annotated single-cell object.

## Inputs

- Bulk RNA-seq count matrix with sample metadata (GSE23649: 8 DBD EAD vs 8 DBD non-EAD; Sample A pre-cold-perfusion, Sample B 2h post-portal-reperfusion)
- Raw human liver transplantation scRNA-seq data from SRA accession SRP313633 loaded into Seurat
- Marker gene sets of the niche cell populations (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) from the single-cell landscape

## Outputs

- Per-sample immune niche signature scores with EAD/non-EAD group labels (niche_ead_validation.csv for GSE23649; ead_niche_verification.csv for SRP313633)
- Welch's t-test statistic and p-value for EAD vs non-EAD niche-score comparison
- EAD vs non-EAD comparison boxplot rendered with ggplot2 (bulk cohort)

## How to apply

First define the niche marker-gene set from the single-cell landscape where comparison of 75,231 cells from non-EAD and EAD patients identified the EAD-associated immune niche comprising MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils. For bulk data (GSE23649, 8 DBD EAD vs 8 DBD non-EAD), normalize the count matrix by scaling each sample to a median of 1000 followed by log transformation, then compute a per-sample signature score from the niche markers, comparing Sample A (pre-cold-perfusion) and Sample B (2h post-portal-reperfusion) profiles as appropriate. For scRNA-seq data (SRP313633), run standard Seurat preprocessing (normalization, dimensionality reduction, clustering), annotate the three niche populations with the scCrossTalk manuscript marker definitions, and score each sample by the abundance of these cells. Test niche-score differences between non-EAD and EAD groups with Welch's t-test (one-sided when mirroring the scCrossTalk paper's comparison strategy), recording the test statistic and p-value. The validation is considered reproduced when the per-sample output table is complete and the test is significant with EAD samples scoring higher, i.e. the same direction as reported in the manuscript.

## Related tools

- **R** (Environment for cohort retrieval, sample normalization to median 1000 plus log transformation, niche score computation, and Welch's t-test) — https://github.com/multitalk/scCrossTalk
- **Seurat** (Preprocessing, dimensionality reduction, clustering, and annotation of the SRP313633 scRNA-seq cohort to identify MAIT cells, GZMB+ GZMK+ NK cells, and S100A12+ neutrophils) — https://github.com/multitalk/scCrossTalk
- **scCrossTalk** (Source of the marker definitions and cell-cell communication context for the niche cell populations defined in the manuscript) — https://github.com/multitalk/scCrossTalk
- **ggplot2** (Rendering the EAD vs non-EAD niche-score comparison boxplot) — https://github.com/multitalk/scCrossTalk

## Examples

```
obj <- create_scCrossTalk(sc_data, sc_celltype, species); obj <- find_lrpairs(object, lrpairs = lrpairs)
```

## Evaluation signals

- The 16-sample GSE23649 output table (niche_ead_validation.csv) is complete with one niche score per sample and non-missing group labels.
- Welch's t-test yields a significant niche-score difference between EAD and non-EAD groups in the same direction as reported (EAD samples scoring higher).
- The scRNA-seq cohort (SRP313633) niche populations are annotated to match the scCrossTalk manuscript marker definitions (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils).
- Output CSVs contain the per-sample niche scores, group labels, test statistic and p-value as required by the workflow.
- Bulk count matrix normalization is verified by median-scaled library sizes of 1000 per sample prior to log transformation.

## Limitations

- Validation depends on cohort comparability: the bulk cohort must be restricted to DBD (donation after brain death) recipients (8 EAD vs 8 non-EAD), and scRNA-seq samples must preserve donor/time-stage metadata (cold perfusion vs portal reperfusion).
- The association is a one-sided direction-of-effect check; failure to reproduce requires distinguishing low cohort power (n=16) from a true mismatch in the niche definition.
- No changelog was found for the source repository, so version drift between the published scCrossTalk marker definitions and newer package versions is untracked.

## Evidence

- [other] Normalize the count matrix by scaling each sample to a median value of 1000 followed by a log transformation.: [evidence span withheld — anchor status: not_found]
- [other] Perform standard Seurat preprocessing (normalization, dimensionality reduction, clustering) and annotate cell populations: [evidence span withheld — anchor status: not_found]
