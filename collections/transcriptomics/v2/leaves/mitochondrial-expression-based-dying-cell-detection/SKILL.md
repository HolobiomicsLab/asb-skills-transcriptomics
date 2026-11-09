---
name: mitochondrial-expression-based-dying-cell-detection
description: Use when preprocessing raw spliced/unspliced gene-by-cell UMI count matrices
  (e.g. from scKB) and you need to flag dying cells via mitochondrial expression percentage
  before quality filtering.
when_to_use_negative:
- Input matrices are already quality-filtered or lack gene IDs matching a mitochondrial
  pattern (mt.pattern is required and must match your annotation)
- You need cell type annotation outside the root of Arabidopsis thaliana — COPILOT
  only supports Arabidopsis root annotation
- You want to keep cells with naturally high mitochondrial expression (e.g. metabolically
  active cell types) where a fixed mt.threshold=5 would wrongly classify them as dying
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: COPILOT
  role: Provides the copilot() function whose mt.pattern/mt.threshold arguments implement
    data-driven dying-cell detection via mitochondrial expression percentage
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: scKB
  role: Produces the raw spliced and unspliced count matrices recommended as COPILOT
    input
- name: R2HTML
  role: Generates the COPILOT HTML summary report containing cell-filtering statistics
- name: DropletUtils
  role: COPILOT dependency used for handling 10x-format droplet count matrices
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:2d007036f599530988af6191f5b7208b62e0533d8014f69395825e899b2867f8
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
  - DropletUtils
  - JupyterLab
  - R2HTML
  - scKB
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# mitochondrial-expression-based-dying-cell-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detect and remove dying cells in single-cell RNA-seq count matrices by thresholding mitochondrial expression percentage, using COPILOT's data-driven detection of cells enriched in mitochondrial expression rather than an arbitrary cutoff.

## When to use

Use when preprocessing raw spliced/unspliced gene-by-cell UMI count matrices (e.g. from scKB) and you need to flag dying cells via mitochondrial expression percentage before quality filtering.

## When NOT to use

- Input matrices are already quality-filtered or lack gene IDs matching a mitochondrial pattern (mt.pattern is required and must match your annotation)
- You need cell type annotation outside the root of Arabidopsis thaliana — COPILOT only supports Arabidopsis root annotation
- You want to keep cells with naturally high mitochondrial expression (e.g. metabolically active cell types) where a fixed mt.threshold=5 would wrongly classify them as dying

## Inputs

- Raw spliced gene-by-cell count matrix from scKB (with row and column names)
- Raw unspliced gene-by-cell count matrix from scKB (with row and column names)
- Mitochondrial gene name/id pattern or character vector (e.g. 'ATMG' for Arabidopsis)

## Outputs

- Cell-level dying-cell/low-quality classifications based on mitochondrial expression percentage
- Quality-filtered spliced and unspliced count matrices in 10x format (filtered.mtx.output.dir)
- COPILOT HTML summary report with cell statistics and UMI histogram plots

## How to apply

Call the exported copilot() function from the COPILOT R package with the required mt.pattern argument set to the mitochondrial gene name/id pattern of your species (e.g. 'ATMG' for Arabidopsis thaliana; a character vector of gene IDs is also accepted). COPILET's rationale is non-arbitrary filtering: it detects the distribution of cells enriched in mitochondrial expression to set the low-quality-cell filtering threshold, together with mt.threshold=5 (default), under which a cell with mitochondrial expression percentage above the threshold is treated as a dying cell. This runs alongside other filtering parameters — filtering.ratio (1 = lenient, 0 = strict), min.UMI.low.quality=100, min.UMI.high.quality=300, and top.percent=1 for high-UMI cells. Optionally supply cp.pattern (e.g. 'ATCG') to also account for chloroplast genes in plant data. The step feeds into production of quality-filtered spliced and unspliced count matrices and the COPILOT HTML summary report.

## Related tools

- **COPILOT** (Provides the copilot() function whose mt.pattern/mt.threshold arguments implement data-driven dying-cell detection via mitochondrial expression percentage) — https://github.com/Hsu-Che-Wei/COPILOT
- **scKB** (Produces the raw spliced and unspliced count matrices recommended as COPILOT input)
- **R2HTML** (Generates the COPILOT HTML summary report containing cell-filtering statistics)
- **DropletUtils** (COPILOT dependency used for handling 10x-format droplet count matrices)

## Examples

```
copilot(sample.name="col0", mt.pattern="ATMG", mt.threshold=5, cp.pattern="ATCG", filtering.ratio=1, remove.doublet=FALSE, do.seurat=FALSE, filtered.mtx.output.dir="filtered_out/")
```

## Evaluation signals

- The COPILOT HTML summary file exists in the output directory and contains cell-filtering statistics consistent with CellRanger 3.1-comparable summary output
- Quality-filtered spliced and unspliced matrices are present in filtered.mtx.output.dir in 10x format
- Dying cells flagged as having mitochondrial expression percentage higher than mt.threshold (default 5) are excluded from the filtered matrices
- The threshold reflects the detected distribution of cells enriched in mitochondrial expression, not an arbitrary user-set value
- Number of high-quality cells after filtering is sensible when inspected in the summary file (UMI histogram, barcode rank plot)

## Limitations

- mt.pattern is a required argument; it must match the gene naming of your transcriptome annotation (e.g. 'ATMG' only works for Arabidopsis-style TAIR10 IDs)
- mt.threshold=5 default is organism- and protocol-dependent; cells with legitimately high mitochondrial expression may be misclassified as dying
- X11 device unavailable in conda-installed R environments can cause COPILOT feature plot generation for the summary file to fail

## Evidence

- [full_text] data-driven threshold from workflow card: [evidence span withheld — anchor status: not_found]
