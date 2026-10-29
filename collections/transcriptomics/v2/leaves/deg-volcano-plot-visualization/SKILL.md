---
name: deg-volcano-plot-visualization
description: Use when you have already called DEGs between two cell-fate groups (e.g.,
  sym_DCIS vs asym_DCIS from GEMLI lineage predictions) and need to visualize genes
  with significant adjusted p-values and log-fold changes such as CDH2, DCAF7, and
  LINC01999.
when_to_use_negative:
- Input lineages have not yet been partitioned into symmetric/asymmetric cell fates
  via extract_cell_fate_lineages — DEG calling and plotting will have no ident groups
- Cell populations contain only distantly related cells, where GEMLI performs less
  robustly and DEG group assignments are unreliable
- No per-cell-type DEG table exists (e.g., only a raw expression matrix without lineage
  predictions) — visualization would not be grounded in called DEGs
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: GEMLI
  role: R package providing extract_cell_fate_lineages, cell_fate_DEG_calling, and
    DEG_volcano_plot used to call and visualize DEGs between symmetric and asymmetric
    lineage cells
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Runtime environment for the GEMLI package and the DEG volcano plot function
  repo: https://github.com/UPSUTER/GEMLI
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:0e576cf94b0cd44d9e3d9ba056331816ffd174e5e7e65f932fc46ad4ef84ebce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# deg-volcano-plot-visualization

## Summary

Generate volcano plots of differentially expressed genes (DEGs) identified between cell fate groups, specifically to visualize DCIS-cell DEGs between symmetric and asymmetric GEMLI-predicted lineages in the human breast cancer scRNA-seq dataset.

## When to use

Use when you have already called DEGs between two cell-fate groups (e.g., sym_DCIS vs asym_DCIS from GEMLI lineage predictions) and need to visualize genes with significant adjusted p-values and log-fold changes such as CDH2, DCAF7, and LINC01999.

## When NOT to use

- Input lineages have not yet been partitioned into symmetric/asymmetric cell fates via extract_cell_fate_lineages — DEG calling and plotting will have no ident groups
- Cell populations contain only distantly related cells, where GEMLI performs less robustly and DEG group assignments are unreliable
- No per-cell-type DEG table exists (e.g., only a raw expression matrix without lineage predictions) — visualization would not be grounded in called DEGs

## Inputs

- GEMLI_items list containing gene_expression matrix (e.g., 'GEMLI_cancer_example_norm_count.RData'), predicted_lineage_table, and cell_type annotation ('GEMLI_cancer_example_cell_type_annotation.RData')
- cell_fate_analysis output from extract_cell_fate_lineages
- DEG table from cell_fate_DEG_calling with min.pct=0.05, logfc.threshold=0.1

## Outputs

- Volcano plot contrasting DEGs between the two named groups (e.g., Sym_DCIS vs Asym_DCIS)
- DEG table with log-fold changes and adjusted p-values for genes including CDH2, DCAF7, and LINC01999

## How to apply

After predicting lineages with GEMLI's predict_lineages and converting them with prediction_to_lineage_information, extract symmetric and asymmetric cell fate lineages via extract_cell_fate_lineages(selection=c('inv_tumor','DCIS'), unique=FALSE, threshold=c(10,10)), where asymmetric lineages must contain at least 10% of each cell type. Then call DEGs with cell_fate_DEG_calling(ident1='sym_DCIS', ident2='asym_DCIS', min.pct=0.05, logfc.threshold=0.1). Finally pass the populated GEMLI_items list to DEG_volcano_plot with the group names (name1='Sym_DCIS', name2='Asym_DCIS') to render a volcano plot of log-fold change versus significance. Verify key genes (CDH2, DCAF7, LINC01999) appear with significant adjusted p-values and that cell.fate counts match the expected distribution (asym_DCIS=77, sym_DCIS=273, sym_inv_tumor=744, asym_inv_tumor=96, NA_DCIS=225).

## Related tools

- **GEMLI** (R package providing extract_cell_fate_lineages, cell_fate_DEG_calling, and DEG_volcano_plot used to call and visualize DEGs between symmetric and asymmetric lineage cells) — https://github.com/UPSUTER/GEMLI
- **R** (Runtime environment for the GEMLI package and the DEG volcano plot function) — https://github.com/UPSUTER/GEMLI

## Examples

```
DEG_volcano_plot(GEMLI_items, name1="Sym_DCIS", name2="Asym_DCIS")
```

## Evaluation signals

- Cell.fate counts match the reported distribution: asym_DCIS=77, sym_DCIS=273, sym_inv_tumor=744, asym_inv_tumor=96, NA_DCIS=225
- CDH2, DCAF7, and LINC01999 are present in the DEG table with significant adjusted p-values
- DEG calling used the documented thresholds min.pct=0.05 and logfc.threshold=0.1
- Volcano plot axis assignment matches name1 (Sym_DCIS) and name2 (Asym_DCIS) group ordering
- Asymmetric lineages were extracted with threshold=c(10,10), ensuring at least 10% of each cell type per lineage

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells, which could distort symmetric/asymmetric group assignments underlying the DEG comparison
- Large variability in growth rates between lineages may decrease GEMLI's prediction performance
- Lineage inference is limited to small and medium-sized lineages (about 1-5 cell divisions); long-term lineage questions need other approaches

## Evidence

- [methods] we notable call DEG for DCIS cells in asymmetric and symmetric lineages: "we notable call DEG for DCIS cells in asymmetric and symmetric lineages"
- [methods] DEG volcano plot invocation: "> DEG_volcano_plot(GEMLI_items, name1="Sym_DCIS", name2="Asym_DCIS")"
- [methods] asymmetry threshold for lineage extraction: "To exclude lineages with a too large asymmetry, we set a threshold to extract asymmetric lineages containing at least 10% of each cell type"
- [abstract] cancer invasiveness findings from breast cancer biopsies: "In human breast cancer biopsies, GEMLI reveals previously unknown gene expression changes at the onset of cancer invasiveness."
- [readme] GEMLI installation and library loading: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`."
