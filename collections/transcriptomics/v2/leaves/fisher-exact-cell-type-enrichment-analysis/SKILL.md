---
name: fisher-exact-cell-type-enrichment-analysis
description: Use when the query input is a list of genes rather than an expression
  matrix — e.g., top-N cluster marker genes from Seurat FindAllMarkers, or trait-associated
  genes mapped from GWAS SNPs — and cell type membership must be tested for enrichment.
when_to_use_negative:
- The query is a scaled gene expression profile (e.g., pbmc_cluster_marker_z_score)
  — use deCS.correlation() with PCC instead of the Fisher's exact test
- The goal is to discover novel cell types not represented in the reference panels
  — deCS is a mapping-based annotation toolkit and fails to characterize novel cell
  types
- Annotating cancer/malignant cells, where expression profile differences between
  healthy and malignant cells make deCS annotations challenging
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: deCS
  role: Provides deCS.fisher() implementing Fisher's exact test of query gene list
    overlap against cell type-specific CTgenes of a reference panel
  repo: https://github.com/bsml320/deCS
- name: Seurat
  role: Used upstream (FindAllMarkers, top_n by avg_log2FC) to produce the per-cluster
    marker gene lists that serve as Fisher test input
- name: ggplot2
  role: Dependency (>= 3.3.2) backing the deCS.fisher plot_figure heatmap output
- name: reshape2
  role: Dependency (>= 1.4.4) of the deCS package used for reshaping enrichment results
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:418f1a96a5773d6216e5ba461284edb17b9f6c49844661fd24d1131a28f63395
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - Seurat
  - deCS
  - dplyr
  - ggplot2
  - reshape2
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# fisher-exact-cell-type-enrichment-analysis

## Summary

Use deCS.fisher() to test whether a query gene list (e.g., cluster top-10 markers or GWAS trait-associated genes) is disproportionately overlapped in cell type-specific genes (CTgenes) of a deCS reference panel, assigning cell types or trait–cell type associations via Fisher's exact test.

## When to use

Use when the query input is a list of genes rather than an expression matrix — e.g., top-N cluster marker genes from Seurat FindAllMarkers, or trait-associated genes mapped from GWAS SNPs — and cell type membership must be tested for enrichment.

## When NOT to use

- The query is a scaled gene expression profile (e.g., pbmc_cluster_marker_z_score) — use deCS.correlation() with PCC instead of the Fisher's exact test
- The goal is to discover novel cell types not represented in the reference panels — deCS is a mapping-based annotation toolkit and fails to characterize novel cell types
- Annotating cancer/malignant cells, where expression profile differences between healthy and malignant cells make deCS annotations challenging

## Inputs

- Gene list per query (cluster top-10 marker genes from Seurat FindAllMarkers by avg_log2FC, or trait-associated genes mapped from GWAS summary statistics)
- deCS cell type specificity score reference panel (e.g., MonacoImmune_main, MonacoImmune_fine, BlueprintEncode_main, DICE_main) loaded via data()
- Optional cutoff parameters: ref_panel_markers_ratio = 0.05, p_threshold, cell_type_threshold, intersection_threshold

## Outputs

- Cell type enrichment/annotation table per query gene list (significant cell types after p_threshold and cell_type_threshold filtering)
- Trait x cell type association heatmap (e.g., Figure 7 pattern, trait_celltype_association_heatmap.pdf) when plot_figure = TRUE
- Tab-separated annotation table written via write.table (e.g., pbmc_deCS_result.txt)

## How to apply

Prepare the query as a gene list: for scRNA-seq clusters, take top-10 markers per cluster by avg_log2FC from FindAllMarkers; for GWAS, derive SNP-mapped trait gene lists with the repository preprocessing script (Example_code/3.GWAS_trait_associate_genes.R). Load a built-in t-score reference panel (e.g., data(MonacoImmune_main) or data(BlueprintEncode_main)) from the installed deCS package. Run deCS.fisher(markers_list, ref_panel), which tests overlap of the query gene set against CTgenes defined as the top 5% highest t-score/z-score genes per cell type (ref_panel_markers_ratio = 0.05). Apply p_threshold and cell_type_threshold filters (e.g., p_threshold = 0.01, cell_type_threshold = 0.5 for annotation; p_threshold = 1e-3, cell_type_threshold = 0.05 for trait enrichment) to call significant cell types. Optionally enable plot_figure to render the annotation/enrichment heatmap and export results with write.table(sep = "\t", quote = F). Concordance with the PCC-based deCS.correlation() approach on the same reference panel is expected for main labels.

## Related tools

- **deCS** (Provides deCS.fisher() implementing Fisher's exact test of query gene list overlap against cell type-specific CTgenes of a reference panel) — https://github.com/bsml320/deCS
- **Seurat** (Used upstream (FindAllMarkers, top_n by avg_log2FC) to produce the per-cluster marker gene lists that serve as Fisher test input)
- **ggplot2** (Dependency (>= 3.3.2) backing the deCS.fisher plot_figure heatmap output)
- **reshape2** (Dependency (>= 1.4.4) of the deCS package used for reshaping enrichment results)

## Examples

```
data(BlueprintEncode_main); res <- deCS.fisher(markers_list, BlueprintEncode_main, ref_panel_markers_ratio = 0.05, p_threshold = 1e-3, cell_type_threshold = 0.05, plot_figure = TRUE)
```

## Evaluation signals

- PBMC 3k validation: main labels for all 9 clusters must match published results — CD4+ T cells (Naive/Memory CD4 T), Monocytes (CD14+/FCGR3A+ Mono), B cells, CD8+ T cells, NK cells, Dendritic cells (DC), and Platelet -> Progenitors as the low-concordance case
- Main cell type labels from deCS.fisher must be concordant with deCS.correlation() run on the same reference panel (all 9 PBMC clusters identical in the published run)
- GWAS heatmap validation: neuropsychiatric disease-, college-, and education-associated genes enrich in neurons, while Alzheimer's disease associates with myeloid lineage cells rather than neurons (Figure 7 pattern)
- Significance filters applied: default CTgene cutoff of top 5% scoring genes per cell type (ref_panel_markers_ratio = 0.05) and user-set p_threshold/cell_type_threshold (e.g., 1e-3 and 0.05 for trait associations)
- Output table is tab-separated without quotes when written with write.table(quote = F)

## Limitations

- deCS does not take tissue information into account (except for HCAF), which may bias annotation results
- As a mapping-based annotation toolkit, deCS fails to characterize novel cell types absent from the reference panels
- deCS may be challenging for cancer cell annotations due to expression profile differences between healthy and malignant cells
- Results depend on the reference database chosen; expanding the references is critical for improving annotation accuracy

## Evidence

- [methods] deCS.fisher(markers_list, ref_panel, type, ref_panel_markers_ratio, top_n, p.adjust.methods, label, intersection_threshold, p_threshold, cell_type_threshold, plot_figure): "deCS.fisher(markers_list, ref_panel, type, ref_panel_markers_ratio, top_n, p.adjust.methods, label, intersection_threshold, p_threshold, cell_type_threshold, plot_figure)"
- [methods] CTgenes top 5% cutoff: "We allow the user to define the cutoff values, e.g., the top 5% genes with highest t-score/z-score as CTgenes"
- [methods] reference panels collected: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune"
- [readme] install instructions: "devtools::install_github("bsml320/deCS")"
- [readme] two test approaches: "Depending on the type of query data, we implemented two test approaches: Correlation analysis and Fisher's exact test for cell type enrichment analysis."
- [abstract] trait-cell type utility: "we demonstrated the broad utility of deCS to identify trait-cell type associations in 51 human complex traits"
