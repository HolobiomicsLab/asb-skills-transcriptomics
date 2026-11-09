---
name: sctransform-normalization
description: Use when raw single-cell count matrices from one or more plant samples
  (e.g. Arabidopsis, Oryza sativa, Zea mays roots) must be variance-stabilized and
  normalized with SCTransform prior to PCA/UMAP, clustering, or CCA/RPCA-based cross-species
  integration.
when_to_use_negative:
- Input is already an SCT-normalized or integrated Seurat object — re-normalizing
  would distort the corrected counts.
- The analysis target is bulk RNA-seq deconvolution or gene regulatory network construction,
  where the scPlant pipeline uses other modules rather than SCTransform normalization.
- Data are non-plant single-cell datasets where scPlant's plant-specific ortholog/genome
  resources (TAIR10, Oryza sativa V7.0, Zm-B73-REFERENCE-NAM-5.0) do not provide anchors.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPlant
  role: Wraps and orchestrates SCTransform normalization within readData/RunSeurat
    and crossSpecies_integrate for plant scRNA-seq data
  repo: https://github.com/compbioNJU/scPlant
- name: Seurat
  role: Underlying package providing SCTransform normalization, SCT assay handling,
    DimPlot, and CCA/RPCA integration methods
  repo: https://github.com/compbioNJU/scPlant
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.xplc.2023.100631
    title: 'scPlant: A versatile framework for single-cell transcriptomic data analysis
      in plants'
  source_content_hash: sha256:dfd171aecd099ec8532cf59b3c1ee7a2bb0986f6131c7194b0bc080102daed24
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
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# sctransform-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalize single-plant scRNA-seq expression matrices with SCTransform (as implemented via the Seurat package inside scPlant) before embedding and cross-species integration, e.g. as the per-dataset normalization step of scPlant::crossSpecies_integrate on Arabidopsis, rice, and maize root data.

## When to use

Use when raw single-cell count matrices from one or more plant samples (e.g. Arabidopsis, Oryza sativa, Zea mays roots) must be variance-stabilized and normalized with SCTransform prior to PCA/UMAP, clustering, or CCA/RPCA-based cross-species integration.

## When NOT to use

- Input is already an SCT-normalized or integrated Seurat object — re-normalizing would distort the corrected counts.
- The analysis target is bulk RNA-seq deconvolution or gene regulatory network construction, where the scPlant pipeline uses other modules rather than SCTransform normalization.
- Data are non-plant single-cell datasets where scPlant's plant-specific ortholog/genome resources (TAIR10, Oryza sativa V7.0, Zm-B73-REFERENCE-NAM-5.0) do not provide anchors.

## Inputs

- Raw scRNA-seq expression matrices (e.g. Arabidopsis, Oryza sativa, Zea mays root count matrices)
- List of species identifiers for ortholog mapping (e.g. c('Ath', 'Osa', 'Zma'))
- One-by-one orthologous gene anchors across the species

## Outputs

- SCTransform-normalized Seurat object (default assay = 'SCT')
- Integrated Seurat object (integratedObj) with a UMAP reduction co-embedding all species
- Cluster assignments (seurat_clusters) and per-cluster species-percentage summaries

## How to apply

Load each raw expression matrix into R via scPlant (e.g. readData with type = 'expMatrix') so it becomes a Seurat object, then run the SCTransform normalization that scPlant wraps from the Seurat package — in the cross-species workflow this happens inside scPlant::crossSpecies_integrate, which maps one-by-one orthologous genes as anchors, normalizes each dataset with SCTransform, and integrates via CCA/RPCA (Stuart*, Butler* et al. 2019) at resolution = 0.5 with species = c('Ath', 'Osa', 'Zma'). After integration, switch the Seurat default assay to 'SCT' with Seurat::DefaultAssay(integratedObj) before computing reduced dimensions. Rationale: SCTransform regularized negative-binomial normalization removes sequencing-depth effects so that co-embedding across species reflects biology rather than library statistics, which is why the validation requires the integrated object to carry an 'SCT' default assay and show co-embedded cells from all three species on the UMAP.

## Related tools

- **scPlant** (Wraps and orchestrates SCTransform normalization within readData/RunSeurat and crossSpecies_integrate for plant scRNA-seq data) — https://github.com/compbioNJU/scPlant
- **Seurat** (Underlying package providing SCTransform normalization, SCT assay handling, DimPlot, and CCA/RPCA integration methods) — https://github.com/compbioNJU/scPlant

## Examples

```
integratedObj <- crossSpecies_integrate(matrices = list(example_Ath, example_Osa, example_Zma), species = c('Ath','Osa','Zma'), resolution = 0.5)
```

## Evaluation signals

- The integrated object file exists and contains an 'SCT' default assay (Seurat::DefaultAssay(integratedObj) == 'SCT').
- The UMAP shows co-embedded cells from all three species (Arabidopsis, rice, maize) with no species-only clusters.
- Cluster composition via scPlant::species_percentage(group_by = 'seurat_clusters') shows overlapping species contributions in conserved clusters (e.g. meristem cells clustering together).
- Conserved marker genes such as MAD2 are expressed in the co-clustered meristem cells.
- Most cell identities are consistent before and after integration, with species-specific cell types preserved.

## Limitations

- Integration relies on one-by-one orthologous genes as anchors, so species lacking good ortholog annotation (e.g. genomes outside TAIR10, Oryza sativa V7.0, Zm-B73-REFERENCE-NAM-5.0 from EnsemblPlants/Phytozome) cannot be integrated reliably.
- SCTransform parameters (e.g. resolution = 0.5 in the demonstrated workflow) were tuned for the Arabidopsis/rice/maize root atlases and may need adjustment for other tissues or species pairs.
- scPlant has no changelog or discussion-section guidance in the article, so parameter behavior across versions is not documented.

## Evidence
