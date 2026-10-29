---
name: scrna-seq-lineage-inference
description: Use when scRNA-seq data lacks lineage labels but you need cell lineage
  relationships, e.g., to study heritable gene expression, symmetric vs asymmetric
  cell fate decisions, or DEGs at cell fate switches.
when_to_use_negative:
- Cell populations contain only distantly related cells — GEMLI performs less robustly
  there
- Long-term lineage tracing over weeks or months is needed — GEMLI is limited to lineages
  over about 1–5 cell divisions
- Lineages with large variability in growth rates, which may decrease GEMLI's performance
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: GEMLI
  role: 'Core R package: predict_lineages, prediction_to_lineage_information, extract_cell_fate_lineages,
    cell_fate_DEG_calling, DEG_volcano_plot'
  repo: https://github.com/UPSUTER/GEMLI
- name: devtools
  role: Installation of GEMLI from GitHub via install_github
  repo: https://github.com/UPSUTER/GEMLI
- name: igraph
  role: Network visualization of predicted lineages via visualize_as_network
- name: topGO
  role: GO biological process over-representation analysis on memory gene sets derived
    from predicted vs ground-truth lineages
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:f013e2d0165f09deae2faa2d9271e990b78570938d72a4363d85695f875b7e2c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  - devtools
  - igraph
  - topGO
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# scrna-seq-lineage-inference

## Summary

Predict small to medium-sized cell lineages (cells sharing a common ancestor over ~1–5 divisions) from scRNA-seq data alone using GEMLI, which leverages genes with stable, heritable expression (gene expression memory) through repeated iterative clustering — no barcode or genetic lineage tracing required.

## When to use

Use when scRNA-seq data lacks lineage labels but you need cell lineage relationships, e.g., to study heritable gene expression, symmetric vs asymmetric cell fate decisions, or DEGs at cell fate switches.

## When NOT to use

- Cell populations contain only distantly related cells — GEMLI performs less robustly there
- Long-term lineage tracing over weeks or months is needed — GEMLI is limited to lineages over about 1–5 cell divisions
- Lineages with large variability in growth rates, which may decrease GEMLI's performance

## Inputs

- Normalized scRNA-seq gene expression matrix (rows = genes, columns = cells), e.g., from GEO (GSE148093 breast cancer, GSE99915 MEF data)
- GEMLI_items list with 'gene_expression' slot; optionally 'barcodes' (barcode ID values, cell ID names) and/or 'cell_type' annotation vector

## Outputs

- Cell-by-cell matrix of lineage confidence scores from predict_lineages
- Lineage assignments per cell from prediction_to_lineage_information
- Symmetric/asymmetric cell fate lineages from extract_cell_fate_lineages (stored in GEMLI_items 'cell_fate_analysis')
- DEG table and volcano plot for lineage-subset comparisons (e.g., Sym_DCIS vs Asym_DCIS)

## How to apply

Install GEMLI from GitHub with devtools::install_github('UPSUTER/GEMLI', subdir='GEMLI_package_v0') and assemble a GEMLI_items list with a 'gene_expression' slot (normalized counts, rows = genes, columns = cells) and optionally 'barcodes' or 'cell_type' slots. Run GEMLI::predict_lineages on the GEMLI_items to obtain a cell-by-cell matrix of lineage confidence scores via repeated iterative clustering (~2–3 min), then convert the predictions with prediction_to_lineage_information into discrete lineage assignments. Optionally validate with test_lineage_prediction against barcode ground truth, or visualize/trim candidate lineages with visualize_as_network, suggest_network_trimming_to_size, and trim_network_to_size (e.g., max_size=2, cutoff=50). For fate-decision analyses, extract symmetric/asymmetric lineages with extract_cell_fate_lineages (threshold=10, i.e., asymmetric lineages must contain at least 10% of each cell type), then call DEGs with cell_fate_DEG_calling (e.g., min.pct=0.05, logfc.threshold=0.1) and plot with DEG_volcano_plot.

## Related tools

- **GEMLI** (Core R package: predict_lineages, prediction_to_lineage_information, extract_cell_fate_lineages, cell_fate_DEG_calling, DEG_volcano_plot) — https://github.com/UPSUTER/GEMLI
- **devtools** (Installation of GEMLI from GitHub via install_github) — https://github.com/UPSUTER/GEMLI
- **igraph** (Network visualization of predicted lineages via visualize_as_network)
- **topGO** (GO biological process over-representation analysis on memory gene sets derived from predicted vs ground-truth lineages)

## Examples

```
library(devtools); install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0"); library(GEMLI); GEMLI_items <- predict_lineages(GEMLI_items)
```

## Evaluation signals

- test_lineage_prediction shows predicted lineages match ground-truth barcode lineages (e.g., GSE99915 barcode annotations)
- Memory genes and GO-term enrichments from GEMLI-predicted lineages correlate with ground-truth-derived ones (Spearman correlation > 0.7, consistent with Fig. 3a)
- Cell fate counts match expected distributions (breast cancer example: asym_DCIS=77, sym_DCIS=273, sym_inv_tumor=744, asym_inv_tumor=96, NA_DCIS=225)
- Known lineage-relevant genes recovered in DEG tables, e.g., CDH2 invasion marker plus DCAF7 and LINC01999 with significant adjusted p-values
- visualize_as_network renders plausible connected lineage clusters when inspected with igraph

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease GEMLI's performance
- Scope is limited to lineages related over about 1–5 cell divisions; longer time spans require lineage-tracing approaches based on naturally occurring genetic marks

## Evidence

- [abstract] a computational tool allowing to robustly identify small to medium-sized cell lineages solely from scRNA-seq datasets: "Here we introduce Gene Expression Memory-based Lineage Inference (GEMLI), a computational tool allowing to robustly identify small to medium-sized cell lineages solely from scRNA-seq datasets."
- [methods] predict_lineages takes GEMLI_items as input: "We can then identify cell lineages through repeated iterative clustering (this may take 2-3min). The `predict_lineages` function takes our GEMLI_items as input"
- [methods] threshold to extract asymmetric lineages containing at least 10% of each cell type: "To exclude lineages with a too large asymmetry, we set a threshold to extract asymmetric lineages containing at least 10% of each cell type"
- [discussion] GEMLI will generally perform less robustly in cell populations that contain only distantly related cells: "However, GEMLI will generally perform less robustly in cell populations that contain only distantly related cells."
- [readme] install_github instructions from README: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`."
