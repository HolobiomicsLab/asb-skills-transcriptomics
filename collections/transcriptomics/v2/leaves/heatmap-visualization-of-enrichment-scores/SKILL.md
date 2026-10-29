---
name: heatmap-visualization-of-enrichment-scores
description: Use when you have pre-ranked differential gene tables for every cluster
  in an integrated scRNA-seq dataset and need to assign cell type identities/states
  from enrichment of curated signature gene sets (e.g., the 75 hematopoiesis GMT sets).
when_to_use_negative:
- When no curated signature gene sets exist for the biological system — the article
  notes 'the user would have to compile and curate customized gene sets for the relevant
  system of interest'
- When clusters have not yet been defined or differential genes per cluster have not
  been pre-ranked by log2FC — the fGSEA ranking step requires cluster-level DE tables
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: SingCellaR
  role: R package providing identifyGSEAPrerankedGenes_for_all_clusters, get_gmtGeneSets,
    Run_fGSEA_for_multiple_comparisons, and plot_heatmap_for_fGSEA_all_clusters for
    enrichment-based cluster annotation
  repo: github.com/supatt-lab/SingCellaR
- name: R
  role: Runtime environment in which the SingCellaR annotation workflow is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.xpro.2022.101266
    title: Processing single-cell RNA-seq datasets using SingCellaR
  source_content_hash: sha256:c0888627e0d4145630e1d6e6c7cef401924e60e757d86272b02d71892f654789
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SingCellaR
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# heatmap-visualization-of-enrichment-scores

## Summary

Visualize fgsea-computed normalized enrichment scores (NES) for curated gene sets across all scRNA-seq clusters as a cluster-by-gene-set heatmap, enabling systematic cell type annotation of integrated clusters (e.g., human HSPCs) in SingCellaR.

## When to use

Use when you have pre-ranked differential gene tables for every cluster in an integrated scRNA-seq dataset and need to assign cell type identities/states from enrichment of curated signature gene sets (e.g., the 75 hematopoiesis GMT sets).

## When NOT to use

- When no curated signature gene sets exist for the biological system — the article notes 'the user would have to compile and curate customized gene sets for the relevant system of interest'
- When clusters have not yet been defined or differential genes per cluster have not been pre-ranked by log2FC — the fGSEA ranking step requires cluster-level DE tables

## Inputs

- SingCellaR object with integrated clusters and cluster.type assignments
- Per-cluster differentially expressed gene tables pre-ranked by log2 fold-change (identifyGSEAPrerankedGenes_for_all_clusters output list)
- Curated GMT gene-set file (e.g., 75 hematopoiesis gene sets)

## Outputs

- NES and enrichment p-value matrix (clusters x gene sets) from fGSEA
- fgsea_nes_heatmap.png — heatmap of NES for each gene set across all clusters used for cell type annotation

## How to apply

First pre-rank differentially expressed genes for each cluster by log2 fold-change using identifyGSEAPrerankedGenes_for_all_clusters on the SingCellaR object, applying min.expFraction and min.log2FC filters. Verify the gene-set universe with get_gmtGeneSets(gmt.file) / listAvailableGeneSets(gmt.file) against the curated 75-set hematopoiesis GMT file. Run Run_fGSEA_for_multiple_comparisons(GSEAPrerankedGenes_list, gmt.file, minSize, maxSize, eps, nPermSimple, n.seed) to compute NES and enrichment p-values for every cluster against every gene set. Finally, plot the resulting NES matrix across all clusters with plot_heatmap_for_fGSEA_all_clusters and save it as fgsea_nes_heatmap.png; annotate each cluster by the gene set(s) with the strongest positive enrichment, consistent with the pre-ranked log2FC ordering.

## Related tools

- **SingCellaR** (R package providing identifyGSEAPrerankedGenes_for_all_clusters, get_gmtGeneSets, Run_fGSEA_for_multiple_comparisons, and plot_heatmap_for_fGSEA_all_clusters for enrichment-based cluster annotation) — github.com/supatt-lab/SingCellaR
- **R** (Runtime environment in which the SingCellaR annotation workflow is executed)

## Examples

```
Run_fGSEA_for_multiple_comparisons(GSEAPrerankedGenes_list, gmt.file, minSize, maxSize, eps, nPermSimple, n.seed); plot_heatmap_for_fGSEA_all_clusters(object, output_file='fgsea_nes_heatmap.png')
```

## Evaluation signals

- fgsea_nes_heatmap.png exists on disk and renders a complete cluster-by-gene-set NES matrix covering all clusters and all 75 gene sets
- Enrichment directions in the heatmap are consistent with the pre-ranked log2FC ordering of genes for each cluster
- Gene-set universe from get_gmtGeneSets(gmt.file) matches the curated GMT file contents (75 hematopoiesis sets)
- Each cluster's assigned identity is justified by the gene set(s) with the strongest positive NES in its heatmap column/row
- fGSEA outputs include both NES and enrichment p-values for every cluster-gene-set pair (no missing entries)

## Limitations

- SingCellaR requires signature gene sets to perform cell type annotation; only 75 pre-curated hematopoiesis gene sets are provided, so other systems require user curation
- Enrichment directions can vary with seed number settings (n.seed / n.seed-dependent variability noted for related analyses)
- SingCellaR lacks automatic object transformation to interoperate with other packages such as Seurat

## Evidence

- [discussion] A heatmap of GSEA enrichment scores for each gene set across all clusters allows the user to annotate a cell type identity and cell states to each cluster: [evidence span withheld — anchor status: not_found]
- [discussion] SingCellaR requires signature gene sets to perform the cell type annotation analysis.: "SingCellaR requires signature gene sets to perform the cell type annotation analysis."
- [discussion] In this protocol, we provide 75 gene sets curated from previous studies relevant to hematopoiesis.: "In this protocol, we provide 75 gene sets curated from previous studies relevant to hematopoiesis."
- [readme] an integrative analysis tool for analysing single-cell RNA sequencing data: "an integrative analysis tool for analysing single-cell RNA sequencing data"
