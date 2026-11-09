---
name: scrna-seq-dataset-acquisition-geo
description: Use when a reproduction or benchmark needs the exact processed single-cell
  RNA-seq data and precomputed UMAP/tSNE embeddings cited in a paper, and the paper's
  data-availability section names a GEO accession (e.g. GSE87038, GSE129788).
when_to_use_negative:
- You need raw sequencing data (FASTQ) or reprocessing from reads — the paper only
  traces processed matrices and precomputed embeddings via GEO, not a reprocessing
  pipeline.
- The dataset of interest is not deposited under a GEO accession (e.g. only on another
  archive); forcing a GEO lookup will fail.
- You only need to visualize data in CellexalVR itself rather than reproduce the 2D-vs-3D
  entropy analysis — then the cellexalvrR data-preparation workflow is the right entry
  point instead.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Environment (R 4.0.0+) in which GEO supplementary files and embeddings are
    downloaded and then handed to the authors' entropy code
  repo: https://cran.r-project.org/src/base/R-4/
- name: entropy_2dvs3d
  role: Downstream consumer of the downloaded mouse gastrulation UMAP/tSNE embeddings
    for Shannon-index overlap calculations
  repo: https://github.com/sonejilab/entropy_2dvs3d
- name: cellexalvrR
  role: R package that prepares single-cell expression data (as_cellexalvrR / export2cellexalvr)
    once downloaded for CellexalVR use
  repo: https://github.com/sonejilab/cellexalvrR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.103251
    title: 'CellexalVR: A virtual reality platform to visualize and analyze single-cell
      omics data'
  source_content_hash: sha256:8f930c0e20b4f5e0a0a61af2d00ca2d03d0de64fb5b8962b2b285955b2a3ee7a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - cellexalvrR
  - entropy_2dvs3d
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# scrna-seq-dataset-acquisition-geo

## Summary

Locate and download publicly deposited scRNA-seq datasets — expression matrices plus precomputed UMAP/tSNE cell-coordinate embeddings — from their GEO accessions (e.g. GSE87038 for mouse gastrulation) so that downstream analyses like the entropy-based 2D-vs-3D overlap comparison can be reproduced.

## When to use

Use when a reproduction or benchmark needs the exact processed single-cell RNA-seq data and precomputed UMAP/tSNE embeddings cited in a paper, and the paper's data-availability section names a GEO accession (e.g. GSE87038, GSE129788).

## When NOT to use

- You need raw sequencing data (FASTQ) or reprocessing from reads — the paper only traces processed matrices and precomputed embeddings via GEO, not a reprocessing pipeline.
- The dataset of interest is not deposited under a GEO accession (e.g. only on another archive); forcing a GEO lookup will fail.
- You only need to visualize data in CellexalVR itself rather than reproduce the 2D-vs-3D entropy analysis — then the cellexalvrR data-preparation workflow is the right entry point instead.

## Inputs

- GEO series accession ID (e.g. GSE87038)
- Processed scRNA-seq expression matrix from GEO supplementary files
- Cell-type/cluster annotation metadata
- Precomputed UMAP and tSNE cell-coordinate embeddings

## Outputs

- Locally downloaded expression matrix and metadata files for the GEO series
- UMAP/tSNE embedding coordinate files ready for entropy_2dvs3d analysis
- Folder per dataset (e.g. one folder per data set within a Data directory)

## How to apply

Extract the GEO series accession from the paper's data-availability or discussion section (this article lists GSE87038 for mouse gastrulation (Pijuan-Sala et al., 2019), GSE129788 for ageing brain, GSE81682 for mouse HSPC, GSE100866 for CITEseq). Download the processed supplementary files — expression matrix, cell-type/cluster annotations, and the precomputed UMAP/tSNE cell-coordinate embeddings — using GEOquery in an R 4.0.0+ environment. Confirm the cell identifiers in the embedding coordinate files match those in the expression matrix and metadata before feeding them to the entropy_2dvs3d code, which computes Shannon-index overlap for cell populations pairwise. Keep the downloaded folder structure stable so downstream scripts can resolve the coordinate files for each embedding (UMAP and tSNE).

## Related tools

- **R** (Environment (R 4.0.0+) in which GEO supplementary files and embeddings are downloaded and then handed to the authors' entropy code) — https://cran.r-project.org/src/base/R-4/
- **entropy_2dvs3d** (Downstream consumer of the downloaded mouse gastrulation UMAP/tSNE embeddings for Shannon-index overlap calculations) — https://github.com/sonejilab/entropy_2dvs3d
- **cellexalvrR** (R package that prepares single-cell expression data (as_cellexalvrR / export2cellexalvr) once downloaded for CellexalVR use) — https://github.com/sonejilab/cellexalvrR

## Examples

```
library(GEOquery); getGEOSuppFiles("GSE87038", makeDirectory = TRUE)
```

## Evaluation signals

- The GEO accession resolves on NCBI GEO and its series metadata matches the paper (mouse gastrulation, Pijuan-Sala et al., 2019 for GSE87038).
- The downloaded supplement contains both the expression matrix and the precomputed UMAP/tSNE cell-coordinate embeddings; if only one embedding is present the acquisition is incomplete.
- Cell identifiers (barcodes) in the embedding coordinate files are a subset of / identical to those in the expression matrix and cell-type annotations.
- The number of cells and cell types/clusters in the metadata matches the values the paper reports for Figures 2C and 2D.
- Feeding the downloaded embeddings into entropy_2dvs3d yields per-cell-type entropy values that can be tabulated for the 2D-vs-3D comparison.

## Limitations

- The paper lists only GEO accessions in its data-availability text; whether precomputed embedding coordinate files are actually included as GEO supplementary files must be verified per series, otherwise the 3D rotations cannot be reproduced as described.
- Only processed data are traced via GEO here — no raw-read reprocessing or QC is covered, so results depend on the authors' original processing choices.
- Backend R calculations assume the environment meets package requirements (cellexalvrR productively used under R 4.0.2 and 4.1.0); older R versions may fail during subsequent steps.

## Evidence

- [other] Download the mouse gastrulation scRNA-seq dataset (Pijuan-Sala et al., 2019, GEO accession GSE87038): [evidence span withheld — anchor status: not_found]
- [discussion] GEO GSE87038 for mouse gastrulation: "Mouse GastrulationPijuan-Sala et al. (2019)GEO GSE87038"
- [readme] README states the entropy tools compute Shannon index from 2d and 3d graphs: [evidence span withheld — anchor status: external_doc]
- [readme] cellexalvrR install command requires R environment: "devtools::install_github("sonejilab/cellexalvrR" )"
