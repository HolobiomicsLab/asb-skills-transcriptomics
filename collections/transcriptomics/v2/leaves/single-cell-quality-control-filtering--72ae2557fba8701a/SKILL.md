---
name: single-cell-quality-control-filtering
description: Use when raw scRNA-seq count matrices from scKB (spliced/unspliced 10x-format
  matrices) need background-noise/dying-cell removal before downstream Seurat analysis,
  RNA velocity, or trajectory inference.
when_to_use_negative:
- When inputs are not raw spliced/unspliced UBIAS matrices from scKB — e.g. already-normalized
  or Cell Ranger-filtered feature matrices
- When the user needs cell type annotation for a species other than the root of Arabidopsis
  thaliana, since COPILOT annotation (do.annotation) supports only Arabidopsis root
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: COPILOT
  role: Performs the non-arbitrary quality filtering by comparing distributions of
    low-quality and high-quality cells; produces filtered matrices and HTML summary
  repo: github.com/Hsu-Che-Wei/COPILOT
- name: scKB
  role: Wrapper of kallisto/bustools that produces the raw spliced and unspliced count
    matrices recommended as COPILOT input
- name: Seurat
  role: Optional downstream normalization, PCA, UMAP and clustering of the filtered
    cells (do.seurat=TRUE)
- name: DoubletFinder
  role: Dependency underlying doublet rate estimation and doublet removal after quality
    filtering
- name: R2HTML
  role: Generates the COPILOT HTML summary report
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_004
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:63e8c26ad8123efb9b0d30ed36634457e72b54c83197a4027a6f77757641ee82
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ohlerlab/COPILOT
  tools:
  - BSgenome
  - COPILOT
  - DoubletFinder
  - DropletUtils
  - R2HTML
  - Seurat
  - bustools
  - scKB
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# single-cell-quality-control-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Non-arbitrary quality filtering of raw spliced/unspliced gene-by-cell UMI count matrices using COPILOT, which detects dying cells via mitochondrial expression and filters low-quality cells by comparing low- vs high-quality cell distributions instead of using arbitrary cutoffs.

## When to use

Use when raw scRNA-seq count matrices from scKB (spliced/unspliced 10x-format matrices) need background-noise/dying-cell removal before downstream Seurat analysis, RNA velocity, or trajectory inference.

## When NOT to use

- When inputs are not raw spliced/unspliced UBIAS matrices from scKB — e.g. already-normalized or Cell Ranger-filtered feature matrices
- When the user needs cell type annotation for a species other than the root of Arabidopsis thaliana, since COPILOT annotation (do.annotation) supports only Arabidopsis root

## Inputs

- Raw spliced gene-by-cell count matrix (10x mtx with row and column names) from scKB/kallisto-bustools
- Raw unspliced gene-by-cell count matrix (10x mtx with row and column names)
- Mitochondrial gene id pattern (e.g. 'ATMG') or character vector of mitochondrial gene names (e.g. parsed from chrM lines of genes.gtf)
- Optional chloroplast gene pattern (cp.pattern, e.g. 'ATCG') and unwanted.genes vector

## Outputs

- Quality-filtered spliced and unspliced count matrices written in 10x format to filtered.mtx.output.dir
- Optional Seurat object with normalization, PCA, UMAP, and clustering of filtered cells
- COPILOT HTML summary file with UMI histogram, filtering threshold, cell/sequencing statistics, and doublet removal results

## How to apply

Call the exported copilot() function with sample.name matching the scKB output directory (or pass spliced.mtx/unspliced.mtx directly) and supply the required mt.pattern (e.g. 'ATMG' for Arabidopsis, or a vector of mitochondrial genes grepped from chrM in genes.gtf for human GRCh38). COPILOT first sets the dying-cell threshold non-arbitrarily (mt.threshold default 5% mitochondrial expression) by detecting the distribution of mitochondria-enriched cells, then iterates the filtering cut-off according to filtering.ratio (lenient=1, strict=0, moderate between) until no cells are more similar to the low-quality than the high-quality expression profile. Barcode fallbacks apply via min.UMI.low.quality=100 and min.UMI.high.quality=300, and top.percent=1 removes the highest-UMI cells (putative doublets can be removed with remove.doublet=TRUE, optionally with estimate.doublet.rate=TRUE). Set do.seurat=TRUE to additionally normalize, run PCA/UMAP and clustering, and write filtered spliced/unspliced matrices plus an HTML summary (UMI histogram, feature plots, statistics) to filtered.mtx.output.dir.

## Related tools

- **COPILOT** (Performs the non-arbitrary quality filtering by comparing distributions of low-quality and high-quality cells; produces filtered matrices and HTML summary) — github.com/Hsu-Che-Wei/COPILOT
- **scKB** (Wrapper of kallisto/bustools that produces the raw spliced and unspliced count matrices recommended as COPILOT input)
- **Seurat** (Optional downstream normalization, PCA, UMAP and clustering of the filtered cells (do.seurat=TRUE))
- **DoubletFinder** (Dependency underlying doublet rate estimation and doublet removal after quality filtering)
- **R2HTML** (Generates the COPILOT HTML summary report)

## Examples

```
copilot(sample.name="pbmc_1k_v2", mt.pattern=mt_genes, filtering.ratio=1, top.percent=1, remove.doublet=TRUE, do.seurat=TRUE, filtered.mtx.output.dir="filtered_out/")
```

## Evaluation signals

- The copilot() run completes without error (script_runs)
- Filtered spliced and unspliced 10x matrices and the HTML summary file exist in filtered.mtx.output.dir (file_exists)
- The HTML summary contains expected cell-filtering statistics, UMI histogram, and threshold, comparable to a CellRanger 3.1 summary
- Number of retained high-quality cells is sensible given the min.UMI.high.quality fallback rule (default 300 UMIs)
- Runtime is within a few minutes of reported behavior (~2 min on the col0 toy data with filtering.ratio=0)

## Limitations

- COPILOT only supports cell type and developmental stage annotation for the root of Arabidopsis thaliana
- X11 device unavailability in conda-installed R environments can cause plotting failures when generating feature plots for the summary file
- Required genome may not be available as a BSgenome object, requiring conversion of customized genomes when building the kallisto index
- Doublet rate estimation relies on 10X Genomics' estimation unless a user-specified doublet.rate is given

## Evidence
