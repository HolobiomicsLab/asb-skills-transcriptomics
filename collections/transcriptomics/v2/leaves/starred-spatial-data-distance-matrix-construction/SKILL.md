---
name: starred-spatial-data-distance-matrix-construction
description: Use when pairing scRNA-seq data (e.g., Allen Brain Atlas mouse_VISp_2018-06-14_exon-matrix.csv)
  with a spatial dataset like STARmap and you need a pairwise distance matrix among
  spatial positions as an optimal-transport input.
when_to_use_negative:
- When the spatial data already provides a distance matrix directly, recomputing from
  coordinates is unnecessary
- When the goal is a cell-cell dissimilarity within the scRNA-seq data itself (sc_dmat)
  rather than a spatial metric on measurement positions
- When dataset size makes the full all-pairs optimal transport intractable — the article
  notes computation 'can become intractable when the dataset is excessively large'
edam_operation: http://edamontology.org/operation_2929
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: SpaOTsc
  role: Consumes the spatial distance matrix as is_dmat in spatial_sc to solve the
    structured optimal transport problem mapping scRNA-seq cells to positions
  repo: https://github.com/zcang/SpaOTsc
- name: numpy
  role: Computes the pairwise Euclidean distance matrix D_spa among spatial coordinates
- name: pandas
  role: Loads the coordinate and expression tables (e.g., STARmap and Allen Brain
    Atlas CSV files)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:dccd8fcc3013ca44c203d35f41057f9a154c411ba077ad6847c19bd3ee3d3dfa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zcang/SpaOTsc
  tools:
  - Seurat
  - SpaOTsc
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_2929
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# starred-spatial-data-distance-matrix-construction

## Summary

Builds the spatial distance matrix D_spa among spatial measurement positions (e.g., the 1549 STARmap mouse visual cortex positions from '20180505_BY3_1kgenes') from their coordinates, so it can be supplied as is_dmat to SpaOTsc's spatial_sc solver for optimal-transport mapping of scRNA-seq cells to tissue locations.

## When to use

Use when pairing scRNA-seq data (e.g., Allen Brain Atlas mouse_VISp_2018-06-14_exon-matrix.csv) with a spatial dataset like STARmap and you need a pairwise distance matrix among spatial positions as an optimal-transport input.

## When NOT to use

- When the spatial data already provides a distance matrix directly, recomputing from coordinates is unnecessary
- When the goal is a cell-cell dissimilarity within the scRNA-seq data itself (sc_dmat) rather than a spatial metric on measurement positions
- When dataset size makes the full all-pairs optimal transport intractable — the article notes computation 'can become intractable when the dataset is excessively large'

## Inputs

- Spatial coordinates of measured positions, e.g., the 1549 STARmap mouse visual cortex positions from '20180505_BY3_1kgenes' (visual_1020 folder)
- Optional companion scRNA-seq expression matrix (mouse_VISp_2018-06-14_exon-matrix.csv) used with the resulting matrix in SpaOTsc

## Outputs

- Pairwise spatial distance matrix D_spa (numpy array, 1549 x 1549 for the STARmap case) supplied as is_dmat to SpaOTsc

## How to apply

Load the spatial coordinates of the measured positions (for the mouse visual cortex case, the 1549 STARmap positions from the '20180505_BY3_1kgenes' file in the 'visual_1020' folder of STARmap Resources) and compute the pairwise Euclidean spatial distance matrix D_spa among them. This matrix is passed as the is_dmat argument to SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat). Rationale: SpaOTsc solves a structured/unbalanced optimal transport problem (spsc.transport_plan(cost_matrix)) whose cost structure requires an explicit spatial metric on the target positions; the transport plan then maps each of the 15413 scRNA-seq cells to layer-annotated positions. Correctness is judged downstream by whether the transport yields the reported spatial-origin assignment performance (micro F1 ≈ 0.48 for L1–L6 layer prediction vs 0.09 for the majority-label baseline).

## Related tools

- **SpaOTsc** (Consumes the spatial distance matrix as is_dmat in spatial_sc to solve the structured optimal transport problem mapping scRNA-seq cells to positions) — https://github.com/zcang/SpaOTsc
- **numpy** (Computes the pairwise Euclidean distance matrix D_spa among spatial coordinates)
- **pandas** (Loads the coordinate and expression tables (e.g., STARmap and Allen Brain Atlas CSV files))

## Examples

```
from spaotsc import SpaOTsc; spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat); spsc.transport_plan(cost_matrix)
```

## Evaluation signals

- The distance matrix is square and symmetric with a zero diagonal; dimensions equal the number of spatial positions (1549 x 1549 for the STARmap mouse visual cortex data)
- Downstream SpaOTsc transport_plan(cost_matrix) runs without shape mismatch between is_dmat and the cost matrix
- Assigning the 15413 scRNA-seq cells to cortical layers L1–L6 from the transport mapping yields a multiclass micro F1 of approximately 0.48
- The resulting micro F1 substantially exceeds the 0.09 majority-population-label baseline, indicating the spatial metric carries real signal
- Per-cell layer predictions can be saved (spatial_origin_assignment.csv) and inspected for non-degenerate layer distribution across L1–L6

## Limitations

- With spatial panels of few genes (e.g., STARmap's 1020 genes vs scRNA-seq), direct expression correlation is noisy, so a common low-dimensional space (e.g., Seurat 'cca') is needed before transport; the distance matrix alone does not resolve this
- Computation can become intractable for excessively large datasets; approximation via a small number of landmark positions is used instead
- The approach does not consider time delay in cell-cell communication (ligand diffusion, intracellular cascade time)

## Evidence

- [methods] SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem: "SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem"
- [methods] We used the data named "20180505_BY3_1kgenes" from the folder "visual_1020".: "We used the data named “20180505_BY3_1kgenes” from the folder “visual_1020”."
- [readme] Assume we have (1) a pandas DataFrame for single-cell data df_sc with rows being cells and columns being genes, (2) a numpy array for distance matrix among spatial locations is_dmat: "Assume we have (1) a pandas DataFrame for single-cell data ``df_sc`` with rows being cells and columns being genes, (2) a numpy array for distance matrix among spatial locations ``is_dmat``"
- [discussion] the computation can become intractable when the dataset is excessively large: "the computation can become intractable when the dataset is excessively large"
