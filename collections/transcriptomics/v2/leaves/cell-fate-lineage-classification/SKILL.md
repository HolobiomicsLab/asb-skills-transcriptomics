---
name: cell-fate-lineage-classification
description: Use when you have GEMLI-predicted lineage assignments plus cell type
  annotations (e.g., DCIS vs inv_tumor cells from GSE148093 breast cancer biopsies)
  and need to separate symmetric from asymmetric (mixed-fate) lineages and profile
  DEGs per fate class.
when_to_use_negative:
- The cell population contains only distantly related cells — the article states GEMLI
  'will generally perform less robustly' in such populations, so lineage-based fate
  classification is unreliable
- The goal is long-term lineage tracing over weeks or months — GEMLI's scope is limited
  to lineages over about 1-5 cell divisions
- No predicted lineage table or cell type annotation exists yet (e.g., raw count matrix
  without running predict_lineages and prediction_to_lineage_information first)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0203
tools:
- name: GEMLI
  role: Provides extract_cell_fate_lineages, cell_type_composition_plot, cell_fate_DEG_calling,
    and DEG_volcano_plot to split predicted lineages into symmetric/asymmetric fates
    and call fate-associated DEGs
  repo: https://github.com/UPSUTER/GEMLI
- name: R
  role: Runtime environment for the GEMLI package (installed via devtools install_github('UPSUTER/GEMLI',
    subdir='GEMLI_package_v0'))
  repo: https://github.com/UPSUTER/GEMLI
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-47158-y
    title: Gene-expression memory-based prediction of cell lineages from scRNA-seq
      datasets
  source_content_hash: sha256:c374d1c64e392414819813ba66bc70d7003b0a91677770f3909d97084ac2bf7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/UPSUTER/GEMLI
  tools:
  - GEMLI
  - R
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cell-fate-lineage-classification

## Summary

Classifies GEMLI-predicted lineages from scRNA-seq data into symmetric versus asymmetric cell fates based on cell type composition, then calls differential expression between the fate classes (e.g., DCIS cells in symmetric vs. asymmetric lineages in human breast cancer). This turns lineage predictions into biological insight about recent cell fate switches.

## When to use

Use when you have GEMLI-predicted lineage assignments plus cell type annotations (e.g., DCIS vs inv_tumor cells from GSE148093 breast cancer biopsies) and need to separate symmetric from asymmetric (mixed-fate) lineages and profile DEGs per fate class.

## When NOT to use

- The cell population contains only distantly related cells — the article states GEMLI 'will generally perform less robustly' in such populations, so lineage-based fate classification is unreliable
- The goal is long-term lineage tracing over weeks or months — GEMLI's scope is limited to lineages over about 1-5 cell divisions
- No predicted lineage table or cell type annotation exists yet (e.g., raw count matrix without running predict_lineages and prediction_to_lineage_information first)

## Inputs

- Normalized gene expression matrix (e.g., GEMLI_cancer_example_norm_count.RData)
- Cell type annotation vector (e.g., GEMLI_cancer_example_cell_type_annotation.RData; DCIS and inv_tumor labels)
- GEMLI_items list including a predicted_lineage_table from predict_lineages converted via prediction_to_lineage_information

## Outputs

- GEMLI_items 'cell_fate_analysis' item with per-cell fate labels (sym_DCIS, asym_DCIS, sym_inv_tumor, asym_inv_tumor, NA_DCIS)
- Cell-type composition plots (plain or upsetR) of lineage composition
- DEG table between symmetric and asymmetric fate classes (e.g., CDH2, DCAF7, LINC01999 with adjusted p-values)
- Volcano plot of Sym_DCIS vs Asym_DCIS DEGs

## How to apply

Assemble the GEMLI_items list with 'gene_expression', 'predicted_lineage_table', and 'cell_type' slots, optionally summarizing lineage composition with cell_type_composition_plot (type='plain' or 'upsetR') to view symmetric DCIS, DCIS__inv_tumor, and symmetric inv_tumor lineages. Run extract_cell_fate_lineages(selection=c('inv_tumor','DCIS'), unique=FALSE, threshold=c(10,10)); the asymmetric class requires at least 10% of each cell type per lineage, which excludes lineages with too large an asymmetry, and the result is stored in the GEMLI_items 'cell_fate_analysis' item. Then call DEGs between fate classes with cell_fate_DEG_calling(ident1='sym_DCIS', ident2='asym_DCIS', min.pct=0.05, logfc.threshold=0.1) and visualize with DEG_volcano_plot(name1='Sym_DCIS', name2='Asym_DCIS'). In the paper's breast cancer example this recovered the invasion marker CDH2 plus DCAF7 and LINC01999 as genes enriched in DCIS cells from asymmetric (DCIS-to-invasive) lineages.

## Related tools

- **GEMLI** (Provides extract_cell_fate_lineages, cell_type_composition_plot, cell_fate_DEG_calling, and DEG_volcano_plot to split predicted lineages into symmetric/asymmetric fates and call fate-associated DEGs) — https://github.com/UPSUTER/GEMLI
- **R** (Runtime environment for the GEMLI package (installed via devtools install_github('UPSUTER/GEMLI', subdir='GEMLI_package_v0'))) — https://github.com/UPSUTER/GEMLI

## Examples

```
GEMLI_items <- extract_cell_fate_lineages(GEMLI_items, selection=c('inv_tumor','DCIS'), unique=FALSE, threshold=c(10,10))
```

## Evaluation signals

- Cell fate counts match the reported distribution: asym_DCIS=77, sym_DCIS=273, sym_inv_tumor=744, asym_inv_tumor=96, NA_DCIS=225
- CDH2, DCAF7, and LINC01999 are present in the DEG table with significant adjusted p-values
- Output is stored in the GEMLI_items 'cell_fate_analysis' item after extract_cell_fate_lineages
- Every asymmetric lineage contains at least 10% of each selected cell type (threshold=c(10,10))
- DEG_volcano_plot renders a volcano comparing Sym_DCIS vs Asym_DCIS without errors

## Limitations

- The 10% asymmetry threshold (threshold=c(10,10)) is a tunable choice; different thresholds will change which lineages are labeled asymmetric
- GEMLI performs less robustly in cell populations that contain only distantly related cells
- Large variability in growth rates between lineages may decrease GEMLI's performance (cannot be excluded)
- GEMLI's scope is limited to lineages over about 1-5 cell divisions; long-term lineage tracing may require other approaches

## Evidence

- [abstract] abstract_discriminate_fates: "GEMLI allows to study heritable gene expression, to discriminate symmetric and asymmetric cell fate decisions and to reconstruct individual multicellular structures"
- [methods] methods_asymmetry_threshold: "To exclude lineages with a too large asymmetry, we set a threshold to extract asymmetric lineages containing at least 10% of each cell type"
- [methods] methods_cell_fate_output: "The function output is stored in GEMLI_items 'cell_fate_analysis' item"
- [methods] methods_volcano: "DEG_volcano_plot(GEMLI_items, name1="Sym_DCIS", name2="Asym_DCIS")"
- [readme] readme_package_purpose: "GEMLI is an R package to predict cell lineages (cells with a common ancestor) from single cell RNA sequencing datasets and to call genes with a high gene expression memory"
