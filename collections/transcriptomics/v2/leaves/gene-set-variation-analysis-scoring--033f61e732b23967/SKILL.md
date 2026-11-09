---
name: gene-set-variation-analysis-scoring
description: Use when scRNA-seq cells need pathway-level activity scores (e.g. MSigDB
  angiogenesis or a user GMT gene set) to flag pathway-high subpopulations within
  a tumor microenvironment.
when_to_use_negative:
- The analysis goal is marker-gene differential expression or cell-type annotation
  rather than pathway-level activity scoring — use scWizard's Seurat clustering or
  neural-network annotation modules instead.
- No predefined gene set exists (no MSigDB set or GMT file) and the pathway signature
  must be discovered de novo from the data.
- The input is already a per-cell or per-cluster feature/score table rather than raw
  or QC-filtered single-cell expression data.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scWizard
  role: Point-and-click Shiny GUI whose 'GSVA' module computes per-cell pathway scores
    on QC-passed single-cell data and displays/saves results; the Plot module visualizes
    score distributions.
  repo: https://github.com/Dulab2020/scWizard
- name: GSVA
  role: The underlying gene set variation analysis method used by scWizard to calculate
    pathway scores (e.g. angiogenesis score) per cell.
- name: GSVAdata
  role: Resource from which user-provided GMT gene set files can be supplied to the
    scWizard GSVA module as an alternative to MSigDB sets.
- name: Seurat
  role: Underlies scWizard's standardized pipeline (dimension reduction, clustering,
    differential expression) that defines the annotated cell populations the GSVA
    scores are aggregated over.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:5189777122064f2dd5fd78c70ab8e5a3a5dadcd5065c084fb5014ea599f3d8b2
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
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# gene-set-variation-analysis-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Per-cell gene set variation analysis (GSVA) scoring of single-cell RNA-seq data — e.g. the angiogenesis signature computed on the ICC tumor microenvironment dataset in the scWizard case study — to quantify pathway activity and rank cell populations such as Macro_NLRP3, Mono_CD14, and Arteries-like cells as angiogenesis-high.

## When to use

Use when scRNA-seq cells need pathway-level activity scores (e.g. MSigDB angiogenesis or a user GMT gene set) to flag pathway-high subpopulations within a tumor microenvironment.

## When NOT to use

- The analysis goal is marker-gene differential expression or cell-type annotation rather than pathway-level activity scoring — use scWizard's Seurat clustering or neural-network annotation modules instead.
- No predefined gene set exists (no MSigDB set or GMT file) and the pathway signature must be discovered de novo from the data.
- The input is already a per-cell or per-cluster feature/score table rather than raw or QC-filtered single-cell expression data.

## Inputs

- scRNA-seq dataset as a Seurat object (rds), counts matrix (csv/rds/h5 with genes as rows and cells as columns), or 10X output folder
- Gene set definition: MSigDB gene sets or a user-uploaded GMT file
- Optionally, cell-type annotations from scWizard's neural-network cell annotation module for aggregating scores per population

## Outputs

- Per-cell GSVA pathway scores for all QC-passed cells (exportable, e.g. gsva_angiogenesis_scores.csv)
- Per-cell-type score distributions/boxplots or UMAP overlays produced via the Plot module
- Ranked list of cell populations flagged as pathway-high (e.g. angiogenesis-high Macro_NLRP3, Mono_CD14, Arteries-like cells)

## How to apply

Load the scRNA-seq data into scWizard via the 'Input Data' module (Seurat object as 'Data-rds', counts matrix csv/rds/h5 as 'Data-counts', or a 10X folder), then apply scWizard's default quality control, which filters cells with <200 detected genes, >10,000 detected genes, or >5% mitochondrial read fraction. Open the 'GSVA' module and compute GSVA scores for the pathway of interest — the article used 'the GSVA method and gene sets from MSigDB database' to calculate the angiogenesis score — uploading a user-supplied GMT file if the gene set is custom. Aggregate per-cell scores by annotated cell type, export the score table (e.g. gsva_angiogenesis_scores.csv), and use the Plot module (UMAP overlay or per-cluster boxplot) to visualize and rank cell populations. Rationale: pathway-level scoring converts sparse single-cell expression into interpretable per-cell pathway activity, allowing identification of angiogenesis-high populations such as myeloid (Macro_NLRP3, Mono_CD14) and endothelial (Arteries-like) cells in the ICC case study.

## Related tools

- **scWizard** (Point-and-click Shiny GUI whose 'GSVA' module computes per-cell pathway scores on QC-passed single-cell data and displays/saves results; the Plot module visualizes score distributions.) — https://github.com/Dulab2020/scWizard
- **GSVA** (The underlying gene set variation analysis method used by scWizard to calculate pathway scores (e.g. angiogenesis score) per cell.)
- **GSVAdata** (Resource from which user-provided GMT gene set files can be supplied to the scWizard GSVA module as an alternative to MSigDB sets.)
- **Seurat** (Underlies scWizard's standardized pipeline (dimension reduction, clustering, differential expression) that defines the annotated cell populations the GSVA scores are aggregated over.)

## Examples

```
devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()  # then upload data, run Quality Control, and use the GSVA module with MSigDB or a GMT file
```

## Evaluation signals

- GSVA scores are computed for every cell in the QC-passed dataset (no missing scores after filtering cells with <200 genes, >10,000 genes, or >5% mitochondrial reads).
- Per-cell-type score distribution highlights a distinct pathway-high population consistent with the scWizard ICC case study: angiogenesis-high Macro_NLRP3 and Mono_CD14 among myeloid cells and Arteries-like cells among endothelial cells.
- The gene set source is verifiable: MSigDB via GSVA or an uploaded GMT file, matching the gene-set nomenclature in the input matrix.
- Score table exports successfully from the GSVA module (users 'can save the results') and plots regenerate in the Plot module from the saved scores.

## Limitations

- Results depend on the chosen gene set: MSigDB sets or a user-supplied GMT define what pathway activity is measured, and parameters 'can be adjusted', so score magnitudes are not comparable across gene set choices without care.
- Cells failing scWizard's default QC (<200 genes, >10,000 genes, >5% mitochondrial reads) are excluded before scoring, so rare low-content cells may receive no score.
- scWizard may not identify tissue-specific cell types, which 'may not be identified and are named in Unknown cluster', limiting which annotated populations pathway scores can be aggregated over.

## Evidence
