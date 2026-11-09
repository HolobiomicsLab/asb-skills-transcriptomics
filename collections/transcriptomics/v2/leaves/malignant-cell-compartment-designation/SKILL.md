---
name: malignant-cell-compartment-designation
description: Use when an annotated cancer scRNA-seq object (e.g., Seurat object from
  scWizard cell annotation) must be partitioned into malignant epithelial versus non-malignant
  compartments using inferred copy-number alterations.
when_to_use_negative:
- Input is a raw unannotated counts matrix — cell annotation must be performed first
  to define reference and observation sets
- The question concerns tissue-specific cell subtypes that scWizard may label 'Unknown'
  (e.g., alveolar cells in lung cancers), making reference-set designation unreliable
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scWizard
  role: Shiny-based GUI that packages the 'inferCNV' function for analyzing large-scale
    chromosome copy number alterations and saving results to 'inforcnv_res'
  repo: https://github.com/Dulab2020/scWizard
- name: infercnv
  role: R package implemented within scWizard's 'inferCNV' function to estimate single-cell
    copy-number alterations from scRNA-seq data
- name: Seurat
  role: Provides the annotated Seurat object input format ('Data-rds') that feeds
    the inferCNV step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:f5c9689bb67eb6bff248508622c1c09b4b3bd2f9fcbd1eea8cde8f676fa94ba0
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
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# malignant-cell-compartment-designation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Designates malignant versus non-malignant cell compartments from scWizard cell-annotation output by running large-scale chromosome copy-number alteration (CNA) analysis with the 'inferCNV' function, using non-epithelial cells (e.g., endothelial cells, fibroblasts) as the reference and annotated malignant epithelial cells as observations, then visualizing results as a clustered CNA heatmap.

## When to use

Use when an annotated cancer scRNA-seq object (e.g., Seurat object from scWizard cell annotation) must be partitioned into malignant epithelial versus non-malignant compartments using inferred copy-number alterations.

## When NOT to use

- Input is a raw unannotated counts matrix — cell annotation must be performed first to define reference and observation sets
- The question concerns tissue-specific cell subtypes that scWizard may label 'Unknown' (e.g., alveolar cells in lung cancers), making reference-set designation unreliable

## Inputs

- Annotated scRNA-seq Seurat object from scWizard cell annotation (e.g., the ICC scRNA-seq dataset)
- Prepared gene location file mapping genes to chromosomal coordinates
- Cell-type labels designating malignant epithelial cells (observation set) and non-malignant non-epithelial cells (reference set)

## Outputs

- inferCNV results with single-cell copy-number alteration estimates per cell
- Clustered inferCNV heatmap separating malignant epithelial cells from non-malignant populations
- Results saved to the 'inforcnv_res' folder of the current working directory

## How to apply

Load the annotated scRNA-seq Seurat object produced by scWizard's hierarchical deep-neural-network cell annotation (7 major cell types, 47 subtypes). In the scWizard GUI, designate the non-malignant non-epithelial populations as the inferCNV reference set and the annotated malignant epithelial cells as the observation set. Upload the prepared gene location file mapping genes to chromosomal coordinates required for CNA analysis, adjust parameters on the inferCNV page, and run the analysis via scWizard's 'inferCNV' function (R package infercnv). Results are saved in the 'inforcnv_res' folder of the current working directory; cells clustered by their CNV profiles should visually separate the malignant epithelial compartment from non-malignant populations such as endothelial cells and fibroblasts.

## Related tools

- **scWizard** (Shiny-based GUI that packages the 'inferCNV' function for analyzing large-scale chromosome copy number alterations and saving results to 'inforcnv_res') — https://github.com/Dulab2020/scWizard
- **infercnv** (R package implemented within scWizard's 'inferCNV' function to estimate single-cell copy-number alterations from scRNA-seq data)
- **Seurat** (Provides the annotated Seurat object input format ('Data-rds') that feeds the inferCNV step)

## Examples

```
devtools::install_github("Dulab2020/scWizard"); scWizard::run_app()  # then use the 'inferCNV' page: set reference = non-epithelial cells, upload gene location file
```

## Evaluation signals

- Output heatmap file exists in the 'inforcnv_res' folder of the current working directory
- Clustered CNV heatmap visually separates the malignant epithelial compartment from non-malignant populations (endothelial cells, fibroblasts)
- Copy-number alterations in epithelial cancer cells exceed those in non-malignant cells, consistent with the epithelial origin of ICC tumors
- The gene location file was successfully uploaded and accepted for copy-number variation analysis
- Compartment designation matches the annotated cell labels from scWizard's hierarchical annotation output

## Limitations

- No numeric CNA threshold is reported; validation is visual (heatmap separation), not threshold-based
- Tissue-specific cell types may not be identified and fall in the Unknown cluster, complicating reference-set choice
- inferCNV requires a separately prepared gene location file that the user must upload

## Evidence

- [full_text] The InferCNV analysis showed copy-number alterations in epithelial cancer cells exceeded those in non-malignant cells such as endothelial cells and fibroblasts, consistent with the epithelial origin of ICC tumors, and was visualized as a clustered CNA heatmap.: [evidence span withheld — anchor status: not_found]
- [methods] Designate the non-malignant (non-epithelial) cell populations as the inferCNV reference set and the annotated malignant epithelial cells as the observation set.: [evidence span withheld — anchor status: not_found]
- [results] Load the annotated ICC scRNA-seq Seurat object produced by the scWizard cell annotation step.: [evidence span withheld — anchor status: not_found]
