---
name: cell-subtype-similarity-map-construction
description: Use when fine cell subtype annotations from different published references
  must be related quantitatively, or when subtypes from a reference (e.g., the PBMC
  multimodal reference) must be transferred onto tumor-derived cells and compared
  across datasets.
when_to_use_negative:
- When only rough level-1 labels are needed and no cross-reference subtype comparison
  or transfer is intended (predCellType alone suffices)
- When the input is spatial transcriptomics spots lacking single-cell resolution and
  no reference subtype map is being evaluated
- When datasets lack sufficient expressed marker genes to support classifier-based
  subtype classification, making similarity estimates unreliable
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scCancer2
  role: Provides runCellSubtypeClassify, predCellType, and similarityCalculation for
    classifier-based subtype annotation and cross-reference similarity map construction
  repo: czythu/scCancer
- name: scCancer
  role: R package loaded alongside dplyr to run the annotation workflow (suppressMessages(library(scCancer)))
  repo: czythu/scCancer
- name: Seurat
  role: Used for normalization (LogNormalize, scale factor 10000) and FindTransferAnchors/TransferData
    to move reference subtypes onto target cells
- name: dplyr
  role: Data manipulation when merging label sets and rbind-ing per-sample results
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:d3dba2bfcaae819e7ea5975e5e570a6cb2259a89088c70cba67faacb5fe115b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/czythu/scCancer
  tools:
  - R
  - Seurat
  - dplyr
  - scCancer
  - scCancer2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Cell subtype similarity map construction

## Summary

Quantitative construction of similarity maps between cell subtype labels defined in different published references by testing pretrained classifiers across scRNA-seq datasets (15 datasets, 594 samples in scCancer2), enabling cross-reference label harmonization and reference-based subtype transfer (e.g., mapping Hao et al. 2021 PBMC subtypes onto tumor samples).

## When to use

Use when fine cell subtype annotations from different published references must be related quantitatively, or when subtypes from a reference (e.g., the PBMC multimodal reference) must be transferred onto tumor-derived cells and compared across datasets.

## When NOT to use

- When only rough level-1 labels are needed and no cross-reference subtype comparison or transfer is intended (predCellType alone suffices)
- When the input is spatial transcriptomics spots lacking single-cell resolution and no reference subtype map is being evaluated
- When datasets lack sufficient expressed marker genes to support classifier-based subtype classification, making similarity estimates unreliable

## Inputs

- Preprocessed Seurat expression objects (.RDS) for each sample (e.g., Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021, four Xue2022 chunks)
- cellSubtypeTemplates.rds sub-models and cell subtype markers files
- Normalized RNA assay data from the Seurat object
- Reference cell subtype annotations (e.g., Hao et al. 2021 multimodal PBMC reference) for anchor-based transfer

## Outputs

- Per-sample fine-labels.RDS and similarity-matrix.RDS
- Integrated-finelabels.RDS (rbind across all authors and chunks)
- Integrated-similaritymatrix.RDS produced by similarityCalculation
- Enriched immune cell composition per annotated structure (e.g., TLS spots: B cells, dendritic cells, monocytes, T cells with plasmablasts, memory B cells, and dendritic cells most abundant)

## How to apply

Load each sample's preprocessed Seurat object (e.g., Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021, and the four Xue2022 chunks) from RDS files and obtain rough level-1 labels with predCellType on the normalized RNA assay, merging T.cells.CD4/T.cells.CD8 into T.cells and intersecting with the default list ('T.cells', 'Myeloid.cells', 'B.cells', 'Fibroblast', 'Endothelial'). Then run runCellSubtypeClassify per major cell type using the cellSubtypeTemplates.rds sub-models and markers files with dropout.modeling = FALSE, unknown.cutoff = 0.3, and umap.plot = FALSE, saving each sample's fine-labels.RDS and similarity-matrix.RDS. Rbind the per-sample fine labels across authors and dataset chunks, save integrated-finelabels.RDS, and run similarityCalculation to produce integrated-similaritymatrix.RDS. Reference subtypes (e.g., plasmablasts, memory B cells, dendritic cells) can additionally be transferred with Seurat FindTransferAnchors/TransferData. Similarity values derived from the trained classifiers quantify how subtype labels in one reference relate to those in another.

## Related tools

- **scCancer2** (Provides runCellSubtypeClassify, predCellType, and similarityCalculation for classifier-based subtype annotation and cross-reference similarity map construction) — czythu/scCancer
- **scCancer** (R package loaded alongside dplyr to run the annotation workflow (suppressMessages(library(scCancer)))) — czythu/scCancer
- **Seurat** (Used for normalization (LogNormalize, scale factor 10000) and FindTransferAnchors/TransferData to move reference subtypes onto target cells)
- **dplyr** (Data manipulation when merging label sets and rbind-ing per-sample results)

## Examples

```
t.results <- runCellSubtypeClassify(expr = expr, submodel.path = submodel.path, markers.path = markers.path, dropout.modeling = FALSE, unknown.cutoff = 0.3, umap.plot = FALSE); all.matrix <- similarityCalculation(all.list, annoPath)
```

## Evaluation signals

- Each sample yields both fine-labels.RDS and similarity-matrix.RDS; missing either indicates an incomplete run of runCellSubtypeClassify
- unknown.cutoff = 0.3 is respected: subtypes failing the similarity cutoff are assigned unknown rather than a forced label
- integrated-finelabels.RDS contains rbind-ed fine labels across all authors (Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021) and all four Xue2022 chunks
- integrated-similaritymatrix.RDS is produced by similarityCalculation from the integrated label list
- Downstream validation: TLS spots show the expected enriched composition (B cells, dendritic cells, monocytes, T cells; plasmablasts, memory B cells, dendritic cells most abundant), consistent with Fig. 5E

## Limitations

- No software version numbers stated for R, scCancer, Seurat, or xgboost, complicating exact reproduction
- No repository URL or accession IDs provided for the HCC datasets used, so input objects must be sourced independently
- Similarity maps depend on the pretrained classifiers built from 15 reference datasets; labels undefined in those references cannot be compared

## Evidence

- [abstract] Based on the trained classifiers, we quantitatively constructed the similarity maps between the cell subtypes defined in different references by testing on all the 15 datasets.: "Based on the trained classifiers, we quantitatively constructed the similarity maps between the cell subtypes defined in different references by testing on all the 15 datasets."
- [methods] similarityCalculation on integrated list: "all.matrix <- similarityCalculation(all.list, annoPath)"
- [methods] runCellSubtypeClassify with sub-model path: "t.results <- runCellSubtypeClassify(expr = expr,
                                    submodel.path = submodel.path,
                                    markers.path = markers.path"
- [methods] unknown cutoff threshold: "unknown.cutoff = 0.3,"
- [readme] README on similarity of subtype labels across datasets: [evidence span withheld — anchor status: external_doc]
