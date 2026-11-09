---
name: single-cell-qc-summary-reporting
description: Use when raw scRNA-seq count matrices (spliced and unspliced from scKB/kallisto/bustools)
  have been quality-filtered with COPILOT and you need to report or verify cell-filtering,
  mitochondrial thresholds, and doublet removal statistics in a CellRanger-3.1-comparable
  summary.
when_to_use_negative:
- Input is already a filtered/processed count matrix or Seurat object rather than
  the raw spliced/unspliced matrices produced by the scKB pipeline
- The goal is cell type or developmental stage annotation for a species other than
  the root of Arabidopsis thaliana — COPILOT only supports annotation for Arabidopsis
  root
- Plotting features from the summary fails in conda-installed R environments lacking
  the X11 device required for feature plots
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: COPILOT
  role: Runs non-arbitrary quality filtering and generates the summary file (UMI histogram,
    thresholds, cell/sequencing statistics, doublet results)
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: scKB
  role: Wrapper of kallisto and bustools producing the raw spliced and unspliced matrices
    recommended as COPILOT summary input
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: Seurat
  role: Performs normalization, PCA, UMAP and clustering whose feature plots appear
    in the COPILOT summary file
- name: DoubletFinder
  role: Dependency used for doublet removal statistics reported in the summary
- name: JupyterLab
  role: Recommended web-based interface for running COPILOT and viewing the summary
    file
  repo: https://jupyterlab.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:456ace05622e97473ad75a47912f30827b7668d9d773f531ac9c35509011a7f3
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
  - BSgenome
  - COPILOT
  - DoubletFinder
  - JupyterLab
  - Seurat
  - bustools
  - scKB
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# single-cell-qc-summary-reporting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate and inspect the COPILOT summary file — a QC report containing the UMI histogram, filtering thresholds, cell/sequencing statistics, and doublet removal results — after non-arbitrary quality filtering of raw spliced/unspliced gene-by-cell matrices from the scKB pipeline.

## When to use

Use when raw scRNA-seq count matrices (spliced and unspliced from scKB/kallisto/bustools) have been quality-filtered with COPILOT and you need to report or verify cell-filtering, mitochondrial thresholds, and doublet removal statistics in a CellRanger-3.1-comparable summary.

## When NOT to use

- Input is already a filtered/processed count matrix or Seurat object rather than the raw spliced/unspliced matrices produced by the scKB pipeline
- The goal is cell type or developmental stage annotation for a species other than the root of Arabidopsis thaliana — COPILOT only supports annotation for Arabidopsis root
- Plotting features from the summary fails in conda-installed R environments lacking the X11 device required for feature plots

## Inputs

- Raw spliced gene-by-cell UMI count matrix (from scKB/kallisto/bustools)
- Raw unspliced gene-by-cell UMI count matrix
- List or pattern of mitochondrial gene names/ids (mt.pattern, e.g. from grepping chrM in genes.gtf)
- Sample name matching the scKB output directory containing the matrices

## Outputs

- COPILOT summary file (HTML) with UMI histogram, filtering threshold, cell/sequencing statistics, doublet removal results, and feature plots
- Quality-filtered spliced and unspliced count matrices written to filtered.mtx.output.dir
- Seurat object with normalization, PCA, UMAP and clustering (when do.seurat=TRUE)

## How to apply

Run the copilot() function on the raw spliced and unspliced matrices produced by scKB, supplying mt.pattern as a mitochondrial gene list (e.g. extracted by grepping chrM from genes.gtf for human GRCh38) and keeping defaults such as mt.threshold=5, filtering.ratio=1 (lenient), top.percent=1, min.UMI.low.quality=100, min.UMI.high.quality=300, estimate.doublet.rate=TRUE, remove.doublet=TRUE, and do.seurat=TRUE. COPILOT automatically renders an HTML summary file containing the UMI histogram, the selected filtering threshold, cell/sequencing statistics, doublet removal results, and Seurat-based feature plots. Inspect the summary to judge whether the retained high-quality cell count is sensible given the min.UMI.high.quality fallback rule, and confirm the threshold selected by comparing low-quality vs high-quality cell distributions is not an arbitrary cutoff. Note the summary file's utility is stated to be directly comparable to CellRanger 3.1, so it can serve as a drop-in QC report alongside that tool's outputs.

## Related tools

- **COPILOT** (Runs non-arbitrary quality filtering and generates the summary file (UMI histogram, thresholds, cell/sequencing statistics, doublet results)) — https://github.com/Hsu-Che-Wei/COPILOT
- **scKB** (Wrapper of kallisto and bustools producing the raw spliced and unspliced matrices recommended as COPILOT summary input) — https://github.com/Hsu-Che-Wei/COPILOT
- **Seurat** (Performs normalization, PCA, UMAP and clustering whose feature plots appear in the COPILOT summary file)
- **DoubletFinder** (Dependency used for doublet removal statistics reported in the summary)
- **JupyterLab** (Recommended web-based interface for running COPILOT and viewing the summary file) — https://jupyterlab.readthedocs.io/en/stable/

## Examples

```
copilot(sample.name="pbmc_1k_v2", mt.pattern=mt_genes, mt.threshold=5, filtering.ratio=1, top.percent=1, min.UMI.low.quality=100, min.UMI.high.quality=300, estimate.doublet.rate=TRUE, remove.doublet=TRUE, do.seurat=TRUE, do.annotation=FALSE, filtered.mtx.output.dir="pbmc_1k_v2_filtered")
```

## Evaluation signals

- The COPILOT summary file is generated and contains a UMI histogram, a selected filtering threshold, cell/sequencing statistics, and doublet removal results
- The number of retained high-quality cells is sensible given the min.UMI.high.quality=300 fallback rule (and min.UMI.low.quality=100 barcode threshold)
- Cells flagged as dying have mitochondrial expression percentage above the mt.threshold=5 default, confirming mt.pattern was supplied correctly
- Filtered spliced and unspliced matrices exist in filtered.mtx.output.dir with consistent cell barcodes
- Summary metrics are directly comparable to a CellRanger 3.1 run on the same sample as a cross-check

## Limitations

- COPILOT only supports cell type and developmental stage annotation for the root of Arabidopsis thaliana; annotation must be disabled (do.annotation=FALSE) for other species such as human PBMC data
- X11 device unavailability in conda-installed R environments causes COPILOT plotting failures for summary-file feature plots
- The required genome may not be available as a BSgenome object, requiring conversion of customized genomes before building the index

## Evidence

- [readme] COPILOT summary file utility comparable to CellRanger: [evidence span withheld — anchor status: external_doc]
