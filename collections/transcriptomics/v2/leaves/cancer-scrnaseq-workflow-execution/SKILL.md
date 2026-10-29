---
name: cancer-scrnaseq-workflow-execution
description: Use when analyzing cancer scRNA-seq data (Seurat object, counts matrix
  in csv/rds/h5, or 10X folder) and you need automated TME cell-type annotation plus
  downstream analyses instead of manual marker-based labeling.
when_to_use_negative:
- The sample is normal/non-cancer tissue or the target cell types are tissue-specific
  (e.g., alveolar cells in lung cancers) — scWizard's TME reference may leave such
  cells in the 'Unknown' cluster
- You need novel de novo cell-type discovery with a reference you build from scratch
  and fully controlled model architecture — the packaged point-and-click workflow
  is designed for standardized automated annotation
- Input data is already annotated and you only need one narrow downstream step — although
  single modules (e.g., CellphoneDB, Monocle) can be run, the full workflow skill
  would be redundant
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scWizard
  role: 'Primary tool: shiny-based GUI packaging QC, Seurat/Harmony batch correction,
    hierarchical deep-neural-network cell annotation (7 major types, 47 subtypes),
    and 11 downstream analyses'
  repo: https://github.com/Dulab2020/scWizard
- name: Seurat
  role: Underlying standardized scRNA-seq pipeline inside scWizard for dimension reduction,
    cell clustering, and differential expression; also one of two batch-effect removal
    methods
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:651438d4dfea5be4e71af4ebbc74e44dfec85950e90f4a436f0b95c0d3fb75a5
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
  - Seurat
  - scWizard
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cancer-scrnaseq-workflow-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Execute scWizard's end-to-end cancer scRNA-seq workflow — QC, optional batch correction, deep-neural-network hierarchical cell-type annotation, and downstream analysis — replicating the ICC case study that resolved 8 marker-verified clusters into TME major types and 47 subtypes.

## When to use

Use when analyzing cancer scRNA-seq data (Seurat object, counts matrix in csv/rds/h5, or 10X folder) and you need automated TME cell-type annotation plus downstream analyses instead of manual marker-based labeling.

## When NOT to use

- The sample is normal/non-cancer tissue or the target cell types are tissue-specific (e.g., alveolar cells in lung cancers) — scWizard's TME reference may leave such cells in the 'Unknown' cluster
- You need novel de novo cell-type discovery with a reference you build from scratch and fully controlled model architecture — the packaged point-and-click workflow is designed for standardized automated annotation
- Input data is already annotated and you only need one narrow downstream step — although single modules (e.g., CellphoneDB, Monocle) can be run, the full workflow skill would be redundant

## Inputs

- Seurat object ('Data-rds')
- Counts matrix in csv, rds, or h5 format ('Data-counts', genes as rows, cells as columns)
- 10X data folder ('Data-10X')
- Optional custom trainset in H5 format (genes as rows, cells as columns) with matching label file

## Outputs

- Per-cell predicted labels among 7 major TME cell types and 47 subtypes (or 'Unknown' for uncovered tissue-specific types)
- Cluster fractions / cell-type composition summary
- UMAP 'single-cell map' colored by major type and subtype
- QC-filtered Seurat object and optional batch-corrected (Seurat/Harmony) object
- Downstream outputs from 11 packaged analyses (GSVA, inferCNV, TF-SCENIC, CellphoneDB, Monocle, correlation, plotting)

## How to apply

Install scWizard from GitHub (devtools::install_github("Dulab2020/scWizard"), R >= 4.0.2, with JAGS installed) and launch the shiny GUI via scWizard::run_app(); click 'install Python' and copy the downloaded trainset into the 'app/www/python' folder before annotating. Run Quality Control with the default filters — removing cells with <200 detected genes, >10,000 genes, or >5% mitochondrial reads — and optionally apply Seurat or Harmony batch correction before annotation if multiple samples/technologies are present. Run the 'Cell annotation' module: the two-step model first classifies individual cells into 7 major TME cell types with the pre-trained hierarchical neural network (ReLU hidden layers, softmax output, PCA-reduced features, trained on 113,976 cells across 13 cancer types), then Seurat subclustering feeds the subtype step assigning one of 47 TME subtypes per subcluster, accepting the predicted subtype with the largest number of cells per subcluster. Adjust hidden-layer node counts only per the documented criteria (between input and output sizes, 2/3 input + 2/3 output, or less than twice input size); otherwise keep defaults. Validate by checking marker gene expression of annotated clusters, visualizing UMAP 'single-cell map' colored by major type and subtype, and confirming behavior against reference accuracies (0.98 major types, 0.85 myeloid subtypes, 0.79 T/NK subtypes).

## Related tools

- **scWizard** (Primary tool: shiny-based GUI packaging QC, Seurat/Harmony batch correction, hierarchical deep-neural-network cell annotation (7 major types, 47 subtypes), and 11 downstream analyses) — https://github.com/Dulab2020/scWizard
- **Seurat** (Underlying standardized scRNA-seq pipeline inside scWizard for dimension reduction, cell clustering, and differential expression; also one of two batch-effect removal methods)

## Examples

```
if (!requireNamespace("devtools", quietly=TRUE)) install.packages("devtools"); devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- Every cell receives a label among the 7 major cell types / 47 subtypes, or 'Unknown' for tissue-specific types not covered by the reference
- Annotated clusters are verified by canonical marker gene expression, with only a few 'Unknown' cells discarded (as in the ICC case study yielding 8 clusters)
- Reference accuracy benchmarks are approached: 0.98 for major cell types, 0.85 for myeloid subtypes, 0.79 for T/NK subtypes on independent labeled datasets (MCC 11,024 cells; CRC 11,717 myeloid; LC 45,555 T cells)
- UMAP colored by predicted major type and subtype shows coherent, non-scattered cluster assignments; cell-type composition fractions sum to 1
- QC thresholds applied as documented: cells with <200 genes, >10,000 genes, or >5% mitochondrial reads are removed before annotation

## Limitations

- Tissue-specific cell types (e.g., alveolar cells in lung cancers) may not be identified and are named in the 'Unknown' cluster
- Identification of tissue-specific subtypes and cellular states is deferred to future updates
- Python environment must be installed via the 'install Python' button and the trainset downloaded and copied to the 'app/www/python' folder before annotation; JAGS must be installed beforehand (source tarball on Windows, conda on Linux)
- Requires R v4.0.2 or greater

## Evidence
