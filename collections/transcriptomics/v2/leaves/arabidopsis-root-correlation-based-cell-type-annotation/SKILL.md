---
name: arabidopsis-root-correlation-based-cell-type-annotation
description: Use when quality-filtered Arabidopsis root scRNA-seq data (e.g. scKB/COPILOT
  output) needs cell type and developmental time annotations matched to the Shahan
  et al. 2022 root atlas.
when_to_use_negative:
- The dataset is not from the root of Arabidopsis thaliana — COPILOT annotation 'only
  supports cell type and developmental stage annotation for the root of Arabidopsis
  thaliana'
- The input is raw reads that have not yet been aligned and quality-filtered; run
  scKB alignment and COPILOT filtering first
- No reference atlas or bulk expression profile is available for the species/tissue
  of interest
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0780
tools:
- name: COPILOT
  role: Runs non-arbitrary quality filtering, doublet removal, Seurat analysis, and
    correlation-based Arabidopsis root annotation via do.annotation=TRUE
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: Seurat
  role: Provides the Seurat object format and the label transfer workflow (11-Label_Transfer)
    used to project celltype.anno and time.anno from the root atlas reference
- name: scKB
  role: Produces the raw spliced and unspliced counts matrices from scRNA-seq reads
    that serve as input to COPILOT before annotation
- name: JupyterLab
  role: Recommended web-based interface for running COPILOT in R (with IRkernel)
  repo: https://jupyterlab.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:3c9dfc2d3cf546531bcb7f3e972d2d569fb64d5b0656a7d2817dd4e23fc87f45
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
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# arabidopsis-root-correlation-based-cell-type-annotation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate Arabidopsis thaliana root single-cell clusters by correlating them against the Shahan & Hsu et al. (2022) root cell atlas (GSE152766), either via COPILOT's do.annotation=TRUE or Seurat label transfer of celltype.anno and time.anno. It leverages a curated reference expression profile so clusters receive cell type and developmental stage labels without manual marker inspection.

## When to use

Use when quality-filtered Arabidopsis root scRNA-seq data (e.g. scKB/COPILOT output) needs cell type and developmental time annotations matched to the Shahan et al. 2022 root atlas.

## When NOT to use

- The dataset is not from the root of Arabidopsis thaliana — COPILOT annotation 'only supports cell type and developmental stage annotation for the root of Arabidopsis thaliana'
- The input is raw reads that have not yet been aligned and quality-filtered; run scKB alignment and COPILOT filtering first
- No reference atlas or bulk expression profile is available for the species/tissue of interest

## Inputs

- COPILOT-processed Seurat object from quality-filtered Arabidopsis root scRNA-seq counts (e.g. raw spliced/unspliced gene-by-cell matrices from scKB, TAIR10 annotation)
- Root_Atlas_seu4.rds reference Seurat object from GSE152766 (Shahan & Hsu et al. 2022)
- Reference expression profile directory (dir_to_bulk) and color scheme file (dir_to_color_scheme) for COPILOT annotation

## Outputs

- Annotated Seurat object (e.g. col0_COPILOT.rds) with populated celltype.anno and time.anno metadata columns
- COPILOT summary file with feature plots colored by annotation

## How to apply

Run copilot() with do.annotation=TRUE (and dir_to_bulk pointing to the reference expression profile, dir_to_color_scheme for the color scheme), which performs Arabidopsis root correlation-based annotation; note this annotation support is limited to the root of Arabidopsis thaliana. For explicit label transfer, load the Root_Atlas_seu4.rds reference Seurat object (GSE152766, Shahan & Hsu et al. 2022) and execute the 11-Label_Transfer workflow to project celltype.anno and time.anno onto the query object (e.g. col0_COPILOT.rds), saving the annotated object afterwards. Optionally set unwanted.genes to protoplasting-induced and cell-cycle related genes before annotation so that artifact-driven expression does not bias correlations. Annotation typically follows quality filtering (mt.threshold=5, filtering.ratio=1), doublet removal (remove.doublet=TRUE), and Seurat normalization/PCA/UMAP/clustering (do.seurat=TRUE, clustering_alg=3, res=0.5).

## Related tools

- **COPILOT** (Runs non-arbitrary quality filtering, doublet removal, Seurat analysis, and correlation-based Arabidopsis root annotation via do.annotation=TRUE) — https://github.com/Hsu-Che-Wei/COPILOT
- **Seurat** (Provides the Seurat object format and the label transfer workflow (11-Label_Transfer) used to project celltype.anno and time.anno from the root atlas reference)
- **scKB** (Produces the raw spliced and unspliced counts matrices from scRNA-seq reads that serve as input to COPILOT before annotation)
- **JupyterLab** (Recommended web-based interface for running COPILOT in R (with IRkernel)) — https://jupyterlab.readthedocs.io/en/stable/

## Examples

```
copilot(sample.name="col0", mt.pattern="ATMG", cp.pattern="ATCG", mt.threshold=5, filtering.ratio=1, remove.doublet=TRUE, do.seurat=TRUE, do.annotation=TRUE, unwanted.genes=proto_genes, clustering_alg=3, res=0.5)
```

## Evaluation signals

- col0_COPILOT.rds exists, loads as a Seurat object, and contains celltype.anno and time.anno metadata columns populated for the quality-filtered, doublet-removed cells
- COPILOT summary file reports a plausible high-quality cell count after filtering and renders feature plots without errors
- Annotation support confirmed to be root-specific: query tissue must be Arabidopsis thaliana root
- Full col0 run completes in the expected ~60 min when label transfer is included (~50 min without)
- Cluster labels correspond to known root cell types/developmental stages from the Shahan et al. 2022 atlas reference

## Limitations

- The current version of COPILOT only supports cell type and developmental stage annotation for the root of Arabidopsis thaliana
- Required genome may not be available as a BSgenome object; converting a customized genome is needed
- X11 device unavailable in conda-installed R environments can cause COPILOT plotting failures for the summary file
- do.annotation defaults to FALSE, so annotation must be explicitly requested

## Evidence

- [discussion] COPILOT annotation limited to Arabidopsis root: "The current version of COPILOT only supports cell type and developmental stage annotation for the root of Arabidopsis thaliana."
- [readme] Reference expression profile argument: "dir_to_bulk`: Directory to reference expression profile for annotation. Default is NULL."
- [methods] do.annotation flag: "do.annotation`: Whether or not to do annotation (boolean). COPILOT only supports annotation on root of _Arabidopsis thaliana_. Default is FALSE."
- [full_text] Full col0 run with label transfer: [evidence span withheld — anchor status: not_found]
