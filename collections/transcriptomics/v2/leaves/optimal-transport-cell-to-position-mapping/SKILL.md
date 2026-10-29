---
name: optimal-transport-cell-to-position-mapping
description: Use when a non-spatial scRNA-seq dataset must be linked to spatial measurements
  of a small set of genes (e.g., seqFISH/in situ reference maps) to predict spatial
  expression patterns or assign cells to spatial regions.
when_to_use_negative:
- When the scRNA-seq dataset is excessively large, where the article notes computation
  can become intractable due to iterative optimal transport over all cell pairs
- When spatial data lacks shared genes with the scRNA-seq data and no low-dimensional
  common space (e.g., via Seurat cca) can be established
- When the goal is time-resolved signaling inference — the approach does not consider
  time delay in cell-cell communication
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3512
tools:
- name: SpaOTsc
  role: Solves the structured/unbalanced optimal transport problem via spatial_sc
    initialization and transport_plan(cost_matrix) to obtain the cell-to-position
    mapping
  repo: github.com/zcang/SpaOTsc
- name: pandas
  role: Loads expression matrices and reference maps (txt/csv/xlsx) into DataFrames;
    SpaOTsc dependency
- name: numpy
  role: Builds dissimilarity, cost, and spatial distance matrices; SpaOTsc dependency
- name: scipy
  role: Supports distance/dissimilarity computations; SpaOTsc dependency
- name: scikit-learn
  role: Scores spatial-origin assignment (micro F1) and LOOCV predictions; SpaOTsc
    dependency
- name: Scanpy
  role: Selects highly variable genes for the Drosophila Dream Challenge workflow
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:6f4cfb2e1934c79c9db4b0d7fa7c47c6a3163f760fa477198633e4cd7c99c216
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Scanpy
  - Seurat
  - SpaOTsc
  - dit
  - matplotlib
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8235294117647058
n_claims: 17
---

# optimal-transport-cell-to-position-mapping

## Summary

Maps individual scRNA-seq cells to spatial positions of a reference spatial transcriptomics dataset by solving a structured/unbalanced optimal transport problem in SpaOTsc, yielding a transport plan that supports spatial gene expression prediction, cell-origin assignment, and downstream cell-cell distance inference.

## When to use

Use when a non-spatial scRNA-seq dataset must be linked to spatial measurements of a small set of genes (e.g., seqFISH/in situ reference maps) to predict spatial expression patterns or assign cells to spatial regions.

## When NOT to use

- When the scRNA-seq dataset is excessively large, where the article notes computation can become intractable due to iterative optimal transport over all cell pairs
- When spatial data lacks shared genes with the scRNA-seq data and no low-dimensional common space (e.g., via Seurat cca) can be established
- When the goal is time-resolved signaling inference — the approach does not consider time delay in cell-cell communication

## Inputs

- scRNA-seq expression matrix (e.g., zdata.matrix.txt GSE66688; dge_normalized.txt from Synapse syn15665609; mouse_VISp_2018-06-14_exon-matrix.csv)
- Spatial reference map of gene measurements at positions (e.g., Spatial_ReferenceMap.xlsx, bdtnp.txt, seqFISH+ 20180505_BY3_1kgenes)
- Spatial position coordinates/distance matrix (e.g., geometry.txt for 3039 Drosophila positions)
- Cost matrix: gene-expression dissimilarity between cells and spatial positions
- Cell-cell dissimilarity matrix (sc_dmat)

## Outputs

- Optimal transport plan gamma* mapping cells to spatial positions
- Predicted spatial gene expression per position (transport-plan-weighted average)
- Cell-to-position/region assignments (e.g., L1-L6 layer predictions)
- Cross-validation metrics: AUC, Spearman RS, RMSE (e.g., zebrafish_loocv_metrics.csv, drosophila_loocv_metrics.csv)

## How to apply

Construct three matrices: the gene-expression dissimilarity (cost) matrix between scRNA-seq cells and spatial positions, the cell-cell dissimilarity matrix Dsc, and the spatial distance matrix Dspa among the reference positions (e.g., 64 zebrafish in situ positions, 3039 Drosophila bdtnp positions from geometry.txt, or 1549 STARmap positions). Initialize SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat) and call spsc.transport_plan(cost_matrix) to solve the structured optimal transport problem and obtain the mapping gamma*. Predict spatial expression at each position as the transport-plan-weighted average sum_j gamma*_j,i * g_j / sum_j gamma*_j,i, or aggregate gamma* to assign cells to spatial regions (e.g., cortical layers L1-L6). Preprocessing matters: binarize scRNA-seq data, select highly variable genes (Seurat tutorial criteria or Scanpy), and apply elementwise log(1+X) transform; for noisy gene-limited spatial data (e.g., STARmap 1020 genes) use Seurat 'cca' to build a low-dimensional common space before computing the cost matrix. Validate by leave-one-out cross-validation over spatially measured genes, scoring with AUC, Spearman rank correlation (RS), and RMSE.

## Related tools

- **SpaOTsc** (Solves the structured/unbalanced optimal transport problem via spatial_sc initialization and transport_plan(cost_matrix) to obtain the cell-to-position mapping) — github.com/zcang/SpaOTsc
- **pandas** (Loads expression matrices and reference maps (txt/csv/xlsx) into DataFrames; SpaOTsc dependency)
- **numpy** (Builds dissimilarity, cost, and spatial distance matrices; SpaOTsc dependency)
- **scipy** (Supports distance/dissimilarity computations; SpaOTsc dependency)
- **scikit-learn** (Scores spatial-origin assignment (micro F1) and LOOCV predictions; SpaOTsc dependency)
- **Scanpy** (Selects highly variable genes for the Drosophila Dream Challenge workflow)

## Examples

```
from spaotsc import SpaOTsc; spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat); spsc.transport_plan(cost_matrix)
```

## Evaluation signals

- LOOCV AUC on zebrafish datasets reproduces Table 1 values (Z. Em. 1 AUC ~0.952 for GSE66688, Z. Em. 2 AUC ~0.936 for GSE112294)
- Drosophila LOOCV reproduces AUC 0.876, Spearman RS 0.495, RMSE 0.225 (D. Em. Table 1 values)
- Mouse visual cortex L1-L6 spatial-origin assignment yields micro F1 ~0.48 (vs 0.09 majority-label baseline)
- Predicted spatial expression is computed as transport-plan-weighted average sum_j gamma*_j,i * g_j / sum_j gamma*_j,i — check this formula in the prediction code
- Output metric CSVs contain columns for AUC, Spearman RS, and RMSE

## Limitations

- Computationally intractable for excessively large datasets; approximation by using only a small number of landmark positions
- Does not consider time delay in cell-cell communication (ligand diffusion time, intracellular cascade reacting time)
- Direct correlation between scRNA-seq and spatial data with few genes (e.g., 1020 STARmap genes) is noisy — a low-dimensional common space via Seurat 'cca' is needed

## Evidence

- [abstract] structured optimal transport mapping: "We present SpaOTsc, a method relying on structured optimal transport to recover spatial properties of scRNA-seq data by utilizing spatial measurements of a relatively small number of genes"
- [methods] transport_plan call: "spsc.transport_plan(cost_matrix)"
- [methods] spatial_sc initialization: "SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem"
- [methods] binarization and HVG selection: "We binarized the scRNA-seq data and selected a set of highly variable genes following the same tutorial."
- [other] cca for STARmap: [evidence span withheld — anchor status: not_found]
- [readme] README minimal usage: "spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat)"
- [discussion] tractability limitation: "the computation can become intractable when the dataset is excessively large"
- [discussion] no time delay: "our approach does not consider the time delay that may take place in cell–cell communication"
