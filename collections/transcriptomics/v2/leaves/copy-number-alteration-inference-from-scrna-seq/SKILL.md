---
name: copy-number-alteration-inference-from-scrna-seq
description: Use when you have an annotated cancer scRNA-seq Seurat object and want
  to identify malignant cells by their copy-number alterations relative to a non-malignant
  reference population.
when_to_use_negative:
- The input has not yet been through scWizard cell annotation, so a non-malignant
  reference population and malignant observation population cannot be defined from
  cell-type labels
- The goal is bulk-tumor copy-number calling from DNA data rather than scRNA-seq-based
  CNA inference
- Tissue-specific cell types are labeled Unknown and may contaminate the reference
  compartment with unannotated cells
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scWizard
  role: Provides the point-and-click 'inferCNV' function that runs large-scale CNA
    analysis and saves results to the current working directory
  repo: https://github.com/Dulab2020/scWizard
- name: infercnv
  role: R package implemented by scWizard to estimate single-cell copy number alterations
    from the scRNA-seq data
- name: Seurat
  role: Supplies the annotated Seurat object (from scWizard's Seurat-based clustering
    and annotation pipeline) used as inferCNV input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:f7ff1d52d95a96dd35a412dd2cf2d6a60e5b32ba9549d9875b8b9f8420e8d546
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
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# copy-number-alteration-inference-from-scrna-seq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Infer large-scale chromosome copy number alterations (CNA) from cancer scRNA-seq data using scWizard's 'inferCNV' function (built on the R infercnv package), and visualize malignant versus non-malignant compartments as a clustered CNA heatmap.

## When to use

Use when you have an annotated cancer scRNA-seq Seurat object and want to identify malignant cells by their copy-number alterations relative to a non-malignant reference population.

## When NOT to use

- The input has not yet been through scWizard cell annotation, so a non-malignant reference population and malignant observation population cannot be defined from cell-type labels
- The goal is bulk-tumor copy-number calling from DNA data rather than scRNA-seq-based CNA inference
- Tissue-specific cell types are labeled Unknown and may contaminate the reference compartment with unannotated cells

## Inputs

- Annotated Seurat object from scWizard cell annotation (e.g., the ICC scRNA-seq dataset)
- Non-malignant (non-epithelial) cell populations designated as the inferCNV reference set
- Annotated malignant epithelial cells designated as the observation set
- Prepared gene location file mapping genes to chromosomal coordinates

## Outputs

- Per-cell large-scale copy number alteration (CNA) estimates
- Clustered inferCNV heatmap separating malignant epithelial cells from non-malignant populations
- InferCNV results and heatmap saved in the 'inforcnv_res' folder of the current working directory

## How to apply

Load the annotated Seurat object produced by scWizard's deep-neural-network cell annotation step (e.g., the ICC scRNA-seq case study), and designate the non-epithelial, non-malignant populations (endothelial cells, fibroblasts) as the inferCNV reference set and the annotated malignant epithelial cells as the observation set. Upload the prepared gene location file mapping genes to chromosomal coordinates, which is required input for the analysis. Run scWizard's 'inferCNV' function to estimate large-scale CNA from the expression matrix; results are saved to the 'inforcnv_res' folder of the current working directory. Generate the clustered inferCNV heatmap with cells ordered by CNV profile and annotated by cell type, so the malignant epithelial compartment can be visually separated from the non-malignant reference populations. In the ICC case, epithelial cancer cells showed copy-number alterations exceeding those of endothelial cells and fibroblasts, consistent with the epithelial origin of ICC tumors.

## Related tools

- **scWizard** (Provides the point-and-click 'inferCNV' function that runs large-scale CNA analysis and saves results to the current working directory) — https://github.com/Dulab2020/scWizard
- **infercnv** (R package implemented by scWizard to estimate single-cell copy number alterations from the scRNA-seq data)
- **Seurat** (Supplies the annotated Seurat object (from scWizard's Seurat-based clustering and annotation pipeline) used as inferCNV input)

## Examples

```
devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()
```

## Evaluation signals

- The 'inforcnv_res' folder of the current working directory exists and contains the inferCNV output and heatmap files
- The clustered CNA heatmap visually separates the malignant epithelial compartment from the non-malignant reference populations (endothelial cells, fibroblasts), matching the ICC case-study figure
- Epithelial cancer cells exhibit more copy-number alterations than the non-malignant reference cells, consistent with the epithelial origin of the tumor
- The gene location file was successfully uploaded and accepted by the 'inferCNV' module before the run

## Limitations

- No numeric threshold is reported for calling malignant cells — validation is by visual inspection of the clustered heatmap against the case-study figure
- Tissue-specific cell types (e.g., alveolar cells in lung cancers) may not be identified and fall in the Unknown cluster, which could corrupt reference/observation designation
- A prepared gene location file mapping genes to chromosomal coordinates must be supplied by the user; it is not generated by the tool

## Evidence
