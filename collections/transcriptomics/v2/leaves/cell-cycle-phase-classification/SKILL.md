---
name: cell-cycle-phase-classification
description: Use when a processed Seurat object or count matrix from single-cell/spatial
  RNA-seq needs per-cell cell cycle phase assignment (G0, G1, S, G2M), e.g. to quantify
  quiescent fractions alongside accordion() cell type annotation.
when_to_use_negative:
- The goal is annotating cell identities rather than cell cycle staging — use accordion()
  with the built-in Accordion database instead
- The goal is scoring a custom pathway or gene signature (e.g. innate immune response
  via in_im_resp_sig) — use accordion_custom() instead
- Cell cycle genes have been regressed out or removed from the expression matrix,
  leaving no signal for the built-in cell cycle marker collection
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: cellmarkeraccordion
  role: R package providing accordion_cellcycle() and the built-in cell cycle marker
    collection used to classify cells into G0, G1, S and G2M phases
  repo: https://github.com/TebaldiLab/cellmarkeraccordion
- name: Seurat
  role: Preprocessing, clustering and object format (Seurat object) accepted by accordion_cellcycle();
    cell cycle assignments and fractions are extracted from the Seurat object
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60900-4
    title: 'Cell Marker Accordion: interpretable single-cell and spatial omics annotation
      in health and disease'
  source_content_hash: sha256:0a445726e5326c1b868e1393338f69c28850b9b3b200103bdd040b35b5878b0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TebaldiLab/cellmarkeraccordion
  tools:
  - Cell Marker Accordion
  - R
  - Seurat
  - cellmarkeraccordion
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cell-cycle-phase-classification

## Summary

Assigns cell cycle phases (G0, G1, S, G2M) to single-cell or spatial omics datasets using the accordion_cellcycle() function of the cellmarkeraccordion R package and its built-in cell cycle marker collection.

## When to use

Use when a processed Seurat object or count matrix from single-cell/spatial RNA-seq needs per-cell cell cycle phase assignment (G0, G1, S, G2M), e.g. to quantify quiescent fractions alongside accordion() cell type annotation.

## When NOT to use

- The goal is annotating cell identities rather than cell cycle staging — use accordion() with the built-in Accordion database instead
- The goal is scoring a custom pathway or gene signature (e.g. innate immune response via in_im_resp_sig) — use accordion_custom() instead
- Cell cycle genes have been regressed out or removed from the expression matrix, leaving no signal for the built-in cell cycle marker collection

## Inputs

- Processed Seurat object with normalized counts and cluster assignments (e.g. mouse bone marrow Mettl3 KO/WT or STM2457/vehicle datasets)
- Raw or normalized count matrix (genes on rows, cells on columns) as alternative input
- Condition metadata column (e.g. 'condition') for per-condition comparison of cell cycle fractions

## Outputs

- Per-cell cell cycle phase labels (G0, G1, S, G2M) stored in the Seurat object
- Per-condition cell cycle fractions (e.g. increased G0 fraction in Mettl3 KO and STM2457 conditions)
- Comparison plots of cell cycle fractions between conditions (e.g. Fig. 8-style visualizations)

## How to apply

Preprocess data with the standard Seurat pipeline (NormalizeData, FindVariableFeatures with vst and 2000 features, ScaleData, RunPCA, FindNeighbors, FindClusters resolution 0.4, RunUMAP), then run accordion_cellcycle() on the Seurat object (the function also accepts a raw or normalized count matrix), which classifies cells into G0, G1, S and G2M using the built-in cell cycle marker collection. The rationale, per the Cell Marker Accordion paper, is to combine cell cycle staging with consistency-weighted marker annotation to detect cell-cycle changes in specific hematopoietic cell types. In the METTL3 modulation use case, cell cycle assignments are extracted together with cell type labels from accordion() (species="Mouse", tissue="bone marrow", annotation_resolution="cluster", max_n_marker=30, allow_unknown=FALSE) and the G0 fraction is compared between conditions (Vehicle vs STM2457, WT vs Mettl3 KO). The reported biology to reproduce is an increased G0 fraction upon Mettl3 knockout or pharmacological inhibition, as shown in Fig. 8. Save phase assignments alongside cell type annotations and comparison plots as result files.

## Related tools

- **cellmarkeraccordion** (R package providing accordion_cellcycle() and the built-in cell cycle marker collection used to classify cells into G0, G1, S and G2M phases) — https://github.com/TebaldiLab/cellmarkeraccordion
- **Seurat** (Preprocessing, clustering and object format (Seurat object) accepted by accordion_cellcycle(); cell cycle assignments and fractions are extracted from the Seurat object)

## Examples

```
data <- accordion_cellcycle(data)  # classifies cells into G0, G1, S and G2M via the built-in cell cycle marker collection
```

## Evaluation signals

- Every cell receives exactly one of the four labels G0, G1, S or G2M from accordion_cellcycle()
- Per-sample cell cycle fractions sum to 1 across the four phases
- Cell cycle fractions are consistent with the accordion() cell type annotations (e.g. quiescent G0-enriched stem cell compartments in bone marrow)
- Reproduction check: G0 fraction is significantly higher in the Mettl3 KO and STM2457 conditions relative to WT/vehicle, matching the paper's Fig. 8 patterns
- Phase assignments and condition-split plots are saved as result files together with the annotation outputs

## Limitations

- Classification depends on the built-in cell cycle marker collection; datasets whose cell cycle genes are poorly covered or filtered out may stage incorrectly
- The paper's validation is demonstrated on specific hematopoietic and other benchmark contexts; behavior outside tested tissues (e.g. highly aberrant disease states) may need manual review
- Functions accept only a Seurat object or a raw/normalized count matrix, so other object formats require conversion first

## Evidence

- [methods] The Cell Marker Accordion includes an R package to automatically identify cell type, cell cycle stage and pathway activation in single-cell and spatial omics data: "The Cell Marker Accordion includes an R package to automatically identify cell type, cell cycle stage and pathway activation in single-cell and spatial omics data"
- [discussion] The Cell Marker Accordion identified cell cycle changes and activation of immune response pathways in specific hematopoietic cell types, consistent with the formation of aberrant endogenous dsRNAs: "The Cell Marker Accordion identified cell cycle changes and activation of immune response pathways in specific hematopoietic cell types, consistent with the formation of aberrant endogenous dsRNAs"
- [readme] All the functions of the cellmarkeraccordion accept as input either a Seurat object or a raw or normalized count matrix.: "All the functions of the <strong>cellmarkeraccordion</strong> accept as input either a Seurat object or a raw or normalized count matrix."
- [other] Assign cell cycle phases with accordion_cellcycle(), which classifies cells into G0, G1, S and G2M using the built-in cell cycle marker collection.: [evidence span withheld — anchor status: not_found]
