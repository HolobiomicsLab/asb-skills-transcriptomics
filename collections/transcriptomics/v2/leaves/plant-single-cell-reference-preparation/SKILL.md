---
name: plant-single-cell-reference-preparation
description: 'Use when a bulk RNA-seq deconvolution task in plants needs a cell-type-resolved
  scRNA-seq reference: an annotated single-cell dataset (e.g. Arabidopsis root Seurat
  object) must be processed and its labels/marker genes readied before running decomposeBulk.'
when_to_use_negative:
- The input is already a bulk expression matrix with no single-cell reference available
  — deconvolution requires an annotated scRNA-seq reference and cannot proceed from
  bulk data alone
- The goal is cross-species single-cell integration or trajectory inference rather
  than bulk deconvolution — use crossSpecies_integrate or RunMonocle2/3/RunSlingshot
  instead
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0780
tools:
- name: scPlant
  role: Provides readData, QCfun, RunSeurat, and decomposeBulk — the reference-building
    and deconvolution entry points
  repo: https://github.com/compbioNJU/scPlant
- name: Seurat
  role: Underlying object model and processing methods for the single-cell reference
    object
- name: RNAMagnet
  role: runCIBERSORT, wrapped by scPlant decomposeBulk, performs the CIBERSORT-style
    estimation against the prepared reference
  repo: https://github.com/veltenlab/rnamagnet
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:71c0595547741046e85a6310e541947627464d1133b10cbe72ff7c0d88770f6f
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
  - RNAMagnet
  - Seurat
  - scPlant
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# plant-single-cell-reference-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare an annotated plant single-cell reference (Seurat object with cell-type labels, e.g. the PRJNA517021 Arabidopsis root scRNA-seq data of Zhang et al., 2019) so that scPlant's decomposeBulk function can deconvolve bulk RNA-seq into cell-type fractions. The reference's cluster-specific marker signatures are the basis for estimating cell-type composition.

## When to use

Use when a bulk RNA-seq deconvolution task in plants needs a cell-type-resolved scRNA-seq reference: an annotated single-cell dataset (e.g. Arabidopsis root Seurat object) must be processed and its labels/marker genes readied before running decomposeBulk.

## When NOT to use

- The input is already a bulk expression matrix with no single-cell reference available — deconvolution requires an annotated scRNA-seq reference and cannot proceed from bulk data alone
- The goal is cross-species single-cell integration or trajectory inference rather than bulk deconvolution — use crossSpecies_integrate or RunMonocle2/3/RunSlingshot instead

## Inputs

- Annotated plant scRNA-seq dataset (e.g. Arabidopsis root scRNA-seq, Zhang et al., 2019, PRJNA517021) with cell-type labels
- Raw or processed single-cell expression matrix compatible with scPlant readData (type='expMatrix')
- Seurat object with cluster-level cell-type annotations
- Companion bulk RNA-seq expression matrix and sample metadata to be deconvolved against the reference

## Outputs

- Pre-processed Seurat single-cell reference object with cell-type labels suitable for decomposeBulk
- Cluster-specific signature gene sets per annotated cell type (internal to decomposeBulk)
- Per-sample cell-type composition estimates and fractionHmp heatmap (downstream products of the prepared reference)

## How to apply

Load a pre-processed plant scRNA-seq dataset with annotated cell-type labels (in the article, the Arabidopsis thaliana root data of Zhang et al., 2019) into a Seurat object, using scPlant's readData for expression matrices (type='expMatrix') and its Seurat-integrated QCfun and RunSeurat steps for basic processing if starting from raw counts. Confirm each cluster carries a biologically meaningful cell-type label, since decomposeBulk derives cluster-specific signature genes from these labels to wrap runCIBERSORT from RNAMagnet. Keep the Seurat object, the bulk expression matrix, and the bulk sample metadata aligned so that result <- decomposeBulk(SeuratObj, exprs, meta) can run directly; fractions per cell type per sample are then visualized with fractionHmp. The reference quality directly bounds deconvolution quality: mislabeled or coarse cell types (e.g. failing to separate vasculature from companion cells) will blur the composition changes the deconvolution is meant to reveal.

## Related tools

- **scPlant** (Provides readData, QCfun, RunSeurat, and decomposeBulk — the reference-building and deconvolution entry points) — https://github.com/compbioNJU/scPlant
- **Seurat** (Underlying object model and processing methods for the single-cell reference object)
- **RNAMagnet** (runCIBERSORT, wrapped by scPlant decomposeBulk, performs the CIBERSORT-style estimation against the prepared reference) — https://github.com/veltenlab/rnamagnet

## Examples

```
result <- decomposeBulk(SeuratObj, exprs, meta)
```

## Evaluation signals

- The Seurat object loads with cell-type-level metadata columns present for every cluster (e.g. vasculature, companion cells distinctly labeled)
- decomposeBulk runs without error: result <- decomposeBulk(SeuratObj, exprs, meta) returns per-sample fractions for each reference cell type
- Estimated fractions per sample are non-negative and approximately sum to 1 across cell types
- fractionHmp produces a heatmap of cell-type fraction per bulk sample without missing cell-type rows
- Deconvolved vasculature and companion cell fractions qualitatively reproduce the stress-dependent composition changes reported in Supplemental Figure 2

## Limitations

- Reference labels bound resolution: only cell types annotated in the single-cell reference can be quantified in bulk samples
- scPlant is stated as free for non-commercial purposes
- RNAMagnet requires MAGIC as a python package dependency for some functionality, adding installation complexity

## Evidence
