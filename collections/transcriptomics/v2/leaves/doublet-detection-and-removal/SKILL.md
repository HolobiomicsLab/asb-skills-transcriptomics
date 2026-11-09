---
name: doublet-detection-and-removal
description: Use when raw spliced/unspliced gene-by-cell count matrices from the scKB
  pipeline are being quality-filtered with COPILOT and doublets must be detected and
  removed (remove.doublet=TRUE) before downstream Seurat analysis.
when_to_use_negative:
- The input is not a gene-by-cell UMI count matrix from scKB (e.g. unaligned FASTQ
  reads, which must first go through the scKB alignment/counting step)
- A custom doublet-removal strategy is required that replaces DoubletFinder, since
  COPILOT's doublet removal is fixed to this dependency and to 10X Genomics-style
  rate estimation
- Only annotation is needed for non-root Arabidopsis or other species, since COPILOT
  annotation is limited to the Arabidopsis thaliana root and this skill concerns doublet
  handling within that pipeline
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: COPILOT
  role: Hosts the doublet-removal step via the remove.doublet and estimate.doublet.rate
    arguments, executed after its non-arbitrary quality filtering
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: DoubletFinder
  role: R package dependency used by COPILOT to detect and remove doublets after quality
    filtering of gene and cell
- name: scKB
  role: Produces the raw spliced and unspliced count matrices that serve as the input
    on which doublet removal is performed
- name: Seurat
  role: Performs normalization, PCA, UMAP and clustering on the doublet-removed cells
    (do.seurat=TRUE)
provenance:
  source_task_ids:
  - task_007
  - task_005
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:758210138b661c7f213dd5eda4c485fccff61428cf70d4d383586de3671903cb
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# doublet-detection-and-removal

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies and removes doublets from gene-by-cell UMI count matrices as an integrated step of the COPILOT preprocessing pipeline, performed after non-arbitrary quality filtering and using DoubletFinder with either a 10X Genomics-style estimated doublet rate or a user-specified rate.

## When to use

Use when raw spliced/unspliced gene-by-cell count matrices from the scKB pipeline are being quality-filtered with COPILOT and doublets must be detected and removed (remove.doublet=TRUE) before downstream Seurat analysis.

## When NOT to use

- The input is not a gene-by-cell UMI count matrix from scKB (e.g. unaligned FASTQ reads, which must first go through the scKB alignment/counting step)
- A custom doublet-removal strategy is required that replaces DoubletFinder, since COPILOT's doublet removal is fixed to this dependency and to 10X Genomics-style rate estimation
- Only annotation is needed for non-root Arabidopsis or other species, since COPILOT annotation is limited to the Arabidopsis thaliana root and this skill concerns doublet handling within that pipeline

## Inputs

- Raw spliced gene-by-cell count matrix (with row and column names) produced by the scKB pipeline
- Raw unspliced gene-by-cell count matrix (with row and column names) produced by the scKB pipeline
- User-specified doublet rate (numeric, optional, default NULL; otherwise estimated per 10X Genomics' estimation)

## Outputs

- Quality-filtered, doublet-removed spliced and unspliced count matrices written to filtered.mtx.output.dir
- Seurat object of the retained cells (when do.seurat=TRUE), e.g. col0_COPILOT.rds after annotation and label transfer
- COPILOT summary file (HTML) with UMI histogram, barcode rank plot and feature plots

## How to apply

Within the copilot() call, set remove.doublet=TRUE so doublets are removed after the gene and cell quality filtering (mt.threshold=5 for mitochondrial/dying cells, filtering.ratio controlling stringency, top.percent=1 for high-UMI cells). Set estimate.doublet.rate=TRUE to let COPILOT estimate the doublet rate according to 10X Genomics' estimation, or supply an explicit doublet.rate as a numeric value instead. Doublet removal relies on the DoubletFinder R package, which is a declared dependency of COPILOT, and runs upstream of do.seurat=TRUE (normalization, PCA, UMAP, clustering). In the protocol's full col0 run, doublet removal is combined with do.seurat=TRUE, do.annotation=TRUE, unwanted.genes (protoplasting/cell-cycle genes), clustering_alg=3 and res=0.5; this complete run takes ~50 min without and ~60 min with label transfer, while the toy-data filtering takes ~2 min.

## Related tools

- **COPILOT** (Hosts the doublet-removal step via the remove.doublet and estimate.doublet.rate arguments, executed after its non-arbitrary quality filtering) — https://github.com/Hsu-Che-Wei/COPILOT
- **DoubletFinder** (R package dependency used by COPILOT to detect and remove doublets after quality filtering of gene and cell)
- **scKB** (Produces the raw spliced and unspliced count matrices that serve as the input on which doublet removal is performed)
- **Seurat** (Performs normalization, PCA, UMAP and clustering on the doublet-removed cells (do.seurat=TRUE))

## Examples

```
copilot(sample.name='col0', mt.pattern='ATMG', cp.pattern='ATCG', mt.threshold=5, filtering.ratio=1, remove.doublet=TRUE, estimate.doublet.rate=TRUE, do.seurat=TRUE)
```

## Evaluation signals

- The copilot() call with remove.doublet=TRUE completes without error and writes quality-filtered spliced/unspliced matrices to filtered.mtx.output.dir
- When estimate.doublet.rate=TRUE is used, the doublet rate applied matches 10X Genomics' estimation for the retained cell count (or the supplied doublet.rate value is respected)
- The output Seurat object contains only doublet-removed, quality-filtered cells and, in the full col0 protocol run, carries populated celltype.anno and time.anno metadata after label transfer
- The COPILOT summary file exists and reports a plausible high-quality cell count after filtering and doublet removal
- Recorded runtime is consistent with the protocol (~50 min without / ~60 min with label transfer for full col0; ~2 min for toy-data filtering)

## Limitations

- Doublet removal is performed only after quality filtering of gene and cell, so badly filtered input (e.g. wrong mt.pattern or min.UMI thresholds) degrades doublet detection
- The default doublet rate is not measured but estimated according to 10X Genomics' estimation; users wanting a different rate must pass doublet.rate explicitly
- COPILOT plotting (including summary feature plots) fails in conda-installed R environments lacking the x11 device
- In the protocol's full col0 run with doublet removal plus Seurat analysis and annotation, expect ~50–60 min wall-clock time

## Evidence

- [readme] R packages: Seurat (>= 3.1.4), Matrix, scales, r, gridExtra, ggplot2, DoubletFinder, DropletUtils, R2HTML: [evidence span withheld — anchor status: not_found]
- [results] Step 7 of the protocol runs COPILOT on the full col0 data with Seurat analysis, doublet removal, and annotation: [evidence span withheld — anchor status: not_found]
