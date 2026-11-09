---
name: orthologous-gene-anchoring
description: Use when scRNA-seq datasets from different plant species must be integrated
  onto a shared UMAP/embedding while keeping cell identities consistent and detecting
  species-specific or conserved (e.g. meristem) cell types.
when_to_use_negative:
- Input is single-species scRNA-seq data — ordinary scPlant/Seurat processing (readData,
  QCfun, RunSeurat) is the right path, not crossSpecies_integrate
- Species pairs lack one-to-one orthologous genes sufficient for anchoring, so the
  ortholog map cannot support CCA/RPCA integration
- The goal is cell-type-specific gene regulatory network construction or trajectory
  inference within one species rather than cross-species integration
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPlant
  role: Provides crossSpecies_integrate, which maps one-to-one orthologous gene anchors,
    applies SCTransform normalization per dataset, and integrates via CCA/RPCA; also
    provides species_percentage for per-cluster species composition.
  repo: https://github.com/compbioNJU/scPlant
- name: Seurat
  role: Underlying integration engine (SCTransform, CCA/RPCA as in Stuart*, Butler*
    et al. 2019); used for DefaultAssay inspection and DimPlot visualization of the
    integrated object.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:bfc30bac09fba44044ac74b60e8191fb758965e9e841b0150df6f53c350d36ab
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
  - Seurat
  - scPlant
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# orthologous-gene-anchoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cross-species single-cell integration that anchors one-to-one orthologous genes between plant species (e.g. Arabidopsis, rice, maize) so their scRNA-seq data can be co-embedded while preserving cell identity, as implemented in scPlant's crossSpecies_integrate built on Seurat's CCA/RPCA integration.

## When to use

Use when scRNA-seq datasets from different plant species must be integrated onto a shared UMAP/embedding while keeping cell identities consistent and detecting species-specific or conserved (e.g. meristem) cell types.

## When NOT to use

- Input is single-species scRNA-seq data — ordinary scPlant/Seurat processing (readData, QCfun, RunSeurat) is the right path, not crossSpecies_integrate
- Species pairs lack one-to-one orthologous genes sufficient for anchoring, so the ortholog map cannot support CCA/RPCA integration
- The goal is cell-type-specific gene regulatory network construction or trajectory inference within one species rather than cross-species integration

## Inputs

- Named list of per-species single-cell expression matrices (e.g. example_Ath, example_Osa, example_Zma scRNA-seq count matrices)
- Species identifiers (e.g. 'Ath', 'Osa', 'Zma') used to map one-to-one orthologous genes as anchors
- Genome/annotation resources for orthology mapping (e.g. TAIR10 for Arabidopsis, Oryza sativa V7.0, Zm-B73-REFERENCE-NAM-5.0 via EnsemblPlants/Phytozome)

## Outputs

- Integrated Seurat object with an 'SCT' default assay and a shared UMAP reduction co-embedding cells from all species
- UMAP plots grouped by 'species' and by 'seurat_clusters' (split.by = 'species')
- Bar plot of species percentages per cluster from scPlant::species_percentage(group_by = 'seurat_clusters')

## How to apply

Load the expression matrices of the species to integrate (e.g. Arabidopsis, rice, maize roots) into a list in R. Call scPlant::crossSpecies_integrate with species = c('Ath', 'Osa', 'Zma') and resolution = 0.5: the function maps one-by-one orthologous genes as anchors, normalizes each dataset with SCTransform, and integrates via CCA/RPCA following Stuart*, Butler* et al. 2019. Set Seurat::DefaultAssay(integratedObj) to 'SCT' and confirm the integrated object dimensions. Visualize with Seurat::DimPlot on the UMAP reduction grouped by 'species' and by 'seurat_clusters' (split.by = 'species'), and quantify species composition per cluster with scPlant::species_percentage(group_by = 'seurat_clusters'). Rationale: orthologous gene anchors restrict the integration space to genes homologous across species, so cells of the same identity from different species co-embed without forcing species-only clusters.

## Related tools

- **scPlant** (Provides crossSpecies_integrate, which maps one-to-one orthologous gene anchors, applies SCTransform normalization per dataset, and integrates via CCA/RPCA; also provides species_percentage for per-cluster species composition.) — https://github.com/compbioNJU/scPlant
- **Seurat** (Underlying integration engine (SCTransform, CCA/RPCA as in Stuart*, Butler* et al. 2019); used for DefaultAssay inspection and DimPlot visualization of the integrated object.)

## Examples

```
integratedObj <- crossSpecies_integrate(matrices = list(example_Ath, example_Osa, example_Zma), species = c('Ath','Osa','Zma'), resolution = 0.5)
```

## Evaluation signals

- The integrated object file exists and Seurat::DefaultAssay(integratedObj) returns 'SCT'
- The UMAP shows cells from all three species co-embedded with no species-only clusters
- Most cell identities are consistent before and after integration when comparing 'seurat_clusters' split by 'species'
- species_percentage output shows each cluster contains cells from multiple species, with species-specific cell types preserved where expected
- Conserved marker genes such as MAD2 are expressed in the meristem cells that cluster together across species

## Limitations

- Integration relies on one-to-one orthologous genes; species without clear one-to-one orthologs cannot be anchored
- Cell identity preservation is empirical — most, but not all, identities may remain consistent, and species-specific cell types may not co-embed
- Results depend on the clustering resolution parameter (e.g. resolution = 0.5 in the reference workflow)

## Evidence
