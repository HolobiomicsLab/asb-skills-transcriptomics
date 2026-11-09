---
name: bulk-rnaseq-count-normalization
description: 'Use when a raw bulk RNA-seq count matrix (e.g., GEO GSE23649: 8 DBD
  EAD vs 8 DBD non-EAD) must be made cross-sample comparable before computing per-sample
  gene-signature scores for group tests such as EAD vs non-EAD.'
when_to_use_negative:
- Input is already a normalized expression matrix (TPM/log-CPM) or a single-cell Seurat
  object — median-1000 scaling is meant for raw integer bulk counts only.
- The goal is differential gene expression calling between conditions; median scaling
  plus log transform supports signature scoring, not DESeq2/edgeR-style dispersion-based
  DE testing.
- Samples lack a two-group design with EAD/non-EAD labels, since the downstream step
  is a two-sample Welch's t-test comparison.
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Environment for retrieving the GSE23649 count matrix/metadata from GEO, applying
    the median-1000 scaling plus log transform, and running Welch's t-test
- name: ggplot2
  role: Renders the EAD vs non-EAD comparison boxplot of per-sample niche signature
    scores
- name: scCrossTalk
  role: Source single-cell method whose EAD immune-niche definition (MAIT cells, GZMB+
    GZMK+ NK cells, S100A12+ neutrophils) supplies the marker genes scored on the
    bulk data
  repo: https://github.com/multitalk/scCrossTalk
- name: Seurat
  role: Upstream single-cell framework (scCrossTalk is compatible with Seurat 5) in
    which the niche populations were defined; not applied to the bulk counts directly
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.eng.2023.12.004
    title: A Single-Cell Landscape of Human Liver Transplantation Reveals a Pathogenic
      Immune Niche Associated with Early Allograft Dysfunction
  source_content_hash: sha256:6ace7f1ec9e90894418e297d2cb388f99c170c767b90e0e20690259bdcc08544
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
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# bulk-rnaseq-count-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalizes a raw bulk RNA-seq count matrix by scaling each sample to a fixed median of 1000 followed by log transformation, making per-sample gene-signature scores comparable across samples. In this paper it validates the single-cell-defined EAD immune niche (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) on the GSE23649 DBD cohort (8 EAD vs 8 non-EAD).

## When to use

Use when a raw bulk RNA-seq count matrix (e.g., GEO GSE23649: 8 DBD EAD vs 8 DBD non-EAD) must be made cross-sample comparable before computing per-sample gene-signature scores for group tests such as EAD vs non-EAD.

## When NOT to use

- Input is already a normalized expression matrix (TPM/log-CPM) or a single-cell Seurat object — median-1000 scaling is meant for raw integer bulk counts only.
- The goal is differential gene expression calling between conditions; median scaling plus log transform supports signature scoring, not DESeq2/edgeR-style dispersion-based DE testing.
- Samples lack a two-group design with EAD/non-EAD labels, since the downstream step is a two-sample Welch's t-test comparison.

## Inputs

- Raw bulk RNA-seq count matrix (integer counts) for GSE23649 retrieved from GEO — 8 DBD EAD vs 8 DBD non-EAD samples
- Sample metadata mapping each sample to EAD status and perfusion timepoint (Sample A pre-cold-perfusion, Sample B 2h post-portal-reperfusion)
- Marker-gene lists for the niche cell populations (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) defined in the single-cell landscape

## Outputs

- Sample-median-scaled (median = 1000) and log-transformed expression matrix
- Per-sample immune-niche signature scores for all 16 samples
- Welch's t-test statistic and p-value for the EAD vs non-EAD niche-score comparison
- niche_ead_validation.csv containing per-sample niche scores and group-comparison statistics
- EAD vs non-EAD comparison boxplot rendered with ggplot2

## How to apply

Retrieve the GSE23649 raw count matrix and sample metadata (8 DBD EAD vs 8 DBD non-EAD; Sample A pre-cold-perfusion, Sample B 2h post-portal-reperfusion) from GEO in R. Scale each sample's counts so its median equals 1000 — a library-size correction that avoids DESeq2/edgeR dispersion fitting — then apply a log transformation (log1p/log2) to the scaled values. On the normalized matrix, compute a per-sample EAD immune-niche signature score by aggregating the marker genes of the niche populations defined in the single-cell landscape (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils), restricted to genes present in the bulk matrix. Compare EAD vs non-EAD scores with Welch's t-test — the paper's comparator for cross-group score differences (cf. the one-sided Welch's test used on LR-pair number/score) — and record the test statistic and p-value. Write the 16-sample score table and comparison statistics to niche_ead_validation.csv and render an EAD vs non-EAD ggplot2 boxplot; the association reproduces only if EAD samples score significantly higher.

## Related tools

- **R** (Environment for retrieving the GSE23649 count matrix/metadata from GEO, applying the median-1000 scaling plus log transform, and running Welch's t-test)
- **ggplot2** (Renders the EAD vs non-EAD comparison boxplot of per-sample niche signature scores)
- **scCrossTalk** (Source single-cell method whose EAD immune-niche definition (MAIT cells, GZMB+ GZMK+ NK cells, S100A12+ neutrophils) supplies the marker genes scored on the bulk data) — https://github.com/multitalk/scCrossTalk
- **Seurat** (Upstream single-cell framework (scCrossTalk is compatible with Seurat 5) in which the niche populations were defined; not applied to the bulk counts directly)

## Examples

```
scaled <- log1p(sweep(counts, 2, apply(counts, 2, median), "/") * 1000)
```

## Evaluation signals

- niche_ead_validation.csv contains a complete 16-sample table (8 EAD + 8 non-EAD) with no missing rows or scores.
- Post-scaling invariant: each sample's counts have a median of exactly 1000 before the log transformation is applied.
- Welch's t-test returns a recorded test statistic and p-value, and EAD samples score significantly higher than non-EAD (same direction as reported).
- Only niche marker genes present in the bulk matrix gene identifiers contribute to each score, and all scores are finite (no NA/Inf).
- The ggplot2 boxplot renders both groups (EAD vs non-EAD) labeled from the sample metadata.

## Limitations

- Median-1000 scaling is a composition-agnostic library-size correction with no dispersion modeling (unlike DESeq2/edgeR), so it supports signature scoring and simple group tests rather than formal differential expression calls.
- Reproduction is judged on a small cohort — 8 DBD EAD vs 8 DBD non-EAD — so the Welch's-test result depends on this limited n and the reported direction (EAD scores higher).
- The validation covers only the DBD samples with the Sample A (pre-cold-perfusion) / Sample B (2h post-portal-reperfusion) design; other donor types or timepoints are untested here.

## Evidence

- [other] GEO retrieval of the GSE23649 counts and metadata: [evidence span withheld — anchor status: not_found]
- [other] Normalization procedure: [evidence span withheld — anchor status: not_found]
- [other] Per-sample niche signature scoring: [evidence span withheld — anchor status: not_found]
- [other] Group comparison statistics: [evidence span withheld — anchor status: not_found]
- [abstract] Validation in independent cohorts: "we verified this immune niche and its association with EAD occurrence in two independent cohorts"
- [readme] scCrossTalk design context for EAD vs non-EAD: "scCrossTalk was designed in our manuscript to dissect the different mechanism underlying cell-cell communications within the transplanted livers of EAD and non-EAD patients."
