---
name: cnv-clustered-heatmap-visualization
description: Use when you have a cell-annotated cancer scRNA-seq object (e.g., Seurat
  object from scWizard annotation) and need to identify malignant cells by copy-number
  alteration and display them in a clustered inferCNV heatmap.
when_to_use_negative:
- Input is a raw unannotated counts matrix without cell-type labels, since no reference
  (non-malignant) or observation (malignant) cell sets can yet be designated
- Analysis goal is pathway scoring or transcription-factor regulation rather than
  chromosome copy-number alterations — use scWizard's 'GSVA' or 'TF-SCENIC' functions
  instead
- Tissue-specific or non-cancer cell types dominate and scWizard's annotation model
  labels them 'Unknown' (e.g., alveolar cells in lung cancers), leaving no reliable
  non-malignant reference set
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3173
- http://edamontology.org/topic_2640
tools:
- name: scWizard
  role: Provides the point-and-click 'inferCNV' function that wraps the inferCNV workflow,
    accepts the uploaded gene location file, and saves results to 'inforcnv_res'
  repo: https://github.com/Dulab2020/scWizard
- name: infercnv
  role: R package that estimates large-scale chromosome copy number alterations (CNA)
    from the scRNA-seq expression data
- name: Seurat
  role: Defines the standardized scRNA-seq pipeline and the annotated Seurat object
    ('Data-rds') that serves as input to the inferCNV step
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.08.028
    title: 'scWizard: A web-based automated tool for classifying and annotating single
      cells and downstream analysis of single-cell RNA-seq data in cancers'
  source_content_hash: sha256:a9ff48fbfcfbc013dd2652d402f64c86f6a8e08d27d9c2af28dc680e64b86048
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
  - http://edamontology.org/topic_3173
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cnv-clustered-heatmap-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Runs inferCNV through scWizard's 'inferCNV' function to estimate large-scale chromosome copy-number alterations (CNA) from annotated cancer scRNA-seq data, using non-malignant cells as reference, and visualizes the result as a clustered CNA heatmap separating malignant epithelial cells from non-malignant populations.

## When to use

Use when you have a cell-annotated cancer scRNA-seq object (e.g., Seurat object from scWizard annotation) and need to identify malignant cells by copy-number alteration and display them in a clustered inferCNV heatmap.

## When NOT to use

- Input is a raw unannotated counts matrix without cell-type labels, since no reference (non-malignant) or observation (malignant) cell sets can yet be designated
- Analysis goal is pathway scoring or transcription-factor regulation rather than chromosome copy-number alterations — use scWizard's 'GSVA' or 'TF-SCENIC' functions instead
- Tissue-specific or non-cancer cell types dominate and scWizard's annotation model labels them 'Unknown' (e.g., alveolar cells in lung cancers), leaving no reliable non-malignant reference set

## Inputs

- Annotated scRNA-seq Seurat object (e.g., 'Data-rds' input) with malignant epithelial and non-malignant cell labels from scWizard cell annotation
- Gene location file mapping genes to chromosomal coordinates (uploaded by the user in the inferCNV module)

## Outputs

- Per-cell copy-number alteration (CNA) estimates from inferCNV
- Clustered inferCNV heatmap with cells grouped by CNV profile and annotated by cell type
- inferCNV result files saved in the 'inforcnv_res' folder of the current working directory

## How to apply

Load the annotated scRNA-seq Seurat object produced by the scWizard cell annotation step, then designate the non-malignant (non-epithelial) populations — such as endothelial cells and fibroblasts in the ICC case study — as the inferCNV reference set and the annotated malignant epithelial cells as the observation set. Prepare and upload the gene location file mapping genes to chromosomal coordinates, which scWizard's 'inferCNV' function requires for copy-number variation analysis. Run the inferCNV R package as implemented in scWizard to estimate per-cell large-scale CNA from the expression data. Generate the clustered inferCNV heatmap with cells ordered by their CNV profiles and annotated by cell type, and save results — scWizard writes them to the 'inforcnv_res' folder of the current working directory. Validate by confirming the output file exists and that the heatmap visually separates the malignant epithelial compartment from non-malignant cells, as reported for the ICC case-study figure (no numeric CNA threshold is specified).

## Related tools

- **scWizard** (Provides the point-and-click 'inferCNV' function that wraps the inferCNV workflow, accepts the uploaded gene location file, and saves results to 'inforcnv_res') — https://github.com/Dulab2020/scWizard
- **infercnv** (R package that estimates large-scale chromosome copy number alterations (CNA) from the scRNA-seq expression data)
- **Seurat** (Defines the standardized scRNA-seq pipeline and the annotated Seurat object ('Data-rds') that serves as input to the inferCNV step)

## Examples

```
scWizard::run_app()  # then open the 'inferCNV' module, upload the gene location file, set malignant epithelial cells as observations and non-malignant cells as reference
```

## Evaluation signals

- Output files exist in the 'inforcnv_res' folder of the current working directory after the run
- The clustered CNV heatmap visually separates malignant epithelial cells from non-malignant populations (endothelial cells, fibroblasts), consistent with the ICC case-study figure
- Copy-number alterations in epithelial cancer cells exceed those in non-malignant cells, matching the expected epithelial origin of ICC tumors
- The gene location file was correctly formatted and uploaded, since the inferCNV module requires it before the analysis can run

## Limitations

- No numeric CNA threshold is reported in the paper; validation of malignancy relies on visual separation in the clustered heatmap
- Tissue-specific cell types may be named 'Unknown' by scWizard's annotation model (e.g., alveolar cells in lung cancers), which can compromise reference-set selection
- The user must supply the prepared gene location file themselves; scWizard does not provide it automatically
- In the README the results folder is spelled 'inforcnv_res' of the current working directory, so path expectations must match scWizard's implementation

## Evidence

- [readme] README inferCNV module details: [evidence span withheld — anchor status: not_found]
- [results] ICC finding on CNA in malignant cells: [evidence span withheld — anchor status: not_found]
