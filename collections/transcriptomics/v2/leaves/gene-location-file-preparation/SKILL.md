---
name: gene-location-file-preparation
description: Use when running the scWizard 'inferCNV' module on an annotated cancer
  scRNA-seq dataset (e.g., the ICC case study) and a gene-to-chromosomal-coordinate
  mapping file must be prepared before copy number analysis.
when_to_use_negative:
- The analysis goal is cell-type classification or subtype annotation rather than
  large-scale chromosome copy number alteration detection — use scWizard's 'Cell annotation'
  module instead
- The input is downstream scRNA-seq analysis without a counts matrix (e.g., already-reduced
  dimensionality data), since CNA estimation needs gene-level expression with mappable
  gene identifiers
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scWizard
  role: Point-and-click GUI whose 'inferCNV' function accepts the prepared gene location
    file and runs copy number variation analysis, saving results to the 'inforcnv_res'
    folder
  repo: https://github.com/Dulab2020/scWizard
- name: infercnv
  role: R package implemented within scWizard's 'inferCNV' function to analyze large-scale
    chromosome copy number alterations from scRNA-seq data
- name: Seurat
  role: Defines the standardized scRNA-seq pipeline in scWizard and supplies the annotated
    Seurat object that designates reference (non-malignant) and observation (malignant
    epithelial) cell sets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:34d602cff4dedcae6c70afdebdd4af7e89da82435a06487955fa972a3bd79b74
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
  - infercnv
  - scWizard
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# gene-location-file-preparation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepare and upload the gene location file mapping genes to chromosomal coordinates required by scWizard's 'inferCNV' function, so that large-scale chromosome copy number alterations (CNA) can be estimated from annotated scRNA-seq data in cancer studies.

## When to use

Use when running the scWizard 'inferCNV' module on an annotated cancer scRNA-seq dataset (e.g., the ICC case study) and a gene-to-chromosomal-coordinate mapping file must be prepared before copy number analysis.

## When NOT to use

- The analysis goal is cell-type classification or subtype annotation rather than large-scale chromosome copy number alteration detection — use scWizard's 'Cell annotation' module instead
- The input is downstream scRNA-seq analysis without a counts matrix (e.g., already-reduced dimensionality data), since CNA estimation needs gene-level expression with mappable gene identifiers

## Inputs

- Annotated scRNA-seq Seurat object with cell-type labels from scWizard's cell annotation step (e.g., ICC malignant epithelial cells vs. non-malignant endothelial cells and fibroblasts)
- Prepared gene location file mapping genes to chromosomal coordinates
- Reference and observation cell population assignments

## Outputs

- inferCNV copy-number alteration estimates per cell
- Clustered inferCNV heatmap separating malignant epithelial cells from non-malignant populations
- Results files saved in the 'inforcnv_res' folder of the current working directory

## How to apply

Start from the annotated Seurat object produced by scWizard's cell annotation step and decide which populations define the inferCNV reference (non-malignant, non-epithelial cells such as endothelial cells and fibroblasts) and the observation set (annotated malignant epithelial cells). Prepare the gene location file mapping each gene to its chromosomal coordinates — scWizard's inferCNV page requires users to upload this prepared file before running copy number variation analysis. Launch the scWizard GUI (installed via devtools::install_github("Dulab2020/scWizard"), requiring R v4.0.2 or greater and jags), open the 'inferCNV' function, upload the gene location file, and adjust the exposed parameters as needed. The module then runs the infercnv analysis to estimate large-scale chromosome copy number alterations and writes results, including the clustered CNA heatmap, to the 'inforcnv_res' folder of the current working directory.

## Related tools

- **scWizard** (Point-and-click GUI whose 'inferCNV' function accepts the prepared gene location file and runs copy number variation analysis, saving results to the 'inforcnv_res' folder) — https://github.com/Dulab2020/scWizard
- **infercnv** (R package implemented within scWizard's 'inferCNV' function to analyze large-scale chromosome copy number alterations from scRNA-seq data)
- **Seurat** (Defines the standardized scRNA-seq pipeline in scWizard and supplies the annotated Seurat object that designates reference (non-malignant) and observation (malignant epithelial) cell sets)

## Examples

```
devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- The output 'inforcnv_res' folder exists in the current working directory and contains the inferCNV results and heatmap files
- The clustered CNV heatmap visually separates the malignant epithelial compartment from non-malignant populations (endothelial cells, fibroblasts), consistent with the ICC case-study figure
- Copy-number alterations in epithelial cancer cells exceed those in non-malignant cells, consistent with the epithelial origin of ICC tumors
- The uploaded gene location file maps every gene in the counts matrix to a chromosomal coordinate so the inferCNV run completes without gene-mapping errors

## Limitations

- No numeric threshold is reported for judging CNA burden separation; validation is visual comparison against the case-study figure
- scWizard's tissue-specific cell types may not be identified and fall into an Unknown cluster, which complicates designating reference vs. observation sets for non-canonical tissues
- Results are saved to the current working directory ('inforcnv_res' folder), so the working directory must be set correctly before launching the GUI

## Evidence

- [other] Designate the non-malignant (non-epithelial) cell populations as the inferCNV reference set and the annotated malignant epithelial cells as the observation set.: [evidence span withheld — anchor status: not_found]
- [results] The InferCNV analysis showed copy-number alterations in epithelial cancer cells exceeded those in non-malignant cells such as endothelial cells and fibroblasts.: [evidence span withheld — anchor status: not_found]
