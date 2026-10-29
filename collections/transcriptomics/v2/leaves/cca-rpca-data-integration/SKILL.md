---
name: cca-rpca-data-integration
description: Use when multiple single-cell transcriptomic matrices from different
  plant species must be co-embedded on a shared UMAP to test whether cell identities
  are preserved across species.
when_to_use_negative:
- When integrating datasets from a single species — ortholog-based anchoring is unnecessary;
  standard Seurat integration suffices
- When no one-to-one orthologous gene mappings are available for the species pair,
  since anchors are the basis of this workflow
- When the goal is cell-type-specific gene regulatory network inference (use MINI-EX/pySCENIC
  instead) rather than cross-species data integration
edam_operation: http://edamontology.org/operation_3934
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0780
tools:
- name: scPlant
  role: Provides crossSpecies_integrate and species_percentage functions that wrap
    ortholog mapping, SCTransform normalization, and CCA/RPCA integration
  repo: https://github.com/compbioNJU/scPlant
- name: Seurat
  role: Underlying integration engine (SCTransform, CCA/RPCA as in Stuart*, Butler*
    et al. 2019); used for DefaultAssay and DimPlot visualization
- name: R
  role: Runtime environment in which the scPlant/Seurat pipeline is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:a58bcb537a3078689c890e4111d0ecf401ed542eab510b75c7705260054080b4
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/compbioNJU/scPlant
  repo_url: https://github.com/compbioNJU/scPlant
  tools:
  - R
  - Seurat
  - scPlant
  edam_operation: http://edamontology.org/operation_3934
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cca-rpca-data-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-species integration of plant scRNA-seq datasets (e.g., Arabidopsis, rice, maize roots) using one-to-one orthologous genes as anchors, with SCTransform normalization and CCA/RPCA-based integration implemented via scPlant::crossSpecies_integrate on top of Seurat methods.

## When to use

Use when multiple single-cell transcriptomic matrices from different plant species must be co-embedded on a shared UMAP to test whether cell identities are preserved across species.

## When NOT to use

- When integrating datasets from a single species — ortholog-based anchoring is unnecessary; standard Seurat integration suffices
- When no one-to-one orthologous gene mappings are available for the species pair, since anchors are the basis of this workflow
- When the goal is cell-type-specific gene regulatory network inference (use MINI-EX/pySCENIC instead) rather than cross-species data integration

## Inputs

- List of per-species single-cell expression matrices (e.g., example_Ath, example_Osa, example_Zma)
- Species identifiers (e.g., 'Ath', 'Osa', 'Zma') for ortholog mapping
- One-to-one orthologous gene mappings used as integration anchors

## Outputs

- Integrated Seurat object with an 'SCT' default assay and a UMAP reduction
- UMAP plots grouped by 'species' and by 'seurat_clusters' (split by species)
- Bar plot of species percentage per cluster from scPlant::species_percentage

## How to apply

Load the per-species expression matrices (e.g., Arabidopsis, rice, maize root scRNA-seq) into a list and call scPlant::crossSpecies_integrate with species = c('Ath', 'Osa', 'Zma') and resolution = 0.5; the function maps one-by-one orthologous genes as anchors, normalizes each dataset with SCTransform, and integrates via CCA/RPCA as in Stuart*, Butler* et al. 2019. Set the integrated object's default assay to 'SCT' (Seurat::DefaultAssay) and confirm the integrated dimensions. Visualize with Seurat::DimPlot on the UMAP reduction grouped by 'species' (axes removed) and by 'seurat_clusters' with split.by = 'species' to compare cluster composition across species. Quantify mixing using scPlant::species_percentage(group_by = 'seurat_clusters') to compute the percentage of cells from each species per cluster and render the bar plot. The rationale is that ortholog anchoring restricts integration to conserved genes, allowing species-specific cell types to remain distinct while conserved identities (e.g., meristem cells expressing MAD2) co-cluster.

## Related tools

- **scPlant** (Provides crossSpecies_integrate and species_percentage functions that wrap ortholog mapping, SCTransform normalization, and CCA/RPCA integration) — https://github.com/compbioNJU/scPlant
- **Seurat** (Underlying integration engine (SCTransform, CCA/RPCA as in Stuart*, Butler* et al. 2019); used for DefaultAssay and DimPlot visualization)
- **R** (Runtime environment in which the scPlant/Seurat pipeline is executed)

## Examples

```
integratedObj <- scPlant::crossSpecies_integrate(matrices = list(example_Ath, example_Osa, example_Zma), species = c('Ath', 'Osa', 'Zma'), resolution = 0.5)
```

## Evaluation signals

- The integrated object file exists and its default assay is 'SCT'
- The UMAP shows co-embedded cells from all three species with no species-only clusters
- Most cell identities are consistent before and after integration, with species-specific cell types preserved
- Meristem cells from different species cluster together and express conserved marker genes such as MAD2
- species_percentage bar plot shows each cluster contains a mixture of species rather than a single-species dominance (except known species-specific types)

## Limitations

- Integration quality depends on one-to-one orthologous genes; species lacking clear orthologs may fail to integrate
- Species-specific cell types may be under-represented or lost in the shared low-dimensional space, so cluster composition must be checked with split-by-species plots
- Integration relies on Seurat's CCA/RPCA method (Stuart*, Butler* et al. 2019); results may vary with Seurat version

## Evidence

- [methods] crossSpecies_integrate usage: "integratedObj <- crossSpecies_integrate(matrices = list(example_Ath, example_Osa, example_Zma),"
- [abstract] scPlant scope: "ranging from basic data processing to advanced demands such as cell-type annotation and deconvolution, trajectory inference, cross-species data integration"
- [readme] README scope statement: "cross-species data integration and cell-type specific gene regulatory network construction."
- [methods] Seurat as integration engine: "by integrating methods of `Seurat` package."
