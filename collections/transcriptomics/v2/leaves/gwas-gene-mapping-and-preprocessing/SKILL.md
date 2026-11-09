---
name: gwas-gene-mapping-and-preprocessing
description: Use when you have GWAS summary statistics (or preprocessed trait-associated
  gene lists) and want to identify which human cell types are significantly associated
  with a complex trait or disease.
when_to_use_negative:
- The query input is a gene expression profile or scRNA-seq cluster matrix — use deCS.correlation()
  instead of deCS.fisher()
- Input gene lists are not derived from GWAS/complex-trait data (e.g., cluster markers
  from FindAllMarkers), since the Fisher enrichment question is trait-to-cell-type
  association
- The target cell types are not represented in the available reference panels (BlueprintEncode,
  DICE, MonacoImmune, HCL, HCAF)
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3170
tools:
- name: deCS
  role: Provides deCS.fisher() Fisher's exact test for cell type enrichment of GWAS
    gene sets, built-in reference panels such as BlueprintEncode_main, and the plot_figure
    heatmap capability
  repo: https://github.com/bsml320/deCS
- name: ggplot2
  role: Rendering dependency used by deCS.fisher's plot_figure parameter to draw the
    trait-cell type association heatmap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.gpb.2022.04.001
    title: '<i>deCS</i> : A Tool for Systematic Cell Type Annotations of Single-Cell
      RNA Sequencing Data Among Human Tissues'
  source_content_hash: sha256:a4c54e6e535f738b971ec005f29c8ecf6f68ac7eb110d95eb2390b8d99f84617
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bsml320/deCS
  tools:
  - deCS
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# gwas-gene-mapping-and-preprocessing

## Summary

Preprocessing workflow that derives SNP lists from publicly available GWAS summary data, maps SNPs to trait-associated genes, and tests those gene sets for enrichment in cell type-specific genes (CTgenes) using deCS's Fisher's exact test, as demonstrated for 51 human complex traits against the BlueprintEncode reference panel.

## When to use

Use when you have GWAS summary statistics (or preprocessed trait-associated gene lists) and want to identify which human cell types are significantly associated with a complex trait or disease.

## When NOT to use

- The query input is a gene expression profile or scRNA-seq cluster matrix — use deCS.correlation() instead of deCS.fisher()
- Input gene lists are not derived from GWAS/complex-trait data (e.g., cluster markers from FindAllMarkers), since the Fisher enrichment question is trait-to-cell-type association
- The target cell types are not represented in the available reference panels (BlueprintEncode, DICE, MonacoImmune, HCL, HCAF)

## Inputs

- GWAS summary statistics for complex traits (e.g., 51 publicly available GWAS datasets)
- Preprocessed trait-associated gene lists produced by Example_code/3.GWAS_trait_associate_genes.R
- Cell type specificity score reference panel (e.g., BlueprintEncode_main loaded via data(BlueprintEncode_main))
- Filter parameters: ref_panel_markers_ratio (top 5% CTgenes cutoff), p_threshold, cell_type_threshold, intersection_threshold

## Outputs

- Trait x cell type enrichment table of significant associations from deCS.fisher
- Significant cell type calls per trait after p_threshold and cell_type_threshold filtering
- Trait-cell type association heatmap (Figure 7 analogue), e.g., trait_celltype_association_heatmap.pdf via plot_figure

## How to apply

Obtain trait-associated gene lists for each GWAS summary dataset using the repository preprocessing script (Example_code/3.GWAS_trait_associate_genes.R), which derives SNPs from GWAS summary data and maps them to genes. Load the pre-calculated cell type specificity score reference panel with data(BlueprintEncode_main) from the installed deCS package. Run deCS.fisher(markers_list, BlueprintEncode_main), which implements Fisher's exact test of whether the query gene set is disproportionately overlapped in CTgenes defined as the top 5% highest t-score/z-score genes per cell type (ref_panel_markers_ratio = 0.05). Assemble results into a trait x cell type table, applying p_threshold and cell_type_threshold filters (e.g., p_threshold = 0.01, cell_type_threshold = 0.5) to call significant cell types. Enable plot_figure to render the trait-cell type association heatmap and verify it against the reported Figure 7 pattern (neuropsychiatric traits enriched in neurons; Alzheimer's disease associated with myeloid lineage cells).

## Related tools

- **deCS** (Provides deCS.fisher() Fisher's exact test for cell type enrichment of GWAS gene sets, built-in reference panels such as BlueprintEncode_main, and the plot_figure heatmap capability) — https://github.com/bsml320/deCS
- **ggplot2** (Rendering dependency used by deCS.fisher's plot_figure parameter to draw the trait-cell type association heatmap)

## Examples

```
library(deCS); data(BlueprintEncode_main); deCS.fisher(markers_list, BlueprintEncode_main, ref_panel_markers_ratio = 0.05, p_threshold = 0.01, cell_type_threshold = 0.5, plot_figure = TRUE)
```

## Evaluation signals

- The GWAS preprocessing script produces a gene list per trait with SNPs mapped to genes before any enrichment test is run
- deCS.fisher runs with ref_panel_markers_ratio = 0.05 so CTgenes correspond to the top 5% highest specificity-score genes per cell type
- Significant cell types are called only after applying p_threshold and cell_type_threshold filters to the trait x cell type enrichment table
- The rendered heatmap reproduces the reported Figure 7 pattern: neuropsychiatric/college/education traits enriched in neurons and Alzheimer's disease associated with myeloid lineage cells rather than neurons
- The enrichment output columns and filters match the deCS.fisher signature (ref_panel_markers_ratio, top_n, p.adjust.methods, intersection_threshold, p_threshold, cell_type_threshold, plot_figure)

## Limitations

- deCS does not take tissue information (except for HCAF) into account, which may bias the annotation results
- Results depend on the chosen reference panel; performance is strongly dependent on the reference database used
- deCS fails to characterize novel cell types as a mapping-based annotation toolkit, so associations are limited to reference panel cell types
- No software version for deCS is stated in the discussion

## Evidence

- [abstract] we demonstrated the broad utility of deCS to identify trait-cell type associations in 51 human complex traits: "we demonstrated the broad utility of deCS to identify trait-cell type associations in 51 human complex traits"
- [methods] CTgene cutoff definition: "We allow the user to define the cutoff values, e.g., the top 5% genes with highest t-score/z-score as CTgenes"
- [methods] filter parameters: "p_threshold = 0.01, cell_type_threshold = 0.5"
- [methods] reference panels: "deCS collected several cell type reference panels, including BlueprintEncode, the Database of Immune Cell Expression (DICE), MonacoImmune"
- [readme] loading BlueprintEncode_main from the installed package: "# BlueprintEncode_main_t_score
data(BlueprintEncode_main)"
