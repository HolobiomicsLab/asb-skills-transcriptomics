---
name: single-cell-spatial-dissimilarity-matrix-construction
description: Use when you have an scRNA-seq matrix (cells x genes) plus a spatial
  dataset with coordinates for a small set of measured genes, and need to build cost/distance
  matrices before solving a spatial optimal transport mapping.
when_to_use_negative:
- You already have a solved transport plan or cell-to-position mapping — this skill
  only prepares SpaOTsc inputs, not downstream cell_cell_distance, clustering, or
  spatial_signaling_ot steps
- The spatial dataset lacks per-position coordinates (no geometry/coordinate file),
  so a physical distance matrix is_dmat cannot be computed
- The dataset is excessively large with many cells, since SpaOTsc's optimal transport
  over all cell pairs can become computationally intractable
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2229
tools:
- name: SpaOTsc
  role: Consumes the cost_matrix, sc_dmat, and is_dmat to initialize SpaOTsc.spatial_sc
    and solve the structured/unbalanced optimal transport mapping
  repo: github.com/zcang/SpaOTsc
- name: pandas
  role: Loads dge_normalized.txt, bdtnp.txt and geometry.txt into DataFrames/arrays
    for matrix construction; SpaOTsc dependency
- name: numpy
  role: Holds the is_dmat, sc_dmat, and cost_matrix as numpy arrays; SpaOTsc dependency
- name: Scanpy
  role: Selects highly variable genes from the binarized, log(1+X)-transformed scRNA-seq
    data before dissimilarity computation
- name: scipy
  role: Supports dissimilarity/distance computations between cells and positions;
    SpaOTsc dependency
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:e3f270ac83eccbbcb4a19d2ff5149f7d8dbf15d510669a483a2f90d103551be2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Scanpy
  - SpaOTsc
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-spatial-dissimilarity-matrix-construction

## Summary

Constructs the three dissimilarity inputs required by SpaOTsc's structured/unbalanced optimal transport — a gene-expression cost matrix between scRNA-seq cells and spatial positions, a cell-cell dissimilarity matrix, and a physical distance matrix among spatial positions — so that a cell-to-position transport plan can be solved.

## When to use

Use when you have an scRNA-seq matrix (cells x genes) plus a spatial dataset with coordinates for a small set of measured genes, and need to build cost/distance matrices before solving a spatial optimal transport mapping.

## When NOT to use

- You already have a solved transport plan or cell-to-position mapping — this skill only prepares SpaOTsc inputs, not downstream cell_cell_distance, clustering, or spatial_signaling_ot steps
- The spatial dataset lacks per-position coordinates (no geometry/coordinate file), so a physical distance matrix is_dmat cannot be computed
- The dataset is excessively large with many cells, since SpaOTsc's optimal transport over all cell pairs can become computationally intractable

## Inputs

- scRNA-seq gene expression matrix (e.g. dge_normalized.txt, Dream Single cell Transcriptomics Challenge, Synapse syn15665609) as a pandas DataFrame with rows = cells, columns = genes
- Spatial in situ hybridization gene matrix (e.g. bdtnp.txt) for measured spatial genes
- Spatial coordinates file (geometry.txt) for the spatial positions (3039 positions in the Drosophila embryo dataset)

## Outputs

- cost_matrix: gene-expression dissimilarity between each scRNA-seq cell and each spatial position (n_cells x n_positions)
- sc_dmat: cell-cell dissimilarity matrix within the single-cell data
- is_dmat: physical distance matrix among the spatial positions computed from geometry.txt
- Binarized, log(1+X)-transformed, HVG-filtered expression matrices ready for SpaOTsc.spatial_sc initialization

## How to apply

Load the scRNA-seq matrix (e.g. dge_normalized.txt from the Dream Single cell Transcriptomics Challenge, Synapse ID syn15665609) and the spatial in situ matrix (bdtnp.txt) with pandas, assigning the 3039 spatial coordinates from geometry.txt. Binarize the scRNA-seq data and apply an elementwise log(1 + X) transformation, then use Scanpy to select highly variable genes, following the tutorial's binarization-plus-HVG procedure. Build the gene-expression dissimilarity between cells and positions as the cost matrix, compute the dissimilarity matrix within the single-cell data (sc_dmat), and derive the physical distance matrix among the 3039 spatial positions (is_dmat) from geometry.txt coordinates. These three inputs instantiate SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat) with the cost matrix passed to spsc.transport_plan(cost_matrix), where the optimal transport problem couples gene-profile similarity with the spatial geometry so each cell-position pair's cost depends on expression dissimilarity. Keep dimensions consistent: rows are cells in df_sc/sc_dmat and positions in is_dmat/cost_matrix.

## Related tools

- **SpaOTsc** (Consumes the cost_matrix, sc_dmat, and is_dmat to initialize SpaOTsc.spatial_sc and solve the structured/unbalanced optimal transport mapping) — github.com/zcang/SpaOTsc
- **pandas** (Loads dge_normalized.txt, bdtnp.txt and geometry.txt into DataFrames/arrays for matrix construction; SpaOTsc dependency)
- **numpy** (Holds the is_dmat, sc_dmat, and cost_matrix as numpy arrays; SpaOTsc dependency)
- **Scanpy** (Selects highly variable genes from the binarized, log(1+X)-transformed scRNA-seq data before dissimilarity computation)
- **scipy** (Supports dissimilarity/distance computations between cells and positions; SpaOTsc dependency)

## Examples

```
from spaotsc import SpaOTsc; spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat); spsc.transport_plan(cost_matrix)
```

## Evaluation signals

- Validation: downstream LOOCV metrics on the Drosophila embryo dataset should match the reported D. Em. results (AUC 0.876, RS 0.495, RMSE 0.225) within a reasonable tolerance
- is_dmat must be square with shape 3039 x 3039 for the Drosophila embryo spatial data, symmetric, with zero diagonal
- cost_matrix must have shape n_cells x n_positions with rows matching df_sc and sc_dmat dimensionality
- spatial coordinates parsed from geometry.txt must reproduce physical distances consistent with the embryo geometry used in the Dream challenge spatial data
- The solved transport plan gamma* from spsc.transport_plan(cost_matrix) should assign non-trivial mass so cell-to-position predictions are well-defined

## Limitations

- Computationally intractable for excessively large datasets; cell-cell distance inference requires iterative optimal transport over all cell pairs, though landmark positions can approximate
- No software version of SpaOTsc stated in the article
- The mapping of a cell-position pair depends on not only pairwise gene-expression similarity but also the mapping of all other pairs, so matrix-construction choices propagate globally

## Evidence

- [methods] We binarized the scRNA-seq data and selected a set of highly variable genes following the same tutorial.: "We binarized the scRNA-seq data and selected a set of highly variable genes following the same tutorial."
- [methods] SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem: "SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem"
- [discussion] the mapping of a cell-position pair depends on not only the gene expression profile similarity between this pair but also the mapping of all other pairs: "the mapping of a cell–position pair depends on not only the gene expression profile similarity between this pair but also the mapping of all other pairs"
- [readme] README minimal usage showing the three input arrays passed to SpaOTsc: "spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat)"
- [methods] the scRNA-seq data and the spatial data (in situ hybridization) were downloaded from the Dream Single cell Transcriptomics Challenge through Synapse ID (syn15665609): "the scRNA-seq data and the spatial data (in situ hybridization) were downloaded from the Dream Single cell Transcriptomics Challenge through Synapse ID (syn15665609)"
