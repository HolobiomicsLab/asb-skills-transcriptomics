---
name: stress-condition-composition-comparison
description: Use when you have bulk RNA-seq expression matrices plus sample metadata
  for samples spanning distinct stress treatments, and an annotated single-cell reference
  atlas, and you need to quantify how cell-type composition changes across conditions.
when_to_use_negative:
- Input is already single-cell data or a cell-composition table rather than bulk RNA-seq
  profiles needing deconvolution
- No annotated single-cell reference atlas exists for the organism/tissue, so cluster-specific
  marker signatures cannot be built
- The goal is cell-type annotation of single-cell data or trajectory inference — these
  are separate scPlant modules (RunSeurat, RunMonocle2/3, RunSlingshot)
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scPlant
  role: Provides the decomposeBulk function that wraps CIBERSORT-based deconvolution
    of bulk RNA-seq against the single-cell reference, plus fractionHmp for fraction
    heatmaps
  repo: https://github.com/compbioNJU/scPlant
- name: RNAMagnet
  role: R package whose runCIBERSORT function is wrapped by scPlant's decomposeBulk
    to estimate cell-type composition
  repo: https://github.com/veltenlab/rnamagnet
- name: Seurat
  role: Format of the single-cell reference object with cell-type labels supplied
    to decomposeBulk
- name: CIBERSORT
  role: Underlying deconvolution algorithm executed via RNAMagnet's runCIBERSORT
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:80f7b7a468231657b69532ee605bdf1ddd58a3d01d962ac02e068f52668a2078
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# stress-condition-composition-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Deconvolves bulk RNA-seq samples collected under different stress conditions against a single-cell reference (e.g., the PRJNA517021 Arabidopsis root scRNA-seq atlas from Zhang et al., 2019) using scPlant's decomposeBulk, then compares estimated cell-type fractions per condition to identify composition shifts such as changes in vasculature and companion cells.

## When to use

Use when you have bulk RNA-seq expression matrices plus sample metadata for samples spanning distinct stress treatments, and an annotated single-cell reference atlas, and you need to quantify how cell-type composition changes across conditions.

## When NOT to use

- Input is already single-cell data or a cell-composition table rather than bulk RNA-seq profiles needing deconvolution
- No annotated single-cell reference atlas exists for the organism/tissue, so cluster-specific marker signatures cannot be built
- The goal is cell-type annotation of single-cell data or trajectory inference — these are separate scPlant modules (RunSeurat, RunMonocle2/3, RunSlingshot)

## Inputs

- Pre-processed single-cell reference Seurat object with annotated cell-type labels (e.g., Arabidopsis root scRNA-seq, PRJNA517021, Zhang et al., 2019)
- Bulk RNA-seq expression matrix (genes x samples) for stress-conditioned samples
- Sample metadata table assigning bulk samples to stress conditions

## Outputs

- Per-sample cell-type fraction estimates from decomposeBulk
- Heatmap (via fractionHmp) of cell-type fractions across bulk RNA-seq samples
- Per-condition comparison of target cell-type fractions (e.g., vasculature, companion cells)

## How to apply

Load the pre-processed single-cell reference Seurat object with annotated cell-type labels, the bulk RNA-seq expression matrix, and the sample metadata for the stress-conditioned samples. Call scPlant's decomposeBulk — which wraps runCIBERSORT from the RNAMagnet R package — passing the Seurat object, expression matrix, and metadata (result <- decomposeBulk(SeuratObj, exprs, meta)) to estimate cell-type proportions per bulk sample from cluster-specific marker signatures. Apply fractionHmp to the deconvolution result and metadata to render a heatmap of cell-type fractions per sample, enabling visual comparison across stressors. Finally, extract fractions for cell types of interest (e.g., vasculature and companion cells) per condition and compare them across treatments to quantify stress-dependent composition changes (Supplemental Figure 2). Judgment of correct application: the heatmap is produced, fractions per sample sum to coherent proportions, and vasculature/companion cell fractions qualitatively reproduce the reported stress-dependent shifts.

## Related tools

- **scPlant** (Provides the decomposeBulk function that wraps CIBERSORT-based deconvolution of bulk RNA-seq against the single-cell reference, plus fractionHmp for fraction heatmaps) — https://github.com/compbioNJU/scPlant
- **RNAMagnet** (R package whose runCIBERSORT function is wrapped by scPlant's decomposeBulk to estimate cell-type composition) — https://github.com/veltenlab/rnamagnet
- **Seurat** (Format of the single-cell reference object with cell-type labels supplied to decomposeBulk)
- **CIBERSORT** (Underlying deconvolution algorithm executed via RNAMagnet's runCIBERSORT)

## Examples

```
result <- decomposeBulk(SeuratObj, exprs, meta); fractionHmp(result, meta)
```

## Evaluation signals

- fractionHmp produces a heatmap of cell-type fractions for every bulk RNA-seq sample
- Estimated fractions per sample are non-negative and coherent as composition estimates across cell types
- Vasculature and companion cell fractions vary substantially across stress conditions, reproducing the stress-dependent composition changes shown in Supplemental Figure 2 (qualitative agreement check)
- decomposeBulk returns a result object for all samples without errors when given the Seurat reference, expression matrix, and metadata

## Limitations

- Deconvolution quality depends on the single-cell reference matching the tissue and organism of the bulk data (here Arabidopsis root); references from other tissues/species are unsupported
- Agreement with the published result is a qualitative check against Supplemental Figure 2 rather than a quantitative benchmark
- RNAMagnet has additional requirements (MAGIC must be installed as a Python package; vignettes assume Seurat v3, though functions work with v2 and v3 objects)

## Evidence
