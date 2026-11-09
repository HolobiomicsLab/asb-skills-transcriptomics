---
name: single-cell-pathway-score-visualization
description: Use when a scRNA-seq dataset has been quality-controlled and cell-annotated
  and you need to quantify a signaling pathway (e.g., angiogenesis) per cell, rank
  cell types by score, and highlight pathway-high subpopulations in the tumor microenvironment.
when_to_use_negative:
- Input is raw unfiltered data — scWizard's default QC thresholds (<200 genes, >10,000
  genes, >5% mitochondrial reads) must be applied first, and low-quality cells would
  distort scores
- The gene set of interest is not defined and no GMT file or MSigDB pathway can be
  specified — GSVA requires a specific gene set or signaling pathway
- The goal is gene-level differential expression rather than gene-set-level pathway
  scoring per cell
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scWizard
  role: Point-and-click GUI that packages Input Data, Quality Control, the GSVA scoring
    module, and the Plot module used to score and visualize pathway activity per cell
  repo: https://github.com/Dulab2020/scWizard
- name: GSVA
  role: Method used inside scWizard's GSVA function to calculate per-cell scores for
    specific gene sets or signaling pathways
- name: GSVAdata
  role: Source of MSigDB gene sets (or user-provided GMT files) used to define the
    angiogenesis gene set for scoring
- name: Seurat
  role: Underlying standardized scRNA-seq pipeline in scWizard (dimension reduction,
    clustering, differential expression) supporting the per-cell-type aggregation
    of scores
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:fa5fbb6fbdd3b9f33142abc88bd2667ee6abc12c284162bd25055c06622f968c
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
yes_rate: 0.7
n_claims: 10
---

# single-cell-pathway-score-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute per-cell pathway activity scores (e.g., angiogenesis) on a QC-passed single-cell RNA-seq dataset using the GSVA module in scWizard with MSigDB gene sets or a user-supplied GMT file, then aggregate and visualize scores by cell type to flag pathway-high populations, as demonstrated in the scWizard ICC case study.

## When to use

Use when a scRNA-seq dataset has been quality-controlled and cell-annotated and you need to quantify a signaling pathway (e.g., angiogenesis) per cell, rank cell types by score, and highlight pathway-high subpopulations in the tumor microenvironment.

## When NOT to use

- Input is raw unfiltered data — scWizard's default QC thresholds (<200 genes, >10,000 genes, >5% mitochondrial reads) must be applied first, and low-quality cells would distort scores
- The gene set of interest is not defined and no GMT file or MSigDB pathway can be specified — GSVA requires a specific gene set or signaling pathway
- The goal is gene-level differential expression rather than gene-set-level pathway scoring per cell

## Inputs

- QC-passed scRNA-seq counts matrix or Seurat object (csv, rds, h5, or 10X folder)
- Gene set definition: MSigDB gene sets or user-uploaded GMT file
- Cell type annotations (e.g., from scWizard's hierarchical neural network annotation) for aggregating per-cell scores

## Outputs

- Per-cell GSVA pathway score table (e.g., gsva_angiogenesis_scores.csv)
- Per-cell-type aggregated score distributions ranking cell populations by pathway activity
- Visualizations (UMAP score overlay or per-cluster boxplots) exported via the Plot module

## How to apply

Load the scRNA-seq data into scWizard via the 'Input Data' module, accepting a Seurat object ('Data-rds') or a counts matrix (csv/rds/h5 as 'Data-counts') or a 10X folder ('Data-10X'). Apply scWizard's default quality control, filtering cells with <200 detected genes, >10,000 detected genes, or >5% mitochondrial read fraction before scoring. Run the 'GSVA' module, which calculates GSVA scores for specific gene sets or signaling pathways; use gene sets from the MSigDB database (as in the paper's angiogenesis score computation) or upload a custom GMT file, adjusting parameters in the GUI. Aggregate the per-cell scores by annotated cell type, export the score table (e.g., gsva_angiogenesis_scores.csv), and use the 'Plot' module for UMAP overlays or per-cluster boxplots to rank populations. In the ICC case study this workflow identified angiogenesis-high subpopulations (Macro_NLRP3, Mono_CD14 myeloid cells and Arteries-like endothelial cells), so correctness is judged by whether the per-cell-type score distribution shows a distinct pathway-high population.

## Related tools

- **scWizard** (Point-and-click GUI that packages Input Data, Quality Control, the GSVA scoring module, and the Plot module used to score and visualize pathway activity per cell) — https://github.com/Dulab2020/scWizard
- **GSVA** (Method used inside scWizard's GSVA function to calculate per-cell scores for specific gene sets or signaling pathways)
- **GSVAdata** (Source of MSigDB gene sets (or user-provided GMT files) used to define the angiogenesis gene set for scoring)
- **Seurat** (Underlying standardized scRNA-seq pipeline in scWizard (dimension reduction, clustering, differential expression) supporting the per-cell-type aggregation of scores)

## Examples

```
if (!requireNamespace("devtools", quietly=TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- GSVA scores are computed for every cell in the QC-passed dataset (no cells dropped by the scoring module after the <200 / >10,000 genes / >5% mitochondrial filters)
- The per-cell-type score distribution highlights a distinct pathway-high population consistent with the scWizard ICC case study (Macro_NLRP3 and Mono_CD14 myeloid cells, Arteries-like endothelial cells highest for angiogenesis)
- The GMT gene set actually loaded matches the intended MSigDB pathway (correct gene identifiers, rownames aligned to the counts matrix)
- Exported score table (gsva_angiogenesis_scores.csv) contains one score per QC-passed cell plus the cell type labels used for aggregation
- Plot module output renders scores across all annotated populations without missing/NaN categories

## Limitations

- GSVA scoring in scWizard requires a pre-defined gene set; custom pathways must be supplied as a GMT file uploaded by the user
- Tissue-specific cell types may fall into an Unknown cluster in scWizard's annotation, which would obscure cell-type-resolved pathway-high populations
- scWizard's annotation model targets 7 major cell types and 47 TME subtypes across 13 cancer types; pathways or cell types outside this scope may require custom training data or manual analysis

## Evidence
