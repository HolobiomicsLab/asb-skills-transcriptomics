---
name: scrna-seq-preprocessing-workflow
description: Use when raw scRNA-seq reads (10X-style droplet data) must be aligned
  and converted to gene-by-cell UMI count matrices and then quality filtered non-arbitrarily,
  especially when spliced/unspliced matrices are needed for RNA velocity or trajectory
  inference.
when_to_use_negative:
- You only need alignment/counting and already have a quality-control strategy — Cell
  Ranger or another tool may suffice, though scKB is recommended when speed or spliced/unspliced
  matrices matter
- You need automated cell type or developmental stage annotation for a species other
  than the root of Arabidopsis thaliana (COPILOT annotation only supports Arabidopsis
  root)
- Your genome is not available as a BSgenome object and you have not converted it,
  since the workflow requires a BSgenome-compatible reference
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scKB
  role: Wrapper script of kallisto and bustools used for accelerated read alignment
    and generation of raw spliced/unspliced transcript count matrices, the recommended
    COPILOT input
- name: COPILOT
  role: Performs non-arbitrary quality filtering by comparing distributions of low-quality
    and high-quality cells; installed via devtools::install_github('Hsu-Che-Wei/COPILOT')
  repo: github.com/Hsu-Che-Wei/COPILOT
- name: kallisto
  role: Pseudoalignment engine underlying scKB/COPILOT, responsible for the ~30x speedup
    over CellRanger 3.1
- name: bustools
  role: Barcode/UMI correction and transcript counting back end invoked through the
    scKB wrapper
- name: Seurat
  role: Invoked by COPILOT (do.seurat=TRUE) to normalize, run PCA, UMAP and clustering
    on the quality-filtered cells
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:b359fd9be740ab804337fefe48725c7f5fbc759614e5c4fc6f24a09fc57de0e3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ohlerlab/COPILOT
  tools:
  - COPILOT
  - Seurat
  - bustools
  - kallisto
  - scKB
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scrna-seq-preprocessing-workflow

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A fast single-cell RNA-seq raw-data processing workflow that uses scKB (a kallisto/bustools wrapper) for alignment and transcript count matrix generation, followed by COPILOT for non-arbitrary quality control by comparing distributions of low-quality (mitochondria-enriched) and high-quality cells, producing spliced and unspliced quality-filtered count matrices.

## When to use

Use when raw scRNA-seq reads (10X-style droplet data) must be aligned and converted to gene-by-cell UMI count matrices and then quality filtered non-arbitrarily, especially when spliced/unspliced matrices are needed for RNA velocity or trajectory inference.

## When NOT to use

- You only need alignment/counting and already have a quality-control strategy — Cell Ranger or another tool may suffice, though scKB is recommended when speed or spliced/unspliced matrices matter
- You need automated cell type or developmental stage annotation for a species other than the root of Arabidopsis thaliana (COPILOT annotation only supports Arabidopsis root)
- Your genome is not available as a BSgenome object and you have not converted it, since the workflow requires a BSgenome-compatible reference

## Inputs

- Raw scRNA-seq FASTQ reads for scKB alignment against a transcriptome annotation (e.g. TAIR10)
- Raw spliced and unpliced counts matrices produced by the scKB pipeline (gene-by-cell UMI matrices with row and column names), stored in a directory matching the sample.name argument
- Mitochondrial gene-id pattern or gene list (e.g. 'ATMG' for Arabidopsis) for the required mt.pattern argument

## Outputs

- Quality-filtered spliced and unspliced count matrices written to filtered.mtx.output.dir
- COPILOT HTML summary file with UMI histogram, feature plots, and statistics (comparable to CellRanger 3.1 summary)
- Seurat object with normalization, PCA, UMAP and clustering results when do.seurat=TRUE

## How to apply

First align raw reads to the transcriptome (e.g. TAIR10 for Arabidopsis) via scKB, the kallisto/bustools wrapper, to produce raw spliced and unspliced counts matrices; scKB and COPILOT are ~30x faster than CellRanger 3.1. Then call copilot() with sample.name matching the directory containing the matrices, the required mt.pattern argument (e.g. 'ATMG' for Arabidopsis mitochondrial gene ids, used with mt.threshold=5 to detect dying cells), and UMI thresholds min.UMI.low.quality=100 and min.UMI.high.quality=300. COPILOT iteratively moves the filtering cut-off according to filtering.ratio (strict: 0; moderate: 0 < ratio < 1; lenient: 1) until no cells are more similar to the low-quality than the high-quality expression profile, and top.percent=1 removes cells with the highest UMI counts. Set remove.doublet=TRUE to estimate and remove doublets after quality filtering, and do.seurat=TRUE to run normalization, PCA, UMAP and clustering via Seurat on the filtered cells. Outputs include quality-filtered spliced/unspliced matrices in filtered.mtx.output.dir and an HTML summary file (UMI histogram, feature plots, statistics) directly comparable to CellRanger 3.1.

## Related tools

- **scKB** (Wrapper script of kallisto and bustools used for accelerated read alignment and generation of raw spliced/unspliced transcript count matrices, the recommended COPILOT input)
- **COPILOT** (Performs non-arbitrary quality filtering by comparing distributions of low-quality and high-quality cells; installed via devtools::install_github('Hsu-Che-Wei/COPILOT')) — github.com/Hsu-Che-Wei/COPILOT
- **kallisto** (Pseudoalignment engine underlying scKB/COPILOT, responsible for the ~30x speedup over CellRanger 3.1)
- **bustools** (Barcode/UMI correction and transcript counting back end invoked through the scKB wrapper)
- **Seurat** (Invoked by COPILOT (do.seurat=TRUE) to normalize, run PCA, UMAP and clustering on the quality-filtered cells)

## Examples

```
copilot(sample.name = "toy_data", mt.pattern = "ATMG", top.percent = 1, filtering.ratio = 0, remove.doublet = TRUE, do.seurat = TRUE)
```

## Evaluation signals

- The copilot() run completes without error (script_runs)
- The filtered spliced/unspliced matrix outputs in filtered.mtx.output.dir and the COPILOT summary HTML file exist (file_exists)
- On the col0 toy data with filtering.ratio=0, wall-clock runtime is within a few minutes of the reported ~2 min behavior (value_in_range)
- The COPILOT summary file contains a UMI histogram, feature plots, and statistics comparable to a CellRanger 3.1 summary
- Filtering iteration terminates because no remaining cells are more similar to the low-quality than the high-quality expression profile

## Limitations

- COPILOT only supports cell type and developmental stage annotation for the root of Arabidopsis thaliana
- The required genome may not be available as a BSgenome object, requiring users to convert a customized genome
- X11 device unavailable in conda-installed R environments can cause COPILOT plotting failures when generating feature plots for the summary file (JupyterLab with IRkernel is recommended)
- The protocol notes troubleshooting gaps for COPILOT dependency installation, scKB execution failures, and quality filtering start failures

## Evidence
