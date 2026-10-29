---
name: differential-gene-expression-analysis
description: Use when you have GEMLI-predicted symmetric/asymmetric cell fate lineages
  with cell type annotations and need to identify DEGs between cells of the same type
  that followed different fate decisions.
when_to_use_negative:
- Cells lack predicted lineage/fate assignments (no cell_fate_analysis in GEMLI_items),
  since the contrast groups here are lineage-defined rather than cluster-defined
- Populations contain only distantly related cells, where GEMLI lineage prediction
  (and hence fate-based DEG grouping) performs less robustly
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: GEMLI
  role: R package providing extract_cell_fate_lineages, cell_fate_DEG_calling, and
    DEG_volcano_plot to define lineage-based cell groups and call DEGs between them
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Runtime environment; GEMLI is installed as an R package via devtools install_github
  repo: https://github.com/UPSUTER/GEMLI
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:538cf26a82d7ea8e635fade22c0d48ef565acaf98b8bb87691fba0ac98886aa5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# differential-gene-expression-analysis

## Summary

Calls differentially expressed genes between cell groups defined by GEMLI-predicted lineage fate (e.g., DCIS cells in symmetric vs. asymmetric lineages from the human breast cancer scRNA-seq dataset), then visualizes them with a volcano plot. This exposes gene expression changes at the onset of cancer invasiveness that are invisible without lineage information.

## When to use

Use when you have GEMLI-predicted symmetric/asymmetric cell fate lineages with cell type annotations and need to identify DEGs between cells of the same type that followed different fate decisions.

## When NOT to use

- Cells lack predicted lineage/fate assignments (no cell_fate_analysis in GEMLI_items), since the contrast groups here are lineage-defined rather than cluster-defined
- Populations contain only distantly related cells, where GEMLI lineage prediction (and hence fate-based DEG grouping) performs less robustly

## Inputs

- GEMLI_items list containing 'gene_expression' (normalized count matrix, e.g., GEMLI_cancer_example_norm_count.RData), 'predicted_lineage_table', and 'cell_type' annotation (GEMLI_cancer_example_cell_type_annotation.RData)
- cell_fate_analysis output from extract_cell_fate_lineages assigning per-cell symmetric/asymmetric fate labels

## Outputs

- DEG table for each contrast (e.g., Sym_DCIS vs. Asym_DCIS) with log fold changes and adjusted p-values, stored in GEMLI_items
- Volcano plot rendering of the DEG table via DEG_volcano_plot

## How to apply

After extract_cell_fate_lineages(selection=c('inv_tumor','DCIS'), unique=FALSE, threshold=c(10,10)) assigns each cell a cell.fate (with asymmetric lineages required to contain at least 10% of each cell type), run cell_fate_DEG_calling(GEMLI_items, ident1='sym_DCIS', ident2='asym_DCIS', min.pct=0.05, logfc.threshold=0.1) to compare DCIS cells in symmetric versus asymmetric lineages. This DEG call on lineage-defined groups is the mechanism by which the paper found CDH2 (a known invasion marker) plus previously uncharacterized progression genes DCAF7 and LINC01999. Store the DEG table in the GEMLI_items and render it with DEG_volcano_plot(GEMLI_items, name1='Sym_DCIS', name2='Asym_DCIS'). Optionally, cell_type_composition_plot (type='plain' or 'upsetR') can be used first to sanity-check lineage composition across symmetric DCIS, DCIS__inv_tumor, and symmetric inv_tumor lineages before DEG calling.

## Related tools

- **GEMLI** (R package providing extract_cell_fate_lineages, cell_fate_DEG_calling, and DEG_volcano_plot to define lineage-based cell groups and call DEGs between them) — https://github.com/UPSUTER/GEMLI
- **R** (Runtime environment; GEMLI is installed as an R package via devtools install_github) — https://github.com/UPSUTER/GEMLI

## Examples

```
GEMLI_items <- cell_fate_DEG_calling(GEMLI_items, ident1='sym_DCIS', ident2='asym_DCIS', min.pct=0.05, logfc.threshold=0.1); DEG_volcano_plot(GEMLI_items, name1="Sym_DCIS", name2="Asym_DCIS")
```

## Evaluation signals

- Cell fate counts match the reported distribution: asym_DCIS=77, sym_DCIS=273, sym_inv_tumor=744, asym_inv_tumor=96, NA_DCIS=225
- CDH2, DCAF7, and LINC01999 are present in the sym_DCIS vs. asym_DCIS DEG table with significant adjusted p-values
- DEG call used the documented parameters: min.pct=0.05 and logfc.threshold=0.1 on ident1='sym_DCIS', ident2='asym_DCIS'
- Volcano plot renders with name1='Sym_DCIS' and name2='Asym_DCIS' labels matching the DEG table entries in GEMLI_items

## Limitations

- GEMLI performs less robustly in cell populations that contain only distantly related cells, weakening lineage-based group definitions for DEG calling
- Large variability in growth rates between lineages may decrease GEMLI's performance and thus confound fate-based comparisons
- GEMLI's scope is limited to lineages over about 1-5 cell divisions; DEG analysis on longer-timescale lineage differences requires other approaches

## Evidence

- [methods] To exclude lineages with a too large asymmetry, we set a threshold to extract asymmetric lineages containing at least 10% of each cell type: "To exclude lineages with a too large asymmetry, we set a threshold to extract asymmetric lineages containing at least 10% of each cell type"
- [methods] DEG for DCIS cells: "we notable call DEG for DCIS cells in asymmetric and symmetric lineages"
- [methods] Volcano plot invocation: "> DEG_volcano_plot(GEMLI_items, name1="Sym_DCIS", name2="Asym_DCIS")"
- [abstract] Breast cancer invasiveness finding: "In human breast cancer biopsies, GEMLI reveals previously unknown gene expression changes at the onset of cancer invasiveness."
- [readme] GEMLI installation via R: "Simply run `library(devtools)` and then `install_github("UPSUTER/GEMLI", subdir="GEMLI_package_v0")`"
