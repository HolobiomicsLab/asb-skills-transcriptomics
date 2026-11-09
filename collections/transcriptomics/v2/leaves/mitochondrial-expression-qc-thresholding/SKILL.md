---
name: mitochondrial-expression-qc-thresholding
description: Use when preprocessing raw scRNA-sev spliced/unspliced count matrices
  (e.g. from scKB) and a data-driven dying-cell cutoff based on mitochondrial read
  percentage is needed before downstream filtering.
when_to_use_negative:
- Input matrices are already quality-filtered, so no dying-cell or low-quality population
  exists to threshold
- The analysis requires arbitrary/manual UMI cutoffs instead of distribution-based,
  non-arbitrary filtering
- Data are not UMI count gene-by-cell matrices from a droplet scRNA-seq pipeline such
  as scKB
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: COPILOT
  role: Computes per-cell mitochondrial expression percentage from mt.pattern-matched
    genes, applies mt.threshold dying-cell cutoff, then performs non-arbitrary iterative
    cell filtering
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: scKB
  role: Produces the raw spliced and unspliced counts matrices recommended as COPILOT
    input
- name: Seurat
  role: Normalizes filtered cells and runs PCA, UMAP and clustering when do.seurat=TRUE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:30729f8de428f08b13f012c186c63d16ec0c1ad47e85c675e36ba3930594b7d6
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
  - Seurat
  - scKB
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# mitochondrial-expression-qc-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sets a mitochondrial expression-percentage cutoff to flag dying cells as the first stage of COPILOT's non-arbitrary single-cell quality control on scKB-produced spliced/unspliced UBI matrices, before iterative low-vs-high-quality cell filtering.

## When to use

Use when preprocessing raw scRNA-sev spliced/unspliced count matrices (e.g. from scKB) and a data-driven dying-cell cutoff based on mitochondrial read percentage is needed before downstream filtering.

## When NOT to use

- Input matrices are already quality-filtered, so no dying-cell or low-quality population exists to threshold
- The analysis requires arbitrary/manual UMI cutoffs instead of distribution-based, non-arbitrary filtering
- Data are not UMI count gene-by-cell matrices from a droplet scRNA-seq pipeline such as scKB

## Inputs

- Raw gene-by-cell spliced counts matrix from scKB (with row and column names)
- Raw gene-by-cell unspliced counts matrix from scKB (with row and column names)
- Directory named to match the sample.name argument containing the matrices
- mt.pattern: mitochondrial gene-id pattern (e.g. 'ATMG') or character vector of mitochondrial genes

## Outputs

- Quality-filtered spliced/unspliced count matrices written to filtered.mtx.output.dir
- Seurat object with normalization, PCA, UMAP and clustering (when do.seurat=TRUE)
- COPILOT HTML summary file with UMI histogram, feature plots and statistics

## How to apply

Call copilot() on the raw scKB spliced/unspliced matrices, supplying mt.pattern (e.g. 'ATMG' for Arabidopsis mitochondrial gene ids) — this argument is required because COPILOT uses it to compute each cell's mitochondrial expression percentage. Cells with mitochondrial expression percentage above mt.threshold (default 5) are treated as dying cells and removed. This thresholding step initializes the two distributions that COPILOT then compares — low-quality (mitochondria-enriched) vs high-quality cells (min.UMI.low.quality=100, min.UMI.high.quality=300) — iterating the filtering cut-off per filtering.ratio (0 = strict, 1 = lenient) until no cells remain more similar to the low-quality profile. After filtering, top.percent (default 1) removes high-UMI cells and remove.doublet=TRUE triggers doublet removal; with do.seurat=TRUE the filtered cells are normalized and embedded via Seurat.

## Related tools

- **COPILOT** (Computes per-cell mitochondrial expression percentage from mt.pattern-matched genes, applies mt.threshold dying-cell cutoff, then performs non-arbitrary iterative cell filtering) — https://github.com/Hsu-Che-Wei/COPILOT
- **scKB** (Produces the raw spliced and unspliced counts matrices recommended as COPILOT input)
- **Seurat** (Normalizes filtered cells and runs PCA, UMAP and clustering when do.seurat=TRUE)

## Examples

```
copilot(sample.name="col0_toy", mt.pattern="ATMG", mt.threshold=5, filtering.ratio=0, top.percent=1, remove.doublet=TRUE, do.seurat=TRUE)
```

## Evaluation signals

- copilot() completes without error, confirming mt.pattern was supplied and matched mitochondrial genes
- Cells with mitochondrial expression percentage above mt.threshold (default 5) are absent from the filtered matrix
- Iterative filtering terminates (no cells remain more similar to the low-quality than the high-quality expression profile), per filtering.ratio semantics
- filtered.mtx.output.dir contains quality-filtered spliced and unspliced matrices and the COPILOT summary HTML file exists
- On the col0 toy data with filtering.ratio=0, wall-clock runtime of the copilot() call is on the order of ~2 minutes

## Limitations

- COPILOT's cell type/developmental stage annotation (do.annotation) only supports the root of Arabidopsis thaliana; other species can only use the filtering steps
- X11 device unavailability in conda-installed R environments can cause COPILOT plotting failures when generating feature plots for the summary file
- mt.pattern is required and organism-specific (e.g. 'ATMG' for Arabidopsis), so the correct mitochondrial gene-id prefix must be known per genome

## Evidence
