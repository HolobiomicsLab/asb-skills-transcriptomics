---
name: single-cell-count-matrix-quality-control
description: Use when raw spliced/unspliced gene-by-cell count matrices (e.g. from
  scKB/kallisto-bustools) need quality filtering, doublet removal, and downstream
  Seurat analysis before cell-type annotation.
when_to_use_negative:
- Input matrices are already quality-filtered — COPILOT expects the raw spliced/unspliced
  counts matrices produced by the scKB pipeline
- The sample is not Arabidopsis thaliana root tissue and do.annotation=TRUE is desired,
  since COPILOT only supports cell type and developmental stage annotation for the
  Arabidopsis root
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: COPILOT
  role: performs non-arbitrary quality filtering, doublet removal, Seurat analysis,
    and Arabidopsis root annotation on the raw count matrices
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: scKB
  role: produces the raw spliced and unspliced counts matrices that are recommended
    as COPILOT input
- name: Seurat
  role: runs normalization, PCA, UMAP, clustering, and downstream label transfer on
    the filtered object
- name: DoubletFinder
  role: dependency used by COPILOT for doublet removal after quality filtering
- name: kallisto
  role: pseudoalignment engine wrapped by scKB/COPILOT, making processing ~30x faster
    than CellRanger 3.1
- name: JupyterLab
  role: recommended web-based interface for running COPILOT notebooks (with IRkernel)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:69d201874ca1d40566b500310abf0e48f9d79368c002f40d06705b9cd7b54b59
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ohlerlab/COPILOT
  repo_url: https://github.com/Hsu-Che-Wei/COPILOT
  tools:
  - COPILOT
  - DoubletFinder
  - JupyterLab
  - Seurat
  - kallisto
  - scKB
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# single-cell-count-matrix-quality-control

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Non-arbitrary quality control of single-cell UMI gene-by-cell count matrices with COPILOT, which filters low-quality cells, dying cells, and doublets by comparing distributions of low- and high-quality cells instead of arbitrary cutoffs, then optionally runs Seurat normalization/clustering and Arabidopsis root annotation.

## When to use

Use when raw spliced/unspliced gene-by-cell count matrices (e.g. from scKB/kallisto-bustools) need quality filtering, doublet removal, and downstream Seurat analysis before cell-type annotation.

## When NOT to use

- Input matrices are already quality-filtered — COPILOT expects the raw spliced/unspliced counts matrices produced by the scKB pipeline
- The sample is not Arabidopsis thaliana root tissue and do.annotation=TRUE is desired, since COPILOT only supports cell type and developmental stage annotation for the Arabidopsis root

## Inputs

- raw spliced gene-by-cell UMI count matrix from scKB (with row and column names)
- raw unspliced gene-by-cell UMI count matrix from scKB (with row and column names)
- optional sample directory named after sample.name containing the matrices
- mitochondrial gene pattern or gene list (e.g. 'ATMG')
- optional chloroplast gene pattern or gene list (e.g. 'ATCG')
- optional character vector of unwanted genes (cell-cycle, organelle, protoplasting genes)
- optional reference Seurat object for label transfer (Root_Atlas_seu4.rds, GSE152766)

## Outputs

- quality-filtered spliced and unspliced count matrices
- COPILOT summary file with UMI histogram, gene histogram, barcode rank plot, and feature plots
- Seurat object with PCA, UMAP, clustering, doublets removed, and optional celltype.anno / time.anno annotations (e.g. col0_COPILOT.rds)

## How to apply

Load the raw spliced and unspliced gene-by-cell UMI count matrices (folder named sample.name, or passed via spliced.mtx/unspliced.mtx) and call copilot() with mt.pattern (e.g. 'ATMG' for Arabidopsis, required argument), cp.pattern (e.g. 'ATCG'), mt.threshold=5 to remove dying cells with mitochondrial expression above 5%, filtering.ratio=1 (lenient; 0=strict) for distribution-based cell filtering, and min.UMI.low.quality=100 / min.UMI.high.quality=300 barcode thresholds. Set remove.doublet=TRUE with estimate.doublet.rate=TRUE (10X-style doublet rate, DoubletFinder-based) to remove doublets after gene/cell quality filtering. Set do.seurat=TRUE to run normalization, PCA, UMAP, and clustering (clustering_alg=3 SLM, res=0.5), do.annotation=TRUE for Arabidopsis root cell-type annotation, and unwanted.genes (e.g. protoplasting/cell-cycle genes) to filter noise genes from the analysis. Inspect the COPILOT summary file (UMI histogram, gene histogram, barcode rank plot, feature plots) to confirm the high-quality cell count is sensible, adjusting min.UMI.high.quality if needed; for the full col0 sample the run takes about 50 min without label transfer and 60 min with label transfer from the Shahan et al. 2022 root atlas (Root_Atlas_seu4.rds, GSE152766).

## Related tools

- **COPILOT** (performs non-arbitrary quality filtering, doublet removal, Seurat analysis, and Arabidopsis root annotation on the raw count matrices) — https://github.com/Hsu-Che-Wei/COPILOT
- **scKB** (produces the raw spliced and unspliced counts matrices that are recommended as COPILOT input)
- **Seurat** (runs normalization, PCA, UMAP, clustering, and downstream label transfer on the filtered object)
- **DoubletFinder** (dependency used by COPILOT for doublet removal after quality filtering)
- **kallisto** (pseudoalignment engine wrapped by scKB/COPILOT, making processing ~30x faster than CellRanger 3.1)
- **JupyterLab** (recommended web-based interface for running COPILOT notebooks (with IRkernel))

## Examples

```
copilot(sample.name="col0", mt.pattern="ATMG", cp.pattern="ATCG", mt.threshold=5, filtering.ratio=1, remove.doublet=TRUE, estimate.doublet.rate=TRUE, do.seurat=TRUE, clustering_alg=3, res=0.5)
```

## Evaluation signals

- COPILOT summary file is generated and its UMI histogram, gene histogram, barcode rank plot, and feature plots report a plausible number of high-quality cells after filtering; adjust min.UMI.high.quality (default 300) if not
- Filtered spliced and unspliced count matrices are written to filtered.mtx.output.dir and share consistent cell barcodes
- Output Seurat object loads in R and contains populated celltype.anno and time.anno metadata for the quality-filtered, doublet-removed cells
- No cells with mitochondrial expression percentage above mt.threshold=5 remain (dying cells removed)
- Runtime is in the reported range (~50 min without label transfer, ~60 min with label transfer for the full col0 dataset)

## Limitations

- COPILOT only supports cell type and developmental stage annotation for the root of Arabidopsis thaliana
- COPILOT plotting can fail in conda-installed R environments where the X11 device required to generate feature plots is unavailable
- Doublet removal relies on DoubletFinder with the 10X Genomics doublet-rate estimation; a user-specified doublet.rate is needed if the estimate is inappropriate
- R >= 3.6.1 plus Seurat >= 3.1.4, DoubletFinder, DropletUtils, and the Python module umap-learn are required (leidenalg only for Leiden clustering)

## Evidence

- [results] runtime finding: [evidence span withheld — anchor status: not_found]
