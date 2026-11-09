---
name: angiogenesis-high-population-identification
description: Use when scoring single-cell RNA-seq (e.g., Seurat object or counts matrix
  from csv/rds/h5/10X) for a pathway signature like angiogenesis to flag which TME
  cell subpopulations are pathway-high.
when_to_use_negative:
- Input is already an aggregated feature/signature score table rather than per-cell
  expression data — the skill requires a QC-passed counts matrix or Seurat object
- The pathway of interest is not represented in MSigDB or an available GMT file and
  no user-supplied gene set can be prepared
- The dataset has not passed quality control or cell-type annotation, since angiogenesis-high
  populations are flagged by per-cell-type aggregation of annotated subpopulations
edam_operation: http://edamontology.org/operation_3232
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scWizard
  role: Shiny-based R package whose 'Input Data', 'Quality Control', 'GSVA', and 'Plot'
    modules are used to load the ICC scRNA-seq data, QC-filter cells, compute per-cell
    GSVA angiogenesis scores, and visualize per-type score distributions
  repo: https://github.com/Dulab2020/scWizard
- name: GSVA
  role: Gene set variation analysis method applied within scWizard to score each cell
    for the angiogenesis gene set
- name: MSigDB
  role: Source database for the angiogenesis gene sets used in the GSVA scoring
- name: GSVAdata
  role: Resource from which user-provided GMT gene set files can be uploaded into
    scWizard's GSVA module
- name: Seurat
  role: Standardized scRNA-seq pipeline (dimension reduction, clustering, differential
    expression) underlying scWizard's processing before/alongside GSVA scoring
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:52e75bf88584a465168231d633639ac72e6f6264f56687e8c6e7db6dfbf98f08
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
  edam_operation: http://edamontology.org/operation_3232
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# angiogenesis-high-population-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies angiogenesis-high cell subpopulations in tumor single-cell RNA-seq data by computing per-cell GSVA angiogenesis scores (MSigDB gene sets) with scWizard, then aggregating and ranking cell types by score — as demonstrated on the intrahepatic cholangiocarcinoma (ICC) dataset in the scWizard case study.

## When to use

Use when scoring single-cell RNA-seq (e.g., Seurat object or counts matrix from csv/rds/h5/10X) for a pathway signature like angiogenesis to flag which TME cell subpopulations are pathway-high.

## When NOT to use

- Input is already an aggregated feature/signature score table rather than per-cell expression data — the skill requires a QC-passed counts matrix or Seurat object
- The pathway of interest is not represented in MSigDB or an available GMT file and no user-supplied gene set can be prepared
- The dataset has not passed quality control or cell-type annotation, since angiogenesis-high populations are flagged by per-cell-type aggregation of annotated subpopulations

## Inputs

- Cancer scRNA-seq dataset as a Seurat object ('Data-rds')
- Counts matrix in csv, rds, or h5 format, genes as rows and cells as columns ('Data-counts')
- 10X output folder ('Data-10X')
- GMT gene set file (optional, e.g., angiogenesis signature from MSigDB or GSVAdata resources)

## Outputs

- Per-cell GSVA angiogenesis score table (e.g., gsva_angiogenesis_scores.csv)
- Per-cell-type aggregated angiogenesis score distribution (UMAP overlay or per-cluster boxplot from the Plot module)
- Ranked list of angiogenesis-high cell subpopulations

## How to apply

Load the cancer scRNA-seq dataset into scWizard via the 'Input Data' module (Seurat object via 'Data-rds', counts matrix via 'Data-counts', or 'Data-10X' folder). Apply the default quality control, filtering cells with <200 detected genes, >10,000 detected genes, or >5% mitochondrial reads as low-quality. Open the 'GSVA' module and compute GSVA scores for the angiogenesis pathway using gene sets from the MSigDB database, optionally uploading a user-provided GMT file and adjusting parameters. Aggregate per-cell scores by annotated cell type (e.g., myeloid subtypes Macro_NLRP3, Mono_CD14 and endothelial Arteries-like cells in the ICC case), export the score table (e.g., gsva_angiogenesis_scores.csv), and visualize distributions via the Plot module (UMAP overlay or per-cluster boxplot) to rank and flag angiogenesis-high populations. The rationale is that pathway-level scoring on QC-passed single cells reveals which TME subpopulations harbor elevated angiogenesis activity that per-gene analysis obscures.

## Related tools

- **scWizard** (Shiny-based R package whose 'Input Data', 'Quality Control', 'GSVA', and 'Plot' modules are used to load the ICC scRNA-seq data, QC-filter cells, compute per-cell GSVA angiogenesis scores, and visualize per-type score distributions) — https://github.com/Dulab2020/scWizard
- **GSVA** (Gene set variation analysis method applied within scWizard to score each cell for the angiogenesis gene set)
- **MSigDB** (Source database for the angiogenesis gene sets used in the GSVA scoring)
- **GSVAdata** (Resource from which user-provided GMT gene set files can be uploaded into scWizard's GSVA module)
- **Seurat** (Standardized scRNA-seq pipeline (dimension reduction, clustering, differential expression) underlying scWizard's processing before/alongside GSVA scoring)

## Examples

```
if (!requireNamespace("devtools", quietly=TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- GSVA scores are computed for every cell in the QC-passed ICC dataset (cells with 200–10,000 detected genes and ≤5% mitochondrial reads)
- Per-cell-type aggregation of scores produces a distinct angiogenesis-high population consistent with the scWizard ICC case study: Macro_NLRP3 and Mono_CD14 among myeloid cells and Arteries-like cells among endothelial cells
- The exported score table (gsva_angiogenesis_scores.csv) contains one row per cell with the angiogenesis GSVA score and cell-type annotation
- Plot-module visualization (UMAP overlay or per-cluster boxplot) shows score distributions separated by cell type, enabling an unambiguous ranking of angiogenesis-high populations
- Uploaded GMT file (if used) matches the gene naming of the input matrix so that the angiogenesis set yields nonzero scores rather than empty gene overlap

## Limitations

- The angiogenesis score depends on the gene set source (MSigDB or a user-supplied GMT); a poorly matched gene set will misrank populations
- scWizard's tissue-specific cell types may not be identified and can fall into an Unknown cluster, limiting interpretation of pathway-high signals for tissue-specific subtypes
- scWizard's annotation covers 7 major cell types and 47 subtypes within the TME; populations outside this taxonomy require custom training data for reliable type assignment before aggregation

## Evidence

- [methods] GSVA angiogenesis scoring with MSigDB gene sets in the ICC case study: "We used the GSVA method and gene sets from MSigDB database [22] to calculate the angiogenesis score."
- [methods] scWizard QC thresholds: "By default, scWizard defined cells with <200 genes detected, > 10,000 genes detected, >5 % of reads mapping to mitochondrial RNA as low-quality cells and filtered them out"
- [readme] GSVA module usage in scWizard: "“GSVA” function is for calculating GSVA score for specific gene sets or signaling pathways. GMT file could be uploaded by users and parameter can be adjusted."
- [readme] Input data formats supported by scWizard: "'Data-counts': The input data is the counts matrix, the rows are genes, and the columns are samples, including csv, rds, and h5 format; 'Data-10X': The input is a folder containing 10x data."
