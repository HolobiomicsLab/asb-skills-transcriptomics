---
name: cibersort-signature-based-fraction-estimation
description: Use when you have a bulk RNA-seq expression matrix plus sample metadata
  and an annotated single-cell reference atlas, and you need to quantify per-sample
  cell-type fractions across conditions.
when_to_use_negative:
- Input is already single-cell data or you only need clustering/annotation — use scPlant's
  QCfun/RunSeurat and annotation modules instead of deconvolution
- No annotated single-cell reference atlas is available for the tissue/organism, so
  reference signatures cannot be built
- The bulk data are not from the same species/reference gene space as the single-cell
  atlas (cross-species mixing would invalidate signature matching)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scPlant
  role: Provides the decomposeBulk deconvolution function and fractionHmp heatmap
    visualization that implement this skill
  repo: https://github.com/compbioNJU/scPlant
- name: RNAMagnet
  role: R package whose runCIBERSORT function is wrapped by decomposeBulk to perform
    the signature-based fraction estimation
  repo: https://github.com/veltenlab/rnamagnet
- name: Seurat
  role: Used to build and hold the annotated single-cell reference object passed to
    decomposeBulk
- name: CIBERSORT
  role: Underlying algorithm for signature-based deconvolution of bulk samples
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:ef4f64eb399a3ee9297284bf128dded2dd438031ee07b4b11bb27240b691e4da
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compbioNJU/scPlant
  repo_url: https://github.com/compbioNJU/scPlant
  tools:
  - CIBERSORT
  - RNAMagnet
  - Seurat
  - scPlant
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cibersort-signature-based-fraction-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates the cell-type composition of bulk RNA-seq samples by deconvolving them against a single-cell reference (Seurat object with annotated labels) using scPlant's decomposeBulk, which wraps CIBERSORT via the RNAMagnet R package. In the scPlant paper this quantified stress-dependent shifts in Arabidopsis root vasculature and companion cell fractions.

## When to use

Use when you have a bulk RNA-seq expression matrix plus sample metadata and an annotated single-cell reference atlas, and you need to quantify per-sample cell-type fractions across conditions.

## When NOT to use

- Input is already single-cell data or you only need clustering/annotation — use scPlant's QCfun/RunSeurat and annotation modules instead of deconvolution
- No annotated single-cell reference atlas is available for the tissue/organism, so reference signatures cannot be built
- The bulk data are not from the same species/reference gene space as the single-cell atlas (cross-species mixing would invalidate signature matching)

## Inputs

- Pre-processed single-cell reference Seurat object with annotated cell-type labels (e.g., Arabidopsis root scRNA-seq, Zhang et al., 2019, PRJNA517021)
- Bulk RNA-seq expression matrix (genes x samples)
- Bulk sample metadata table (sample-to-condition mapping)

## Outputs

- Per-sample cell-type fraction estimates from decomposeBulk (result object)
- Heatmap of cell-type fractions per bulk sample drawn by fractionHmp
- Per-condition cell-type fraction comparisons (e.g., vasculature and companion cell fractions across stress treatments)

## How to apply

Load a pre-processed, cell-type-annotated Seurat object built from single-cell data (in the paper, the PRJNA517021 Arabidopsis root scRNA-seq reference of Zhang et al., 2019). Provide the bulk RNA-seq expression matrix and its sample metadata as inputs to scPlant's decomposeBulk, which calls runCIBERSORT from the RNAMagnet package to fit reference-signature-based cell-type fractions per bulk sample. Then call fractionHmp on the deconvolution result and metadata to draw a heatmap of each cell type's fraction per sample. Finally, extract the cell types of interest (e.g., vasculature and companion cells) per condition and compare across treatments to quantify compositional changes.

## Related tools

- **scPlant** (Provides the decomposeBulk deconvolution function and fractionHmp heatmap visualization that implement this skill) — https://github.com/compbioNJU/scPlant
- **RNAMagnet** (R package whose runCIBERSORT function is wrapped by decomposeBulk to perform the signature-based fraction estimation) — https://github.com/veltenlab/rnamagnet
- **Seurat** (Used to build and hold the annotated single-cell reference object passed to decomposeBulk)
- **CIBERSORT** (Underlying algorithm for signature-based deconvolution of bulk samples)

## Examples

```
result <- decomposeBulk(SeuratObj, exprs, meta); fractionHmp(result, meta)
```

## Evaluation signals

- fractionHmp produces a heatmap without error, confirming the deconvolution result and metadata are well-formed
- Estimated fractions per sample are non-negative and sum to ~1 across cell types
- Vasculature and companion cell fractions qualitatively agree with the stress-dependent composition changes shown in Supplemental Figure 2
- Deconvolution uses the same gene space as the annotated Seurat reference (e.g., TAIR10 gene IDs for Arabidopsis) — no major gene ID mismatch between bulk matrix and reference

## Limitations

- Requires a species- and tissue-matched annotated single-cell reference; accuracy depends on the quality and completeness of the reference atlas
- RNAMagnet (and thus decomposeBulk) requires MAGIC as a python package dependency, which can complicate installation
- Deconvolution of bulk data is an advanced add-on module in scPlant; the underlying CIBERSORT approach may struggle with cell types absent from the reference or with very dissimilar bulk conditions

## Evidence

- [methods] decomposeBulk wraps runCIBERSORT: "result <- decomposeBulk(SeuratObj, exprs, meta)"
- [readme] RNAMagnet install: [evidence span withheld — anchor status: external_doc]
- [abstract] deconvolution listed as supported analysis: [evidence span withheld — anchor status: not_found]
