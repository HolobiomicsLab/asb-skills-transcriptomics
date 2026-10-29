---
name: reference-atlas-label-transfer
description: Use when a COPILOT-processed Arabidopsis root scRNA-seq Seurat object
  needs cell type and developmental stage labels without manual annotation, by leveraging
  the published root atlas reference.
when_to_use_negative:
- The organism/tissue is not Arabidopsis thaliana root — COPILOT annotation 'only
  supports cell type and developmental stage annotation for the root of Arabidopsis
  thaliana'
- The query matrix has not been quality filtered (raw droplet matrix); label transfer
  should follow COPILOT non-arbitrary filtering and doublet removal
- A customized genome/transcriptome is required but unavailable as a BSgenome object
  and the user cannot convert it
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: COPILOT
  role: Performs non-arbitrary quality filtering, doublet removal, Seurat analysis
    and correlation-based annotation against the Arabidopsis root reference before
    label transfer
  repo: https://github.com/Hsu-Che-Wei/COPILOT
- name: Seurat
  role: Provides the Seurat object format for query and reference (Root_Atlas_seu4.rds)
    and the anchoring framework used in the 11-Label_Transfer workflow
- name: scKB
  role: Generates the raw spliced and unspliced gene-by-cell count matrices used as
    COPILOT input prior to annotation transfer
- name: JupyterLab
  role: Recommended web-based interface in which the 11-Label_Transfer workflow is
    executed
  repo: https://jupyterlab.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xpro.2022.101729
    title: Protocol for fast scRNA-seq raw data processing using scKB and non-arbitrary
      quality control with COPILOT
  source_content_hash: sha256:2629982c349f3178c248b2314a1b4a81cb4a3410a9dbc74b177d7b48efa1f288
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# reference-atlas-label-transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Transfer cell type (celltype.anno) and developmental time (time.anno) annotations from the Shahan et al. (2022) Arabidopsis root atlas reference (GSE152766, Root_Atlas_seu4.rds) onto a COPILOT quality-filtered, doublet-removed Seurat object using the protocol's 11-Label_Transfer workflow, adding ~10 min to the ~50 min full col0 run.

## When to use

Use when a COPILOT-processed Arabidopsis root scRNA-seq Seurat object needs cell type and developmental stage labels without manual annotation, by leveraging the published root atlas reference.

## When NOT to use

- The organism/tissue is not Arabidopsis thaliana root — COPILOT annotation 'only supports cell type and developmental stage annotation for the root of Arabidopsis thaliana'
- The query matrix has not been quality filtered (raw droplet matrix); label transfer should follow COPILOT non-arbitrary filtering and doublet removal
- A customized genome/transcriptome is required but unavailable as a BSgenome object and the user cannot convert it

## Inputs

- Raw spliced and unspliced gene-by-cell count matrices from the scKB pipeline (column/row-named Matrix objects)
- COPILOT-processed Seurat object of quality-filtered, doublet-removed cells
- Root_Atlas_seu4.rds reference Seurat object (GSE152766, Shahan & Hsu et al. 2022 Arabidopsis root atlas)
- Reference expression profile directory (dir_to_bulk) and color scheme file directory (dir_to_color_scheme) for annotation

## Outputs

- Annotated Seurat object (e.g. col0_COPILOT.rds) with celltype.anno and time.anno metadata columns populated for all high-quality cells
- COPILOT summary file (HTML with UMI histogram, gene histogram, barcode rank plot, feature plots) including annotation plots

## How to apply

First run copilot() on the raw spliced/unspliced scKB count matrices (mt.pattern='ATMG', cp.pattern='ATCG', mt.threshold=5, filtering.ratio=1, remove.doublet=TRUE, do.seurat=TRUE) to obtain a filtered Seurat object of high-quality cells. Load the Root_Atlas_seu4.rds reference Seurat object (GSE152766, Shahan & Hsu et al. 2022) and execute the protocol's 11-Label_Transfer workflow in JupyterLab to transfer both celltype.anno and time.anno metadata columns onto the query object. COPILOT's built-in annotation uses correlation against a reference expression profile supplied via dir_to_bulk and a color scheme via dir_to_color_scheme, and supports only Arabidopsis thaliana root. Save the result (e.g. col0_COPILOT.rds) and verify celltype.anno and time.anno are populated for all quality-filtered, doublet-removed cells.

## Related tools

- **COPILOT** (Performs non-arbitrary quality filtering, doublet removal, Seurat analysis and correlation-based annotation against the Arabidopsis root reference before label transfer) — https://github.com/Hsu-Che-Wei/COPILOT
- **Seurat** (Provides the Seurat object format for query and reference (Root_Atlas_seu4.rds) and the anchoring framework used in the 11-Label_Transfer workflow)
- **scKB** (Generates the raw spliced and unspliced gene-by-cell count matrices used as COPILOT input prior to annotation transfer)
- **JupyterLab** (Recommended web-based interface in which the 11-Label_Transfer workflow is executed) — https://jupyterlab.readthedocs.io/en/stable/

## Examples

```
copilot(sample.name="col0", mt.pattern="ATMG", cp.pattern="ATCG", mt.threshold=5, filtering.ratio=1, remove.doublet=TRUE, do.seurat=TRUE, do.annotation=TRUE, unwanted.genes=protoplasting_genes, clustering_alg=3, res=0.5)
```

## Evaluation signals

- col0_COPILOT.rds exists and loads as a Seurat object
- celltype.anno and time.anno metadata columns are populated (non-NA) for all quality-filtered, doublet-removed cells
- The COPILOT summary file reports a plausible high-quality cell count after filtering; adjust min.UMI.high.quality (default 300) if not sensible
- Runtime is consistent with the protocol: ~50 min without label transfer and ~60 min with label transfer for the full col0 dataset
- Annotation labels are restricted to Arabidopsis root cell types and developmental stages defined by the Shahan et al. (2022) atlas color scheme

## Limitations

- COPILOT annotation supports only the root of Arabidopsis thaliana for cell type and developmental stage labels
- X11 device unavailable in conda-installed R environments causes feature-plot generation in the COPILOT summary file to fail
- Protoplasting-induced genes must be removed via unwanted.genes before clustering/annotation to avoid protoplasting artifacts

## Evidence

- [results] Label transfer from Shahan et al. 2022 atlas: [evidence span withheld — anchor status: not_found]
