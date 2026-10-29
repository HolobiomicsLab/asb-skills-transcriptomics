---
name: single-cell-rnaseq-reference-loading
description: Use when a raw scRNA-seq reference (e.g. 10x Chromium, GSE173778) and
  a spatial transcriptomics dataset (e.g. 10x Visium slides) must be ingested as the
  starting point of a SpatialDDLS deconvolution workflow.
when_to_use_negative:
- Input is already a SpatialDDLS object with shared genes selected — re-running createSpatialDDLSobject
  would repeat filtering unnecessarily.
- The goal is to evaluate an already-trained model on simulated profiles rather than
  ingest raw single-cell and spatial data.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SpatialDDLS
  role: Provides createSpatialDDLSobject and the SpatialDDLS object class; installed
    from CRAN
  repo: https://CRAN.R-project.org/package=SpatialDDLS
- name: SpatialDDLSdata
  role: Supplies the MouseDLN.SCE and MouseDLN.ST example datasets loaded with data()
  repo: https://github.com/diegommcc/SpatialDDLSdata
- name: SingleCellExperiment
  role: Container class for the scRNA-seq reference passed to createSpatialDDLSobject
- name: SpatialExperiment
  role: Container class for the 10x Visium spatial data passed to createSpatialDDLSobject
- name: devtools
  role: Used to install SpatialDDLSdata from GitHub
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae072
    title: 'SpatialDDLS: an R package to deconvolute spatial transcriptomics data
      using neural networks'
  source_content_hash: sha256:f2fe6cdbcef36e7a86f427ce7fd7ed23295c02a52da5de86c6792dac14f6d459
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SingleCellExperiment
  - SpatialDDLS
  - SpatialDDLSdata
  - SpatialExperiment
  - devtools
  - ggplot2
  - ggpubr
  - keras
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# single-cell-rnaseq-reference-loading

## Summary

Loads paired scRNA-seq and spatial transcriptomics reference data (SingleCellExperiment and SpatialExperiment objects) into a SpatialDDLS object via createSpatialDDLSobject, applying logFC-based per-cluster gene filtering so downstream deconvolution models train on shared informative genes.

## When to use

Use when a raw scRNA-seq reference (e.g. 10x Chromium, GSE173778) and a spatial transcriptomics dataset (e.g. 10x Visium slides) must be ingested as the starting point of a SpatialDDLS deconvolution workflow.

## When NOT to use

- Input is already a SpatialDDLS object with shared genes selected — re-running createSpatialDDLSobject would repeat filtering unnecessarily.
- The goal is to evaluate an already-trained model on simulated profiles rather than ingest raw single-cell and spatial data.

## Inputs

- SingleCellExperiment object with scRNA-seq reference data (e.g. MouseDLN.SCE from 10x Chromium, GEO GSE173778)
- SpatialExperiment object with spatial transcriptomics data (e.g. MouseDLN.ST from 10x Visium slides)

## Outputs

- SpatialDDLS object (e.g. mouseDLN.SDDLS) containing the filtered single-cell reference, the spatial data, and the shared feature set ready for genMixedCellProp

## How to apply

Install SpatialDDLS from CRAN and SpatialDDLSdata from GitHub (devtools::install_github('diegommcc/SpatialDDLSdata')), then load MouseDLN.SCE (SingleCellExperiment with scRNA-seq) and MouseDLN.ST (SpatialExperiment with 10x Visium spatial data). Call createSpatialDDLSobject with sc.filt.genes.cluster = TRUE, sc.n.genes.per.cluster = 150, and sc.min.mean.counts = 2, which triggers logFC-based feature selection per cell cluster and yields 1041 genes after removing 11 mitochondrial genes. The function also intersects genes between the single-cell and spatial datasets (10941 shared genes) and removes 16 genes without expression in any cell, ensuring the neural network trains only on features present in both modalities.

## Related tools

- **SpatialDDLS** (Provides createSpatialDDLSobject and the SpatialDDLS object class; installed from CRAN) — https://CRAN.R-project.org/package=SpatialDDLS
- **SpatialDDLSdata** (Supplies the MouseDLN.SCE and MouseDLN.ST example datasets loaded with data()) — https://github.com/diegommcc/SpatialDDLSdata
- **SingleCellExperiment** (Container class for the scRNA-seq reference passed to createSpatialDDLSobject)
- **SpatialExperiment** (Container class for the 10x Visium spatial data passed to createSpatialDDLSobject)
- **devtools** (Used to install SpatialDDLSdata from GitHub)

## Examples

```
library("SpatialDDLS"); data(MouseDLN.SCE); data(MouseDLN.ST); mouseDLN.SDDLS <- createSpatialDDLSobject(sc.filt.genes.cluster = TRUE, sc.n.genes.per.cluster = 150, sc.min.mean.counts = 2)
```

## Evaluation signals

- Console reports '=== Number of shared genes between single-cell and spatial transcriptomics datasets: 10941' for the mouse lymph node example.
- LogFC-based filtering reduces the reference to 1041 genes ('=== Number of genes after filtering based on logFC: 1041').
- 11 mitochondrial genes are removed ('=== Number of removed mitochondrial genes: 11').
- 16 genes without expression in any cell are dropped during preprocessing.
- The returned object is a valid SpatialDDLS object that can be passed directly to genMixedCellProp without errors.

## Limitations

- Feature selection parameters (sc.n.genes.per.cluster, sc.min.mean.counts) directly determine the gene set size; the 1041-gene result is specific to the MouseDLN example with these settings.
- A working Python interpreter with TensorFlow is required for downstream model training; the installTFpython function installs a conda environment named spatialddls-env with these dependencies.
- The bundled MouseDLN.SCE/MouseDLN.ST data are specific to murine lymph nodes (10x Visium + 10x Chromium, GSE173778); other datasets require their own SingleCellExperiment/SpatialExperiment objects.

## Evidence

- [methods] Filter step: createSpatialDDLSobject parameters: "sc.filt.genes.cluster = TRUE, sc.n.genes.per.cluster = 150, sc.min.mean.counts = 2"
- [methods] LogFC filtering result: "=== Number of genes after filtering based on logFC: 1041"
- [methods] Shared gene intersection between modalities: "=== Number of shared genes between single-cell and spatial transcriptomics datasets: 10941"
- [methods] Data loading from SpatialDDLSdata: "# SingleCellExperiment with scRNA-seq
data(MouseDLN.SCE)"
- [methods] Dataset installation from GitHub: "devtools::install_github("diegommcc/SpatialDDLSdata")"
- [readme] Python/TensorFlow dependency for the R package: [evidence span withheld — anchor status: external_doc]
