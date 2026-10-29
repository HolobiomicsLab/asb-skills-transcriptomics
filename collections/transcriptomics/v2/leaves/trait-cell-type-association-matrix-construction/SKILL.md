---
name: trait-cell-type-association-matrix-construction
description: Use when you have preprocessed GWAS trait-associated gene lists (e.g.,
  from 51 GWAS summary datasets) and want to map traits to associated human cell types
  via CTgene enrichment.
when_to_use_negative:
- The query is a gene expression profile (scRNA-seq/bulk RNA-seq matrix) rather than
  a gene list — deCS.correlation() should be used instead of deCS.fisher().
- The goal is annotating scRNA-seq clusters rather than linking traits to cell types
  via enrichment over CTgenes.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: deCS
  role: Provides deCS.fisher() Fisher's exact test, built-in reference panels (BlueprintEncode_main),
    and the plot_figure heatmap capability
  repo: https://github.com/bsml320/deCS
- name: ggplot2
  role: Dependency used by deCS.fisher's plot_figure parameter to render the trait-cell
    type association heatmap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:7781bdeecdcac743018cfbf88da8c2a810a944631a33551bdee4c004cf25fc57
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - deCS
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# trait-cell-type-association-matrix-construction

## Summary

Constructs a trait x cell type association table/heatmap by testing whether GWAS trait-associated gene sets are disproportionately overlapped in cell type-specific genes (CTgenes) using deCS's Fisher's exact test against a specificity-score reference panel such as BlueprintEncode_main.

## When to use

Use when you have preprocessed GWAS trait-associated gene lists (e.g., from 51 GWAS summary datasets) and want to map traits to associated human cell types via CTgene enrichment.

## When NOT to use

- The query is a gene expression profile (scRNA-seq/bulk RNA-seq matrix) rather than a gene list — deCS.correlation() should be used instead of deCS.fisher().
- The goal is annotating scRNA-seq clusters rather than linking traits to cell types via enrichment over CTgenes.

## Inputs

- Preprocessed GWAS trait-associated gene lists (SNPs mapped to genes) for 51 complex traits
- BlueprintEncode_main cell type specificity score reference panel (built-in deCS data)
- CTgene cutoff: ref_panel_markers_ratio = 0.05 (top 5% highest t-score/z-score genes per cell type)

## Outputs

- Trait x cell type table of significant Fisher's exact test associations
- trait_celltype_association_heatmap.pdf (Figure 7 heatmap of trait-cell type associations)

## How to apply

Obtain trait-associated gene lists from GWAS summary data via the repository preprocessing script (Example_code/3.GWAS_trait_associate_genes.R). Load a cell type specificity score reference panel with data(BlueprintEncode_main) from the installed deCS package. For each trait gene list, call deCS.fisher(markers_list, BlueprintEncode_main), which implements Fisher's exact test against CTgenes defined as the top 5% highest-scoring genes per cell type (ref_panel_markers_ratio = 0.05). Assemble significant results into a trait x cell type table applying p_threshold and cell_type_threshold filters (e.g., p_threshold = 1e-3, cell_type_threshold = 0.05), then plot the heatmap via deCS.fisher's plot_figure parameter (using ggplot2) and save as trait_celltype_association_heatmap.pdf.

## Related tools

- **deCS** (Provides deCS.fisher() Fisher's exact test, built-in reference panels (BlueprintEncode_main), and the plot_figure heatmap capability) — https://github.com/bsml320/deCS
- **ggplot2** (Dependency used by deCS.fisher's plot_figure parameter to render the trait-cell type association heatmap)

## Examples

```
data(BlueprintEncode_main); results <- deCS.fisher(markers_list, BlueprintEncode_main, ref_panel_markers_ratio = 0.05, p_threshold = 1e-3, cell_type_threshold = 0.05, plot_figure = TRUE)
```

## Evaluation signals

- Reproduced heatmap matches the reported Figure 7 pattern: neuropsychiatric disease-, college-, and education-associated genes mainly enriched in neurons.
- Alzheimer's disease maps to myeloid lineage cells rather than neurons, as reported.
- Significant cell types are called only after applying both p_threshold and cell_type_threshold filters.
- CTgene sets correspond to the top 5% of highest specificity-score genes per cell type (ref_panel_markers_ratio = 0.05).
- Result table is a complete trait x cell type matrix covering all 51 GWAS traits.

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias annotation results.
- CTgene cutoffs (ref_panel_markers_ratio, p_threshold, cell_type_threshold) are user-defined and affect which associations are called significant.
- No software version for deCS is stated in the discussion.

## Evidence

- [abstract] deCS demonstrated broad utility for trait-cell type associations: "we demonstrated the broad utility of deCS to identify trait-cell type associations in 51 human complex traits"
- [methods] deCS.fisher function signature with filter parameters: "deCS.fisher(markers_list, ref_panel, type, ref_panel_markers_ratio, top_n, p.adjust.methods, label, intersection_threshold, p_threshold, cell_type_threshold, plot_figure)"
- [methods] CTgene cutoff definition: "We allow the user to define the cutoff values, e.g., the top 5% genes with highest t-score/z-score as CTgenes"
- [readme] deCS built-in reference panels including BlueprintEncode: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune"
- [methods] Filter threshold examples: "p_threshold = 1e-3, cell_type_threshold = 0.05"
