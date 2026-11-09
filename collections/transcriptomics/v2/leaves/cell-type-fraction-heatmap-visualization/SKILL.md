---
name: cell-type-fraction-heatmap-visualization
description: Use when bulk RNA-seq samples have been deconvolved against a single-cell
  reference (e.g., via scPlant decomposeBulk on an annotated Seurat object) and you
  need a heatmap of per-sample cell-type fractions to compare compositions across
  conditions.
when_to_use_negative:
- Input is raw single-cell count data that has not been clustered and cell-type annotated
  — deconvolution signatures require labeled clusters, so run scPlant's QCfun/RunSeurat/annotation
  steps first
- The goal is cell-type annotation of the single-cell data itself rather than estimating
  composition of bulk samples from a reference
- No bulk RNA-seq matrix or sample metadata is available — decomposeBulk requires
  both (SeuratObj, exprs, meta)
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPlant
  role: Provides the decomposeBulk deconvolution function and the fractionHmp heatmap
    function used to estimate and visualize per-sample cell-type fractions
  repo: https://github.com/compbioNJU/scPlant
- name: RNAMagnet
  role: R package whose runCIBERSORT function is wrapped by scPlant decomposeBulk
    to estimate cell-type composition of bulk samples from single-cell reference signatures
  repo: https://github.com/veltenlab/rnamagnet
- name: Seurat
  role: Provides the Seurat object holding the annotated single-cell reference passed
    as the first argument to decomposeBulk
- name: CIBERSORT
  role: Underlying deconvolution algorithm invoked via RNAMagnet's runCIBERSORT inside
    decomposeBulk
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:f9da0bcbc6e1b1da7b6f695ef870ed7bd3de4f33dc1b14567afe114256d02ce8
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cell-type-fraction-heatmap-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Visualize scPlant deconvolution results by calling fractionHmp on the decomposeBulk output to draw a heatmap of the estimated fraction of each cell type (e.g., Arabidopsis root vasculature and companion cells) in each bulk RNA-seq sample, enabling comparison of cell-type composition across conditions such as stress treatments.

## When to use

Use when bulk RNA-seq samples have been deconvolved against a single-cell reference (e.g., via scPlant decomposeBulk on an annotated Seurat object) and you need a heatmap of per-sample cell-type fractions to compare compositions across conditions.

## When NOT to use

- Input is raw single-cell count data that has not been clustered and cell-type annotated — deconvolution signatures require labeled clusters, so run scPlant's QCfun/RunSeurat/annotation steps first
- The goal is cell-type annotation of the single-cell data itself rather than estimating composition of bulk samples from a reference
- No bulk RNA-seq matrix or sample metadata is available — decomposeBulk requires both (SeuratObj, exprs, meta)

## Inputs

- Pre-processed single-cell reference Seurat object with annotated cell-type labels (e.g., Arabidopsis root scRNA-seq from PRJNA517021, Zhang et al. 2019)
- Bulk RNA-seq expression matrix for the samples of interest (e.g., stress-conditioned samples)
- Sample metadata mapping each bulk sample to its condition (e.g., stress treatment)

## Outputs

- Deconvolution result: estimated cell-type composition (fractions) per bulk sample
- Heatmap (from fractionHmp) showing the fraction of each cell type in each bulk RNA-seq sample
- Extracted per-condition cell-type fractions (e.g., vasculature and companion cell fractions) for cross-condition comparison

## How to apply

First run scPlant's decomposeBulk function — which wraps runCIBERSORT from the RNAMagnet R package — passing the pre-processed, cell-type-annotated single-cell reference Seurat object (e.g., built from the PRJNA517021 Arabidopsis root scRNA-seq data of Zhang et al., 2019), the bulk expression matrix, and the sample metadata. Then call fractionHmp on the deconvolution result and the metadata to draw a heatmap showing the fraction of each cell type in each bulk RNA-seq sample. The decision point is comparative: extract the fractions for cell types of interest (here, vasculature and companion cells) per condition and compare them across stress treatments to quantify composition alterations. Correct application is judged by qualitative agreement of the heatmap with the published result (Supplemental Figure 2), where vasculature and companion cell composition is substantially altered across stressors.

## Related tools

- **scPlant** (Provides the decomposeBulk deconvolution function and the fractionHmp heatmap function used to estimate and visualize per-sample cell-type fractions) — https://github.com/compbioNJU/scPlant
- **RNAMagnet** (R package whose runCIBERSORT function is wrapped by scPlant decomposeBulk to estimate cell-type composition of bulk samples from single-cell reference signatures) — https://github.com/veltenlab/rnamagnet
- **Seurat** (Provides the Seurat object holding the annotated single-cell reference passed as the first argument to decomposeBulk)
- **CIBERSORT** (Underlying deconvolution algorithm invoked via RNAMagnet's runCIBERSORT inside decomposeBulk)

## Examples

```
result <- decomposeBulk(SeuratObj, exprs, meta); fractionHmp(result, meta)
```

## Evaluation signals

- fractionHmp produces a heatmap with one row per cell type and one column per bulk sample, populated with fractions from the decomposeBulk result
- Extracted vasculature and companion cell fractions vary across stress conditions (qualitative agreement with Supplemental Figure 2's stress-dependent composition changes)
- Per-sample cell-type fractions are interpretable as composition estimates (non-negative, derived from the CIBERSORT-based decomposition)
- The deconvolution uses cluster-specific marker genes from the annotated single-cell reference, not an ad hoc signature list
- Sample metadata passed to fractionHmp matches the column order/identifiers of the bulk expression matrix

## Limitations

- Quality depends on the single-cell reference: it must be pre-processed and carry accurate annotated cell-type labels (e.g., from Zhang et al., 2019 Arabidopsis root data)
- RNAMagnet is an important requirement and needs to be installed as instructed; decomposeBulk wraps runCIBERSORT from RNAMagnet, so the reference cell types must be representable in the CIBERSORT signature framework
- Validation in the source task is qualitative agreement against the reported figure (Supplemental Figure 2), not a quantitative benchmark

## Evidence

- [methods] decomposeBulk invocation: "result <- decomposeBulk(SeuratObj, exprs, meta)"
- [abstract] scPlant scope including deconvolution: "ranging from basic data processing to advanced demands such as cell-type annotation and deconvolution, trajectory inference"
- [readme] RNAMagnet installation requirement: [evidence span withheld — anchor status: external_doc]
- [intro] Deconvolution module description: "including automatic cell-type annotation (wherever possible) and deconvolution (for bulk data), trajectory inference, cross-species data integration"
