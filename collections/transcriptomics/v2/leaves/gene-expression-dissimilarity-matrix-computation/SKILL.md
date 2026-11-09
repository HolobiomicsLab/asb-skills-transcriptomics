---
name: gene-expression-dissimilarity-matrix-computation
description: Use when connecting an scRNA-seq dataset (e.g. zebrafish GSE66688/GSE112294
  or Drosophila dge_normalized.txt) to spatial gene measurements and you must supply
  cost_matrix, sc_dmat, and is_dmat to SpaOTsc.
when_to_use_negative:
- The dataset is excessively large, since iterative optimal transport over all cell
  pairs becomes computationally intractable (approximation via a small number of landmark
  positions is the fallback)
- There are no spatial measurements of even a small number of genes to align with
  the scRNA-seq data, so no meaningful M can be formed
- The task only needs clustering or downstream spatial-transcriptomic analysis of
  already spatially resolved data (e.g. seqFISH+) without a scRNA-seq-to-space mapping
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SpaOTsc
  role: Consumes M, Dsc, and Dspa to compute the structured unbalanced optimal transport
    plan between cells and spatial positions
  repo: github.com/zcang/SpaOTsc
- name: pandas
  role: Loads zdata.matrix.txt, Spatial_ReferenceMap.xlsx, dge_normalized.txt, bdtnp.txt,
    and geometry.txt into DataFrames used to build the dissimilarity matrices
- name: numpy
  role: Holds the cost_matrix, sc_dmat, and is_dmat arrays passed to SpaOTsc.spatial_sc
- name: scipy
  role: SpaOTsc dependency used for the distance/dissimilarity computations underlying
    M, Dsc, and Dspa
- name: Scanpy
  role: Selection of highly variable genes before computing the dissimilarity matrices
    for the Drosophila data
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:0eda5594a5578ad5f696305166d0729eb9492c6c46cf971e4620840839bc70b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - POT
  - Scanpy
  - SpaOTsc
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# gene-expression-dissimilarity-matrix-computation

## Summary

Computes the gene-expression dissimilarity matrix M (cells vs. spatial positions), the cell-cell dissimilarity matrix Dsc, and the spatial distance matrix Dspa that serve as the cost and structured-distance inputs for SpaOTsc's unbalanced structured optimal transport plan. Correct construction of these matrices determines the fidelity of the cell-to-position mapping.

## When to use

Use when connecting an scRNA-seq dataset (e.g. zebrafish GSE66688/GSE112294 or Drosophila dge_normalized.txt) to spatial gene measurements and you must supply cost_matrix, sc_dmat, and is_dmat to SpaOTsc.

## When NOT to use

- The dataset is excessively large, since iterative optimal transport over all cell pairs becomes computationally intractable (approximation via a small number of landmark positions is the fallback)
- There are no spatial measurements of even a small number of genes to align with the scRNA-seq data, so no meaningful M can be formed
- The task only needs clustering or downstream spatial-transcriptomic analysis of already spatially resolved data (e.g. seqFISH+) without a scRNA-seq-to-space mapping

## Inputs

- Binarized / log(1+X)-normalized scRNA-seq gene-by-cell matrix (e.g. zdata.matrix.txt from GSE66688, 851 cells; 6hpf cells from GSE112294, 5693 cells)
- Spatial gene measurement matrix (e.g. Spatial_ReferenceMap.xlsx, 64-position in situ map; bdtnp.txt Drosophila in situ hybridization matrix)
- Cell/position coordinates file (geometry.txt) for computing Dspa
- Set of selected highly variable genes following the Seurat tutorial

## Outputs

- Gene-expression dissimilarity matrix M (n_cells x m_positions) usable as cost_matrix
- Cell-cell dissimilarity matrix Dsc (n_cells x n_cells)
- Spatial distance matrix Dspa (m_positions x m_positions)
- Optimal transport plan gamma* (e.g. 1297 x 3039 for the Drosophila embryo) from spsc.transport_plan(cost_matrix)

## How to apply

Preprocess expression so cells and spatial positions are comparable: for the zebrafish datasets binarize the scRNA-seq data, select highly variable genes following the Seurat tutorial, and apply an elementwise log(1 + X) transform; for GSE112294 6hpf cells normalize to 10000 total counts per cell then logp1-transform. From these aligned matrices compute M as the dissimilarity between each of the n cells and each of the m spatial positions (e.g. 1297 cells vs 3039 Drosophila in situ positions), Dsc as the cell-cell dissimilarity, and Dspa from the geometry.txt coordinates. Initialize the model with SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat) and pass M as cost_matrix to spsc.transport_plan(cost_matrix); SpaOTsc then solves the unbalanced, structured OT problem whose cost term is <gamma, M>_F with KL weight penalties and the structured term alpha*sum L(Dsc(i,k), Dspa(j,l)) gamma_ij gamma_kl. The rationale is that the mapping of a cell-position pair depends both on expression-profile similarity and on the mapping of all other pairs, so M, Dsc, and Dspa jointly encode expression and geometry.

## Related tools

- **SpaOTsc** (Consumes M, Dsc, and Dspa to compute the structured unbalanced optimal transport plan between cells and spatial positions) — github.com/zcang/SpaOTsc
- **pandas** (Loads zdata.matrix.txt, Spatial_ReferenceMap.xlsx, dge_normalized.txt, bdtnp.txt, and geometry.txt into DataFrames used to build the dissimilarity matrices)
- **numpy** (Holds the cost_matrix, sc_dmat, and is_dmat arrays passed to SpaOTsc.spatial_sc)
- **scipy** (SpaOTsc dependency used for the distance/dissimilarity computations underlying M, Dsc, and Dspa)
- **Scanpy** (Selection of highly variable genes before computing the dissimilarity matrices for the Drosophila data)

## Examples

```
spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat); spsc.transport_plan(cost_matrix)
```

## Evaluation signals

- The exported transport plan has the expected shape (e.g. 1297 x 3039 for Drosophila) with nonnegative entries
- Row and column marginals of gamma* approximately match the weight vectors omega_1 and omega_2 as required by the unbalanced KL penalties
- Leave-one-out cross-validation on the zebrafish datasets reproduces AUC close to the reported Table 1 values of 0.952 (851-cell GSE66688 pair) and 0.936 (5693-cell GSE112294 pair)
- Predicted spatial gene expression at the 64 zebrafish positions, computed as the transport-plan-weighted average of single-cell expression, qualitatively matches measured in situ patterns
- Per-gene LOOCV scores (AUC for binarized patterns, Spearman RS, RMSE) are written to zebrafish_loocv_metrics.csv with all three columns populated

## Limitations

- Computation becomes intractable when the dataset is excessively large; effective approximation uses only a small number of landmark positions
- The mapping of a cell-position pair depends on gene-expression profile similarity plus the mapping of all other pairs, so a poorly normalized or poorly gene-selected M propagates globally through the transport plan
- No software version of SpaOTsc is stated in the article, so dependency versions in requirements.txt must be pinned for reproducibility

## Evidence

- [methods] SpaOTsc constructs a mapping between cells and spatial positions: "SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem"
- [readme] cost_matrix is the gene-expression dissimilarity between single cells and spatial locations: "a numpy array for dissimilarity between single-cell data and spatial data ``cost_matrix``"
- [readme] Initialization call couples the three matrices: "spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat)"
- [methods] Preprocessing decision point before matrix construction: "We binarized the scRNA-seq data and selected a set of highly variable genes following the same tutorial."
