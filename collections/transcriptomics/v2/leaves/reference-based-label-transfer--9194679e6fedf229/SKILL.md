---
name: reference-based-label-transfer
description: Use when query cells or spatial spots carry only coarse labels (e.g.,
  'B.cells', 'T.cells', 'Myeloid.cells') and a well-annotated reference atlas exists
  to resolve finer subtypes such as plasmablasts, memory B cells, or dendritic cells.
when_to_use_negative:
- The query dataset has no meaningful shared cell types with the reference (e.g.,
  tumor-only or non-hematopoietic tissue where PBMC subtypes do not map)
- Only raw counts are available without normalization or rough level-1 labels — the
  skill presupposes predCellType coarse labels and LogNormalize-normalized data
- The goal is de novo clustering/annotation of a dataset with no suitable annotated
  reference atlas
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: scCancer2
  role: Provides predCellType rough labeling, runCellSubtypeClassify sub-models, similarityCalculation,
    and the spatial transcriptomics module that detects TLS spots onto which labels
    are transferred
  repo: https://doi.org/10.5281/zenodo.10477296
- name: Seurat
  role: FindTransferAnchors/TransferData perform the anchor-based label transfer from
    the Hao et al. 2021 PBMC reference onto query cells
- name: dplyr
  role: Used in the scCancer2 R workflow for label manipulation (e.g., merging T.cells.CD4
    and T.cells.CD8 into T.cells)
- name: scCancer
  role: Underlying R toolkit whose libraries and file paths (e.g., system.file('txt',
    package='scCancer')) support the annotation workflow
  repo: https://github.com/czythu/scCancer
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bioinformatics/btae028
    title: 'scCancer2: data-driven in-depth annotations of the tumor microenvironment
      at single-level resolution'
  source_content_hash: sha256:b738e4f7cf17fe3fa8570380bd5747e72683da7cf0043c73823e6112ddca6bc4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - dplyr
  - scCancer
  - scCancer2
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# reference-based-label-transfer

## Summary

Transfer fine-grained cell subtype labels from a curated reference scRNA-seq dataset (e.g., the Hao et al. 2021 multi-modal PBMC reference) onto query cells or spatial spots using Seurat anchor-based integration, enabling subtype resolution beyond a study's own annotations.

## When to use

Use when query cells or spatial spots carry only coarse labels (e.g., 'B.cells', 'T.cells', 'Myeloid.cells') and a well-annotated reference atlas exists to resolve finer subtypes such as plasmablasts, memory B cells, or dendritic cells.

## When NOT to use

- The query dataset has no meaningful shared cell types with the reference (e.g., tumor-only or non-hematopoietic tissue where PBMC subtypes do not map)
- Only raw counts are available without normalization or rough level-1 labels — the skill presupposes predCellType coarse labels and LogNormalize-normalized data
- The goal is de novo clustering/annotation of a dataset with no suitable annotated reference atlas

## Inputs

- Preprocessed Seurat expression objects (.RDS) for each sample (e.g., Ma2022, Zhang2019, Lu2022, Zheng2017, Sun2021, Liu2023, Losic2020, Ma2021, four Xue2022 chunks)
- Normalized RNA assay (LogNormalize, scale factor 10000)
- cellSubtypeTemplates.rds sub-models and markers files
- Annotated reference scRNA-seq dataset (Hao et al. 2021 multi-modal PBMC reference)
- Detected TLS spots from scCancer2's spatial transcriptomics module

## Outputs

- Per-sample fine-labels.RDS with subtype annotations
- Per-sample similarity-matrix.RDS
- Integrated fine labels across samples (integrated-finelabels.RDS)
- Integrated similarity matrix (integrated-similaritymatrix.RDS)
- Enriched immune cell composition of TLS spots (e.g., B cells, dendritic cells, monocytes, T cells; plasmablasts, memory B cells, dendritic cells most abundant)

## How to apply

First obtain rough level-1 labels on the normalized RNA assay with scCancer2's predCellType, merging T.cells.CD4 and T.cells.CD8 into T.cells and intersecting with the default list ('T.cells', 'Myeloid.cells', 'B.cells', 'Fibroblast', 'Endothelial'). Then assign fine subtype labels per major type with runCellSubtypeClassify using the cellSubtypeTemplates.rds sub-models, markers files, dropout.modeling = FALSE, unknown.cutoff = 0.3, and umap.plot = FALSE, saving each sample's fine-labels.RDS and similarity-matrix.RDS. Next, use Seurat FindTransferAnchors/TransferData to project subtypes from the Hao et al. (2021) multi-modal PBMC reference onto the query cells, mapping subtypes such as plasmablasts, memory B cells, and dendritic cells. Rbind per-sample fine labels across all authors and Xue2022 chunks into integrated-finelabels.RDS, then run similarityCalculation to produce integrated-similaritymatrix.RDS. For spatial applications, map the transferred annotations onto TLS spots detected by scCancer2's spatial module and compute the enriched immune cell composition per spot.

## Related tools

- **scCancer2** (Provides predCellType rough labeling, runCellSubtypeClassify sub-models, similarityCalculation, and the spatial transcriptomics module that detects TLS spots onto which labels are transferred) — https://doi.org/10.5281/zenodo.10477296
- **Seurat** (FindTransferAnchors/TransferData perform the anchor-based label transfer from the Hao et al. 2021 PBMC reference onto query cells)
- **dplyr** (Used in the scCancer2 R workflow for label manipulation (e.g., merging T.cells.CD4 and T.cells.CD8 into T.cells))
- **scCancer** (Underlying R toolkit whose libraries and file paths (e.g., system.file('txt', package='scCancer')) support the annotation workflow) — https://github.com/czythu/scCancer

## Examples

```
t.results <- runCellSubtypeClassify(expr = expr, submodel.path = submodel.path, markers.path = markers.path, dropout.modeling = FALSE, unknown.cutoff = 0.3, umap.plot = FALSE); all.matrix <- similarityCalculation(all.list, annoPath)
```

## Evaluation signals

- TLS spots show the expected enriched composition — B cells, dendritic cells, monocytes, and T cells, with plasmablasts, memory B cells, and dendritic cells most abundant — consistent with Fig. 5E
- Rough labels after merging/intersection are limited to the expected set: 'T.cells', 'Myeloid.cells', 'B.cells', 'Fibroblast', 'Endothelial'
- runCellSubtypeClassify runs with unknown.cutoff = 0.3 produce fine labels plus a similarity matrix for every sample; all samples' fine-labels.RDS and similarity-matrix.RDS exist before integration
- Rbind of per-sample labels across all authors and the four Xue2022 chunks succeeds, and similarityCalculation yields integrated-similaritymatrix.RDS without missing samples

## Limitations

- No software version numbers stated for R, scCancer, Seurat, or xgboost, so exact reproducibility of the transfer step is uncertain
- No repository URL or accession IDs are provided for the HCC scRNA-seq datasets, so query data acquisition must be resolved separately
- Subtype labels are only as good as the reference: transfer from the PBMC reference maps immune subtypes but does not resolve non-immune lineages beyond the sub-model templates

## Evidence

- [methods] Rough label annotation with predCellType: "t.results <- predCellType(expr@assays[["RNA"]]@data)
expr$Cell.Type <- t.results$type.pred"
- [methods] Fine label annotation with runCellSubtypeClassify: "t.results <- runCellSubtypeClassify(expr = expr,
                                    submodel.path = submodel.path,
                                    markers.path = markers.path"
- [methods] unknown.cutoff parameter: "unknown.cutoff = 0.3,"
- [methods] Similarity calculation across integrated labels: "all.matrix <- similarityCalculation(all.list, annoPath)"
- [methods] Dataset composition used as query inputs: "authors <- c("Ma2022", "Zhang2019", "Lu2022", "Zheng2017",
             "Sun2021", "Liu2023", "Losic2020", "Ma2021")"
- [abstract] Spatial transcriptomics module enabling TLS spot annotation: "scCancer2 integrated a module to process the spatial transcriptomic data and analyze the spatial features of TME."
- [abstract] Similarity maps between cell subtypes across references: "Based on the trained classifiers, we quantitatively constructed the similarity maps between the cell subtypes defined in different references by testing on all the 15 datasets."
