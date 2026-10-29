---
name: cancer-invasiveness-deg-validation
description: Use when analyzing human breast cancer scRNA-seq (e.g., GSE148093) with
  GEMLI to discriminate symmetric vs asymmetric DCIS/inv_tumor cell fates and call
  DEGs marking the onset of invasiveness.
when_to_use_negative:
- The dataset is not scRNA-seq (e.g., bulk RNA-seq or spatial data without single-cell
  lineages), since GEMLI predicts lineages solely from single-cell expression memory
- The cell population contains only distantly related cells, where the paper states
  GEMLI performs less robustly
- The goal is long-term lineage tracing over weeks or months, which exceeds GEMLI's
  scope of lineages over about 1-5 cell divisions and requires genetic-mark-based
  approaches
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: GEMLI
  role: R package used to predict lineages, extract symmetric/asymmetric cell fate
    lineages, and call DCIS DEGs (cell_fate_DEG_calling, DEG_volcano_plot)
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Runtime environment in which the GEMLI package and all workflow functions
    (predict_lineages, extract_cell_fate_lineages, cell_fate_DEG_calling) are executed
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: Used to install the GEMLI package from GitHub via install_github('UPSUTER/GEMLI',
    subdir='GEMLI_package_v0')
  repo: https://github.com/UPSUTER/GEMLI
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:76c60aef6ffa5d4602c92e14215973ea1a1e3371126152c3c63c948bcc41b6e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cancer-invasiveness-deg-validation

## Summary

Reproduces the GEMLI breast cancer case study: predicting DCIS-to-invasive (asymmetric) cell lineages from scRNA-seq and calling differentially expressed genes between DCIS cells in symmetric vs asymmetric lineages to uncover invasion markers (CDH2) and novel candidates (DCAF7, LINC01999).

## When to use

Use when analyzing human breast cancer scRNA-seq (e.g., GSE148093) with GEMLI to discriminate symmetric vs asymmetric DCIS/inv_tumor cell fates and call DEGs marking the onset of invasiveness.

## When NOT to use

- The dataset is not scRNA-seq (e.g., bulk RNA-seq or spatial data without single-cell lineages), since GEMLI predicts lineages solely from single-cell expression memory
- The cell population contains only distantly related cells, where the paper states GEMLI performs less robustly
- The goal is long-term lineage tracing over weeks or months, which exceeds GEMLI's scope of lineages over about 1-5 cell divisions and requires genetic-mark-based approaches

## Inputs

- Normalized scRNA-seq gene expression matrix (GEMLI_cancer_example_norm_count.RData, human breast cancer, GSE148093)
- Cell type annotation table (GEMLI_cancer_example_cell_type_annotation.RData; DCIS and inv_tumor labels)
- GEMLI_items list with 'gene_expression', 'predicted_lineage_table', and 'cell_type' slots

## Outputs

- Lineage assignments grouped into sym_DCIS, asym_DCIS, sym_inv_tumor, asym_inv_tumor, and NA classes (stored in GEMLI_items 'cell_fate_analysis' item)
- DEG table between sym_DCIS and asym_DCIS cells including CDH2, DCAF7, LINC01999 with adjusted p-values
- Volcano plot (DEG_volcano_plot, Sym_DCIS vs Asym_DCIS) and optional cell-type composition plots (plain/upsetR)

## How to apply

Assemble a GEMLI_items list from the normalized count matrix ('GEMLI_cancer_example_norm_count.RData'), cell type annotation ('GEMLI_cancer_example_cell_type_annotation.RData'), and the predicted_lineage_table obtained via predict_lineages and prediction_to_lineage_information; optionally inspect lineage composition with cell_type_composition_plot (type='plain' or 'upsetR'). Extract symmetric and asymmetric cell fate lineages with extract_cell_fate_lineages(selection=c('inv_tumor','DCIS'), unique=FALSE, threshold=c(10,10)), where asymmetric lineages must contain at least 10% of each cell type to exclude overly asymmetric lineages. Call DEGs between DCIS cells of symmetric versus asymmetric lineages using cell_fate_DEG_calling(ident1='sym_DCIS', ident2='asym_DCIS', min.pct=0.05, logfc.threshold=0.1) and visualize with DEG_volcano_plot(name1='Sym_DCIS', name2='Asym_DCIS'). Validate against the paper's cell.fate distribution (asym_DCIS=77, sym_DCIS=273, sym_inv_tumor=744, asym_inv_tumor=96, NA_DCIS=225) and confirm CDH2, DCAF7, and LINC01999 appear in the DEG table with significant adjusted p-values.

## Related tools

- **GEMLI** (R package used to predict lineages, extract symmetric/asymmetric cell fate lineages, and call DCIS DEGs (cell_fate_DEG_calling, DEG_volcano_plot)) — https://github.com/UPSUTER/GEMLI
- **R** (Runtime environment in which the GEMLI package and all workflow functions (predict_lineages, extract_cell_fate_lineages, cell_fate_DEG_calling) are executed) — https://github.com/UPSUTER/GEMLI
- **devtools** (Used to install the GEMLI package from GitHub via install_github('UPSUTER/GEMLI', subdir='GEMLI_package_v0')) — https://github.com/UPSUTER/GEMLI

## Examples

```
DEG_volcano_plot(GEMLI_items, name1="Sym_DCIS", name2="Asym_DCIS")
```

## Evaluation signals

- Cell.fate counts match the reported distribution: asym_DCIS=77, sym_DCIS=273, sym_inv_tumor=744, asym_inv_tumor=96, NA_DCIS=225
- CDH2, DCAF7, and LINC01999 are present in the DEG table with significant adjusted p-values
- Asymmetric lineages contain at least 10% of each cell type (threshold=c(10,10)) after extract_cell_fate_lineages
- GEMLI_items contains a 'cell_fate_analysis' item populated by extract_cell_fate_lineages
- DEG_volcano_plot renders with the Sym_DCIS vs Asym_DCIS contrast using min.pct=0.05 and logfc.threshold=0.1

## Limitations

- GEMLI will generally perform less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease GEMLI's performance (cannot be excluded)
- GEMLI's scope is limited to lineages over about 1-5 cell divisions; long-term lineage tracing over weeks or months requires other approaches such as naturally occurring genetic marks

## Evidence

- [abstract] In human breast cancer biopsies, GEMLI reveals previously unknown gene expression changes at the onset of cancer invasiveness.: "In human breast cancer biopsies, GEMLI reveals previously unknown gene expression changes at the onset of cancer invasiveness."
- [methods] To exclude lineages with a too large asymmetry, we set a threshold to extract asymmetric lineages containing at least 10% of each cell type: "To exclude lineages with a too large asymmetry, we set a threshold to extract asymmetric lineages containing at least 10% of each cell type"
- [methods] DEG_volcano_plot(GEMLI_items, name1="Sym_DCIS", name2="Asym_DCIS"): "> DEG_volcano_plot(GEMLI_items, name1="Sym_DCIS", name2="Asym_DCIS")"
- [methods] The function output is stored in GEMLI_items 'cell_fate_analysis' item: "The function output is stored in GEMLI_items 'cell_fate_analysis' item"
- [readme] GEMLI is an R package to predict cell lineages (cells with a common ancestor) from single cell RNA sequencing datasets: "GEMLI is an R package to predict cell lineages (cells with a common ancestor) from single cell RNA sequencing datasets"
