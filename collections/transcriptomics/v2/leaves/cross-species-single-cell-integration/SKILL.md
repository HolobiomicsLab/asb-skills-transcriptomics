---
name: cross-species-single-cell-integration
description: Use when you have scRNA-seq expression matrices from two or more plant
  species and need to test whether homologous cell types co-embed after integration
  using orthologous genes as anchors.
when_to_use_negative:
- You are analyzing a single species dataset — crossSpecies_integrate requires multiple
  species matrices anchored on orthologous genes
- Your goal is within-species dataset integration/batch correction of replicates rather
  than cross-species cell-identity comparison
- You need cell-type-specific gene regulatory network construction or trajectory inference
  — use scPlant's MINI-EX/pySCENIC or RunMonocle2/RunMonocle3/RunSlingshot modules
  instead
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPlant
  role: Provides crossSpecies_integrate (ortholog anchoring, SCTransform normalization,
    CCA/RPCA integration) and species_percentage for quantifying species composition
    per cluster
  repo: https://github.com/compbioNJU/scPlant
- name: Seurat
  role: Underlying integration engine; provides SCTransform, CCA/RPCA integration,
    DimPlot, and DefaultAssay access on the integrated object
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:2a47f9e9923f2670a454d38bb2e1f416e6c2c1219b2ce025619f6dfad016389c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-species-single-cell-integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates plant single-cell transcriptomic datasets from multiple species (e.g., Arabidopsis, rice, maize roots) using scPlant::crossSpecies_integrate, which anchors datasets on one-to-one orthologous genes, SCTransform-normalizes each dataset, and integrates via CCA/RPCA in the Seurat framework to test whether cell identities are preserved across species.

## When to use

Use when you have scRNA-seq expression matrices from two or more plant species and need to test whether homologous cell types co-embed after integration using orthologous genes as anchors.

## When NOT to use

- You are analyzing a single species dataset — crossSpecies_integrate requires multiple species matrices anchored on orthologous genes
- Your goal is within-species dataset integration/batch correction of replicates rather than cross-species cell-identity comparison
- You need cell-type-specific gene regulatory network construction or trajectory inference — use scPlant's MINI-EX/pySCENIC or RunMonocle2/RunMonocle3/RunSlingshot modules instead

## Inputs

- Per-species scRNA-seq expression matrices (e.g., Arabidopsis thaliana, Oryza sativa, Zea mays root scRNA-seq count matrices loaded into R)
- Species codes vector, e.g., c('Ath', 'Osa', 'Zma')
- One-to-one orthologous gene mappings derived from genome/gff3 files (TAIR10, Oryza sativa V7.0, Zm-B73-REFERENCE-NAM-5.0)

## Outputs

- Integrated Seurat object (integratedObj) with a 'SCT' default assay and UMAP reduction
- UMAP plots grouped by 'species' and by 'seurat_clusters' with split.by = 'species'
- Bar plot of per-species cell percentage per cluster from scPlant::species_percentage(group_by = 'seurat_clusters')

## How to apply

Load the per-species scRNA-seq expression matrices into R and pass them as a list to scPlant::crossSpecies_integrate with the species codes (e.g., species = c('Ath', 'Osa', 'Zma')) and a clustering resolution such as 0.5. The function maps one-by-one orthologous genes as anchors, normalizes each dataset with SCTransform, and integrates via CCA/RPCA as in Stuart*, Butler* et al. 2019 (Seurat integration), leveraging genome/gff3 resources such as TAIR10, Oryza sativa V7.0, and Zm-B73-REFERENCE-NAM-5.0 for orthology mapping. After integration, set Seurat::DefaultAssay(integratedObj) to 'SCT' and confirm the integrated object dimensions. Visualize with Seurat::DimPlot on the UMAP reduction grouped by 'species' and by 'seurat_clusters' split by species to compare cluster composition across species. Quantify mixing with scPlant::species_percentage(group_by = 'seurat_clusters') rendered as a bar plot, and check that conserved marker genes (e.g., MAD2 in meristem cells) are expressed in co-clustered cells.

## Related tools

- **scPlant** (Provides crossSpecies_integrate (ortholog anchoring, SCTransform normalization, CCA/RPCA integration) and species_percentage for quantifying species composition per cluster) — https://github.com/compbioNJU/scPlant
- **Seurat** (Underlying integration engine; provides SCTransform, CCA/RPCA integration, DimPlot, and DefaultAssay access on the integrated object)

## Examples

```
integratedObj <- crossSpecies_integrate(matrices = list(example_Ath, example_Osa, example_Zma), species = c('Ath', 'Osa', 'Zma'), resolution = 0.5)
```

## Evaluation signals

- Integrated object file exists and Seurat::DefaultAssay(integratedObj) is set to 'SCT' with confirmed integrated dimensions
- UMAP shows co-embedded cells from all species with no species-only clusters (successful mixing)
- Most cell identities are consistent before and after integration, with species-specific cell types preserved
- species_percentage bar plot shows every cluster contains cells from multiple species
- Co-clustered meristem cells express conserved marker genes such as MAD2

## Limitations

- Integration relies on one-to-one orthologous genes, so species lacking high-quality genome annotations or orthology information may lose anchors
- Clustering outcome depends on the chosen resolution parameter (e.g., 0.5 in the demonstrated workflow)
- scPlant is available for non-commercial purposes only

## Evidence
