---
name: leave-one-out-cross-validation-of-spatial-gene-expression
description: Use when scRNA-seq data are being mapped onto a spatial reference map
  of a small number of measured genes (e.g. 64-position in situ hybridization maps)
  and an unbiased estimate of spatial gene-expression reconstruction quality is needed.
when_to_use_negative:
- The dataset has no spatially measured reference genes or positions, so held-out
  spatial patterns cannot be scored (e.g. purely non-spatial scRNA-seq without an
  in situ map)
- The dataset is excessively large, since the article notes the optimal-transport
  computation 'can become intractable when the dataset is excessively large' — use
  landmark-position approximation instead of full LOOCV over all cell pairs
- Spatial measurements are live spatial transcriptomics data (e.g. seqFISH+, Slide-seq)
  where the goal is direct integration rather than reconstruction validated against
  a small gene panel
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SpaOTsc
  role: Constructs the cell-to-position map by solving the spatial optimal transport
    problem (spsc.transport_plan(cost_matrix)) whose transport-plan-weighted averages
    yield the spatial expression predictions being cross-validated
  repo: https://github.com/zcang/SpaOTsc
- name: pandas
  role: Loads zdata.matrix.txt and Spatial_ReferenceMap.xlsx and writes the aggregated
    LOOCV metrics CSV
- name: numpy
  role: Builds the dissimilarity/distance matrices (M, Dsc, Dspa) and computes transport-plan-weighted
    averages of single-cell expression
- name: scipy
  role: Computes cell-cell and cell-position dissimilarities (e.g. correlation/canberra
    distances) used as the cost matrix
- name: scikit-learn
  role: Computes evaluation metrics such as AUC for binarized spatial prediction patterns
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:359b1566332f1e2c308ba958363f97d8a7db26a84257a20902c5757a9b25042c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zcang/SpaOTsc
  tools:
  - SpaOTsc
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# leave-one-out-cross-validation-of-spatial-gene-expression

## Summary

Validates an scRNA-seq-to-spatial mapping produced by SpaOTsc by holding out each spatially measured gene, re-deriving the optimal transport plan, predicting the held-out gene's spatial pattern at each position, and scoring predictions with AUC, Spearman rank correlation (RS), and RMSE against the in situ reference map (e.g. the 64-position/47-gene zebrafish reference).

## When to use

Use when scRNA-seq data are being mapped onto a spatial reference map of a small number of measured genes (e.g. 64-position in situ hybridization maps) and an unbiased estimate of spatial gene-expression reconstruction quality is needed.

## When NOT to use

- The dataset has no spatially measured reference genes or positions, so held-out spatial patterns cannot be scored (e.g. purely non-spatial scRNA-seq without an in situ map)
- The dataset is excessively large, since the article notes the optimal-transport computation 'can become intractable when the dataset is excessively large' — use landmark-position approximation instead of full LOOCV over all cell pairs
- Spatial measurements are live spatial transcriptomics data (e.g. seqFISH+, Slide-seq) where the goal is direct integration rather than reconstruction validated against a small gene panel

## Inputs

- scRNA-seq gene-by-cell count matrix, e.g. zdata.matrix.txt (GSE66688, 851 cells) or GSE112294 6hpf cells (5693 cells)
- Spatial reference map of in situ hybridization measurements (e.g. Spatial_ReferenceMap.xlsx, 64 positions x 47 genes)
- Cell-to-position dissimilarity/cost matrix M between scRNA-seq cells and spatial positions
- Cell-cell dissimilarity matrix Dsc (sc_dmat)
- Spatial distance matrix Dspa among the 64 positions (is_dmat)

## Outputs

- Per-gene LOOCV prediction scores (AUC on binarized spatial patterns, Spearman rank correlation RS, RMSE) for each held-out spatial gene
- Aggregated per-dataset cross-validation metrics written to zebrafish_loocv_metrics.csv, comparable to Table 1 AUC values of 0.952 and 0.936
- Predicted spatial expression pattern per held-out gene across the 64 reference positions

## How to apply

First preprocess each scRNA-seq dataset (e.g. binarize the Seurat-tutorial zebrafish zdata.matrix.txt from GSE66688, select highly variable genes following the Seurat tutorial, apply log(1+X); for GSE112294 extract 6hpf cells, normalize to 10000 counts/cell, and logp1-transform). Build the three matrices SpaOTsc requires — the cell-to-position dissimilarity matrix M (cost_matrix), the cell-cell dissimilarity matrix Dsc (sc_dmat), and the spatial distance matrix Dspa (is_dmat) — then initialize SpaOTsc.spatial_sc and compute the mapping with spsc.transport_plan(cost_matrix). Predict each position's expression as the transport-plan-weighted average of single-cell gene expression. For leave-one-out cross-validation, hold out each spatially measured gene (47 genes in the zebrafish reference), re-derive the mapping from the remaining genes, predict the held-out gene's pattern at all 64 positions, and score it with AUC (binarized patterns), Spearman RS, and RMSE. Aggregate per-gene, per-dataset scores and compare against the reported Table 1 values (zebrafish embryo 1 AUC 0.952; zebrafish embryo 2 AUC 0.936); the larger 5693-cell GSE112294 dataset should perform only slightly inferior to the 851-cell GSE66688 dataset.

## Related tools

- **SpaOTsc** (Constructs the cell-to-position map by solving the spatial optimal transport problem (spsc.transport_plan(cost_matrix)) whose transport-plan-weighted averages yield the spatial expression predictions being cross-validated) — https://github.com/zcang/SpaOTsc
- **pandas** (Loads zdata.matrix.txt and Spatial_ReferenceMap.xlsx and writes the aggregated LOOCV metrics CSV)
- **numpy** (Builds the dissimilarity/distance matrices (M, Dsc, Dspa) and computes transport-plan-weighted averages of single-cell expression)
- **scipy** (Computes cell-cell and cell-position dissimilarities (e.g. correlation/canberra distances) used as the cost matrix)
- **scikit-learn** (Computes evaluation metrics such as AUC for binarized spatial prediction patterns)

## Examples

```
from spaotsc import SpaOTsc; spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat); gamma = spsc.transport_plan(cost_matrix)
```

## Evaluation signals

- Output CSV (zebrafish_loocv_metrics.csv) exists and contains columns for AUC, Spearman RS, and RMSE with one row per spatially measured gene per dataset
- Reproduced mean AUC for the two zebrafish datasets falls close to the reported Table 1 values of 0.952 (Z. Em. 1) and 0.936 (Z. Em. 2)
- The larger second dataset (5693 cells, GSE112294) performs only slightly inferior to the first (851 cells, GSE66688), matching the reported finding
- Each gene's prediction is derived from a mapping re-fit with that gene held out — verify no leakage of the held-out gene's spatial measurement into the cost matrix M
- Predicted patterns at all 64 positions are on the same scale/normalization as the in situ reference for valid RS and RMSE comparison

## Limitations

- Computationally intractable for excessively large datasets: re-deriving the transport plan for every held-out gene requires iterative optimal transport over all cell pairs
- Approximation quality degrades when only a small number of landmark positions are used
- The approach does not consider time delay in cell-cell communication (ligand diffusion time, intracellular cascade reacting time)
- No software version of SpaOTsc is stated in the paper; exact AUC reproduction may vary across dependency versions

## Evidence

- [abstract] cross-validation of spatial gene expression: "Four datasets are employed for cross-validation of spatial gene expression prediction"
- [methods] mapping by optimal transport: "SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem"
- [methods] preprocessing following the Seurat tutorial: "We binarized the scRNA-seq data and selected a set of highly variable genes following the same tutorial."
- [readme] transport plan API call: "spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat)"
- [discussion] computational tractability caveat: "the computation can become intractable when the dataset is excessively large"
