---
name: seurat-normalization-dimensionality-reduction-clustering
description: Use when raw spliced/unspliced gene-by-cell UMI matrices from the scKB/kallisto
  pipeline need non-arbitrary QC followed by Seurat normalization, PCA, UMAP, and
  clustering, e.g. Arabidopsis root datasets.
when_to_use_negative:
- Input is already a quality-filtered, normalized Seurat object — COPILOT expects
  raw spliced/unspliced scKB count matrices as input
- Species other than Arabidopsis thaliana root when do.annotation=TRUE, since COPILOT
  annotation only supports the Arabidopsis root
- Downstream analysis requiring only CellRanger-style outputs without spliced/unspliced
  matrices for RNA velocity or trajectory inference
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: COPILOT
  role: Wraps quality filtering and invokes Seurat for normalization, PCA, UMAP and
    clustering via do.seurat=TRUE
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: Seurat
  role: R package (>= 3.1.4) performing normalization, PCA, UMAP and clustering on
    the quality-filtered matrices
- name: DoubletFinder
  role: R dependency used to remove doublets after quality filtering when remove.doublet=TRUE
- name: scKB
  role: kallisto/bustools wrapper producing the raw spliced and unspliced matrices
    that serve as COPILOT input
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:73ea8f3e7986c547ce67205f1ed674a34065a7d07a444948317a1eeac1a3b890
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
  - scKB
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# seurat-normalization-dimensionality-reduction-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs Seurat-based normalization, PCA, UMAP and clustering on COPILOT quality-filtered Arabidopsis root scRNA-seq count matrices (do.seurat=TRUE), optionally with correlation-based annotation, producing an analyzed Seurat object.

## When to use

Use when raw spliced/unspliced gene-by-cell UMI matrices from the scKB/kallisto pipeline need non-arbitrary QC followed by Seurat normalization, PCA, UMAP, and clustering, e.g. Arabidopsis root datasets.

## When NOT to use

- Input is already a quality-filtered, normalized Seurat object — COPILOT expects raw spliced/unspliced scKB count matrices as input
- Species other than Arabidopsis thaliana root when do.annotation=TRUE, since COPILOT annotation only supports the Arabidopsis root
- Downstream analysis requiring only CellRanger-style outputs without spliced/unspliced matrices for RNA velocity or trajectory inference

## Inputs

- Raw spliced gene-by-cell UMI count matrix from scKB (with row and column names)
- Raw unspliced gene-by-cell UMI count matrix from scKB (with row and column names)
- Sample name matching the scKB output directory
- Mitochondrial gene pattern (e.g. 'ATMG'; required argument)
- Optional: chloroplast gene pattern (e.g. 'ATCG'), unwanted gene list, annotation reference files (dir_to_bulk, dir_to_color_scheme)

## Outputs

- Spliced and unspliced quality-filtered count matrices (filtered.mtx.output.dir)
- Seurat object with normalization, PCA, UMAP embedding and clustering results
- COPILOT summary file (UMI histogram, gene histogram, barcode rank plot, feature plots)

## How to apply

Call the copilot() function on the raw spliced and unpliced count matrices produced by scKB, setting do.seurat=TRUE so that after non-arbitrary quality filtering (mt.pattern like 'ATMG' with mt.threshold=5, cp.pattern like 'ATCG', filtering.ratio=1, min.UMI.low.quality=100, min.UMI.high.quality=300) and DoubletFinder doublet removal (remove.doublet=TRUE, estimate.doublet.rate=TRUE) the function performs normalization, PCA, UMAP and clustering in Seurat (>= 3.1.4). Control clustering with clustering_alg=3 (SLM algorithm; 1=Louvain, 2=Louvain with multilevel refinement, 4=Leiden requiring leidenalg python) and res=0.5. Optionally pass unwanted.genes (e.g. protoplasting/cell-cycle related genes) and set do.annotation=TRUE for Arabidopsis root cell type annotation. Verify results via the COPILOT summary file (UMI histogram, gene histogram, barcode rank plot, feature plots), adjusting min.UMI.high.quality if the high-quality cell count is implausible. On the full col0 dataset this step plus annotation takes about 50 min, or about 60 min with label transfer.

## Related tools

- **COPILOT** (Wraps quality filtering and invokes Seurat for normalization, PCA, UMAP and clustering via do.seurat=TRUE) — https://github.com/Hsu-Che-Wei/COPILOT
- **Seurat** (R package (>= 3.1.4) performing normalization, PCA, UMAP and clustering on the quality-filtered matrices)
- **DoubletFinder** (R dependency used to remove doublets after quality filtering when remove.doublet=TRUE)
- **scKB** (kallisto/bustools wrapper producing the raw spliced and unspliced matrices that serve as COPILOT input)

## Examples

```
copilot(sample.name="col0", mt.pattern="ATMG", cp.pattern="ATCG", mt.threshold=5, filtering.ratio=1, remove.doublet=TRUE, estimate.doublet.rate=TRUE, do.seurat=TRUE, clustering_alg=3, res=0.5)
```

## Evaluation signals

- COPILOT summary file is generated with UMI histogram, gene histogram, barcode rank plot and feature plots
- Reported number of high-quality cells after filtering is plausible after inspecting the summary, adjusting min.UMI.high.quality if needed
- Output Seurat object exists, loads in R, and contains UMAP reductions and cluster assignments at res=0.5
- Spliced and unspliced quality-filtered matrices are written to filtered.mtx.output.dir with matching cell barcodes
- Runtime for the full col0 run is on the order of 50 min without annotation / 60 min with annotation

## Limitations

- COPILOT annotation (do.annotation=TRUE) only supports cell type and developmental stage annotation for the root of Arabidopsis thaliana
- COPILOT and its dependencies require R (>= 3.6.1), Seurat (>= 3.1.4), DoubletFinder, DropletUtils and the Python module umap-learn (leidenalg optional for Leiden clustering)
- Conda-installed R environments may lack the X11 device, causing failures when generating feature plots for the COPILOT summary file
- Leiden clustering (clustering_alg=4) requires the leidenalg Python module; HVG selection defaults to FALSE with HVGN=200

## Evidence
