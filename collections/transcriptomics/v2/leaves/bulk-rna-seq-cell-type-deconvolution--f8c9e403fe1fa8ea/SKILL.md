---
name: bulk-rna-seq-cell-type-deconvolution
description: Use when you have bulk RNA-seq expression matrix plus sample metadata
  and an annotated plant single-cell reference (e.g. Seurat object from Arabidopsis
  root scRNA-seq), and need cell-type fraction estimates per sample.
when_to_use_negative:
- Input is already a single-cell dataset needing annotation or clustering rather than
  bulk data — use scPlant's cell-type annotation module instead.
- No annotated single-cell reference atlas is available for the organism/tissue, since
  decomposeBulk requires a labeled Seurat reference to estimate fractions.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPlant
  role: Provides the decomposeBulk and fractionHmp functions used to deconvolve bulk
    RNA-seq against a single-cell reference and visualize fractions
  repo: https://github.com/compbioNJU/scPlant
- name: RNAMagnet
  role: Supplies the underlying runCIBERSORT deconvolution algorithm wrapped by scPlant's
    decomposeBulk
  repo: https://github.com/veltenlab/rnamagnet
- name: Seurat
  role: Format of the annotated single-cell reference object passed as input to decomposeBulk
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:1fd1ad8e433bbee56009a597e2751a88cb28ef8d22711799188858c0657a3987
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bulk-rna-seq-cell-type-deconvolution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantify cell-type composition of bulk RNA-seq samples by deconvolving them against a single-cell reference: scPlant's decomposeBulk function (wrapping RNAMagnet's runCIBERSORT) estimates cell-type fractions from an annotated Seurat single-cell reference, and fractionHmp visualizes them as a per-sample heatmap.

## When to use

Use when you have bulk RNA-seq expression matrix plus sample metadata and an annotated plant single-cell reference (e.g. Seurat object from Arabidopsis root scRNA-seq), and need cell-type fraction estimates per sample.

## When NOT to use

- Input is already a single-cell dataset needing annotation or clustering rather than bulk data — use scPlant's cell-type annotation module instead.
- No annotated single-cell reference atlas is available for the organism/tissue, since decomposeBulk requires a labeled Seurat reference to estimate fractions.

## Inputs

- Annotated single-cell reference as a Seurat object with cell-type labels (e.g. from PRJNA517021 Arabidopsis root scRNA-seq, Zhang et al., 2019)
- Bulk RNA-seq expression matrix (genes x samples)
- Sample metadata table for the bulk RNA-seq samples (e.g. stress-condition labels)

## Outputs

- Per-sample cell-type fraction estimates from decomposeBulk
- Heatmap of cell-type fractions per bulk sample produced by fractionHmp
- Per-condition cell-type composition comparison (e.g. vasculature and companion cell fractions across stressors)

## How to apply

Load a pre-processed single-cell reference Seurat object with annotated cell-type labels (in the demonstrated task, built from the PRJNA517021 Arabidopsis root scRNA-seq data of Zhang et al., 2019). Load the bulk RNA-seq expression matrix and sample metadata for the stress-conditioned samples. Call decomposeBulk(SeuratObj, exprs, meta), which internally wraps the runCIBERSORT function from the RNAMagnet R package (install via devtools::install_github("veltenlab/rnamagnet"); MAGIC must be available as a python dependency), to estimate each cell type's fraction in each bulk sample. Pass the deconvolution result and metadata to fractionHmp to draw a heatmap of cell-type fractions per sample. Finally extract per-condition fractions (e.g. vasculature and companion cells across stress treatments) and compare them quantitatively. Judge correctness by confirming the heatmap renders and that fraction patterns qualitatively reproduce the reported figure (here, Supplemental Figure 2).

## Related tools

- **scPlant** (Provides the decomposeBulk and fractionHmp functions used to deconvolve bulk RNA-seq against a single-cell reference and visualize fractions) — https://github.com/compbioNJU/scPlant
- **RNAMagnet** (Supplies the underlying runCIBERSORT deconvolution algorithm wrapped by scPlant's decomposeBulk) — https://github.com/veltenlab/rnamagnet
- **Seurat** (Format of the annotated single-cell reference object passed as input to decomposeBulk)

## Examples

```
result <- decomposeBulk(SeuratObj, exprs, meta); fractionHmp(result, meta)
```

## Evaluation signals

- decomposeBulk returns a result object containing non-negative per-sample fraction estimates for every annotated cell type in the reference
- fractionHmp successfully produces a heatmap with one column per bulk sample and one row per cell type
- Estimated fractions are comparable in scale across samples (composition profile sums to ~1 per sample, as expected from deconvolution)
- Extracted vasculature and companion cell fractions qualitatively agree with the stress-dependent composition changes reported in Supplemental Figure 2

## Limitations

- Deconvolution quality depends entirely on the reference atlas: the demonstrated case relies on the Zhang et al. (2019) Arabidopsis root scRNA-seq reference and its cluster-specific marker genes; missing or mislabeled cell types will bias fraction estimates.
- Results were validated only qualitatively (visual agreement against Supplemental Figure 2), not by a quantitative accuracy metric.
- RNAMagnet installation requires MAGIC as a python package, which can complicate setup, and the vignettes assume Seurat v3 objects (though functions work with v2).

## Evidence

- [methods] decomposeBulk wraps runCIBERSORT: "result <- decomposeBulk(SeuratObj, exprs, meta)"
- [methods] RNAMagnet provides the deconvolution backend: "To install RNAMagnet R package, see https://github.com/veltenlab/rnamagnet."
- [readme] MAGIC is a required dependency of RNAMagnet: [evidence span withheld — anchor status: external_doc]
- [intro] scPlant advertises deconvolution for bulk data as a core capability: "including automatic cell-type annotation (wherever possible) and deconvolution (for bulk data), trajectory inference, cross-species data integration, and cell-type-specific gene regulatory"
- [methods] The demonstrated reference dataset is the Zhang et al. 2019 Arabidopsis scRNA-seq data: "We provide a real scRNA-seq data of Arabidopsis thaliana ([Zhang et al., 2019]"
