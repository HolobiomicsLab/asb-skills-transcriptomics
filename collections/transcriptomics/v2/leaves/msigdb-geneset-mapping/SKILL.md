---
name: msigdb-geneset-mapping
description: Use when scoring per-cell pathway activity (e.g., angiogenesis) from
  a scRNA-seq counts matrix or Seurat object using GSVA with MSigDB gene sets or a
  user-uploaded GMT file, to flag cell types with elevated pathway activity.
when_to_use_negative:
- Input is already a per-cell score or feature table rather than raw counts or a Seurat
  object — GSVA needs the expression matrix
- The question concerns cell-type annotation or clustering rather than pathway-level
  scoring (use scWizard's neural-network annotation or Seurat clustering modules instead)
- The pathway of interest is not in MSigDB and no GMT gene set file is available for
  upload
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scWizard
  role: GUI/R package providing the 'Input Data', 'Quality Control', and 'GSVA' modules
    used to load data, filter cells, and compute per-cell GSVA scores
  repo: https://github.com/Dulab2020/scWizard
- name: GSVA
  role: Method underlying scWizard's 'GSVA' function for calculating pathway scores
    for specific gene sets or signaling pathways
- name: GSVAdata
  role: Optional resource for supplying user-provided GMT gene set files uploaded
    into the GSVA module
- name: Seurat
  role: Standardized scRNA-seq pipeline (dimension reduction, clustering, differential
    expression) underlying scWizard, providing the cell-type annotations used to aggregate
    scores
- name: MSigDB
  role: Source of gene sets (e.g., angiogenesis signature) used to compute pathway
    scores
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:076ecb70a4308fcb896b0dc267369c9bcea5cbd04e8d694049aa390f0ad3dcc8
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Dulab2020/scWizard
  repo_url: https://github.com/Dulab2020/scWizard
  tools:
  - GSVA
  - GSVAdata
  - MSigDB
  - Seurat
  - scWizard
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# msigdb-geneset-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Score single cells against MSigDB-derived gene sets using scWizard's GSVA module, as demonstrated by computing angiogenesis pathway scores on the ICC scRNA-seq dataset to identify angiogenesis-high cell populations.

## When to use

Use when scoring per-cell pathway activity (e.g., angiogenesis) from a scRNA-seq counts matrix or Seurat object using GSVA with MSigDB gene sets or a user-uploaded GMT file, to flag cell types with elevated pathway activity.

## When NOT to use

- Input is already a per-cell score or feature table rather than raw counts or a Seurat object — GSVA needs the expression matrix
- The question concerns cell-type annotation or clustering rather than pathway-level scoring (use scWizard's neural-network annotation or Seurat clustering modules instead)
- The pathway of interest is not in MSigDB and no GMT gene set file is available for upload

## Inputs

- Seurat object (Data-rds) or counts matrix in csv/rds/h5 format (genes as rows, cells as columns) or a 10X data folder
- QC-passed single-cell expression matrix (cells with 200–10,000 detected genes and ≤5% mitochondrial reads)
- MSigDB gene set collection, or a user-supplied GMT gene set file

## Outputs

- Per-cell GSVA pathway scores for every QC-passed cell (e.g., gsva_angiogenesis_scores.csv)
- Per-cell-type aggregated score table ranking cell populations by pathway activity
- Visualizations (e.g., UMAP overlay or per-cluster boxplot) from the Plot module

## How to apply

Load the scRNA-seq data into scWizard via the 'Input Data' module, accepting a Seurat object (Data-rds), a counts matrix (csv/rds/h5), or a 10X folder. Apply scWizard's default quality control, which filters cells with <200 detected genes, >10,000 detected genes, or >5% mitochondrial reads. In the 'GSVA' module, score cells with the pathway gene set (e.g., angiogenesis) using gene sets from the MSigDB database; alternatively upload a user-provided GMT gene set file and adjust parameters. The angiogenesis gene set used in the ICC case study was applied via the GSVA method with MSigDB gene sets. Aggregate per-cell scores by annotated cell type, save the results (the GSVA module supports saving results, e.g., gsva_angiogenesis_scores.csv), and use the Plot module to visualize score distributions (UMAP overlay or per-cluster boxplots) and rank cell types to flag pathway-high populations.

## Related tools

- **scWizard** (GUI/R package providing the 'Input Data', 'Quality Control', and 'GSVA' modules used to load data, filter cells, and compute per-cell GSVA scores) — https://github.com/Dulab2020/scWizard
- **GSVA** (Method underlying scWizard's 'GSVA' function for calculating pathway scores for specific gene sets or signaling pathways)
- **GSVAdata** (Optional resource for supplying user-provided GMT gene set files uploaded into the GSVA module)
- **Seurat** (Standardized scRNA-seq pipeline (dimension reduction, clustering, differential expression) underlying scWizard, providing the cell-type annotations used to aggregate scores)
- **MSigDB** (Source of gene sets (e.g., angiogenesis signature) used to compute pathway scores)

## Examples

```
devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()  # then use Input Data -> Quality Control -> GSVA module with MSigDB angiogenesis set or uploaded GMT
```

## Evaluation signals

- GSVA scores are computed for every cell in the QC-passed dataset (no cells dropped after filtering at <200 genes, >10,000 genes, or >5% mitochondrial reads)
- Per-cell-type score distribution highlights a distinct angiogenesis-high population consistent with the scWizard ICC case study
- In the ICC data, Macro_NLRP3 and Mono_CD14 among myeloid cells and Arteries-like cells among endothelial cells show the highest angiogenesis scores
- Saved result file (e.g., gsva_angiogenesis_scores.csv) contains per-cell scores keyed to annotated cell types and can be re-loaded in the Plot module
- Scored pathway corresponds to the intended MSigDB set (or the uploaded GMT) rather than a default/unspecified gene set

## Limitations

- GMT files must be supplied by the user if the pathway is not covered by MSigDB; scWizard's GSVA module supports but does not bundle custom gene sets
- GSVA is one of several downstream modules and depends on prior quality control and cell-type annotation to make per-type rankings interpretable
- The GSVA module's parameters can be adjusted but the article does not document defaults beyond the method and gene-set source

## Evidence
