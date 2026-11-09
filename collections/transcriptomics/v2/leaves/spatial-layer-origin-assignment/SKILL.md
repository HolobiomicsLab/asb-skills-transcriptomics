---
name: spatial-layer-origin-assignment
description: Use when you need to recover the spatial origin (e.g., cortical layer
  L1–L6) of individual cells from non-spatial scRNA-seq data by mapping them onto
  a spatially resolved dataset with layer annotations via optimal transport.
when_to_use_negative:
- The scRNA-seq data already contains spatial coordinates or the spatial data lacks
  layer/region annotations to transport onto
- The spatial and transcriptomic datasets share too few genes to build a meaningful
  common space (direct correlation between the 1020 STARmap genes and scRNA-seq is
  noisy without the CCA step)
- The goal is de novo spatial transcriptomics analysis (e.g., Giotto pipelines) rather
  than mapping existing scRNA-seq cells onto spatial positions
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_2229
tools:
- name: SpaOTsc
  role: Solves the structured/unbalanced optimal transport problem that maps scRNA-seq
    cells to layer-annotated spatial positions; provides spatial_sc initialization
    and transport_plan solver
  repo: https://github.com/zcang/SpaOTsc
- name: Seurat
  role: Its 'cca' utility computes a low-dimensional common space between the 1020-gene
    STARmap data and the scRNA-seq data to reduce noise in cross-platform correlation
- name: pandas
  role: Loads the scRNA-seq exon matrix CSV and the STARmap data as DataFrames
- name: numpy
  role: Builds the spatial distance matrix D_spa and dissimilarity matrices (cost_matrix,
    sc_dmat) as numpy arrays
- name: scikit-learn
  role: Computes the multiclass micro F1 score of the L1–L6 layer predictions; also
    a SpaOTsc dependency
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:5cd550abfdf207b4351f2da9fa70b96e0c3cf9ecdf42af90aac01d534ded8816
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spatial-layer-origin-assignment

## Summary

Assigns each cell in a non-spatial scRNA-seq dataset (e.g., Allen Brain Atlas mouse visual cortex scRNA-seq) to a spatial region/layer (L1–L6) by combining SpaOTsc's structured/unbalanced optimal transport mapping with layer-annotated STARmap positions, and evaluates the assignment as multiclass micro F1.

## When to use

Use when you need to recover the spatial origin (e.g., cortical layer L1–L6) of individual cells from non-spatial scRNA-seq data by mapping them onto a spatially resolved dataset with layer annotations via optimal transport.

## When NOT to use

- The scRNA-seq data already contains spatial coordinates or the spatial data lacks layer/region annotations to transport onto
- The spatial and transcriptomic datasets share too few genes to build a meaningful common space (direct correlation between the 1020 STARmap genes and scRNA-seq is noisy without the CCA step)
- The goal is de novo spatial transcriptomics analysis (e.g., Giotto pipelines) rather than mapping existing scRNA-seq cells onto spatial positions

## Inputs

- Allen Brain Atlas mouse visual cortex scRNA-seq exon matrix (mouse_VISp_2018-06-14_exon-matrix.csv; GEO: GSE66688)
- STARmap mouse visual cortex data '20180505_BY3_1kgenes' from the 'visual_1020' folder (1549 positions, 1020 genes)
- Cortical layer annotations (L1–L6) for the STARmap spatial positions

## Outputs

- Per-cell spatial layer predictions for each of the 15413 scRNA-seq cells (spatial_origin_assignment.csv)
- Multiclass micro F1 score for L1–L6 spatial-origin assignment (~0.48)
- SpaOTsc transport plan (mapping between scRNA-seq cells and spatial positions)

## How to apply

Load the Allen Brain Atlas scRNA-seq matrix mouse_VISp_2018-06-14_exon-matrix.csv with pandas and the STARmap data '20180505_BY3_1kgenes' from the 'visual_1020' folder. Because the spatial data contains only 1020 genes and direct correlation is noisy, first use Seurat's 'cca' utility to compute a low-dimensional common space and derive the cell-position dissimilarity cost_matrix and the cell-cell dissimilarity matrix D_sc; build the spatial distance matrix D_spa among the 1549 STARmap positions from their coordinates. Initialize SpaOTsc with spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat) and solve the transport problem with spsc.transport_plan(cost_matrix). Assign each of the 15413 scRNA-seq cells to a cortical layer (L1–L6) from the weighted transport mapping to layer-annotated positions, then compute the multiclass micro F1 with scikit-learn and save per-cell layer predictions (e.g., spatial_origin_assignment.csv).

## Related tools

- **SpaOTsc** (Solves the structured/unbalanced optimal transport problem that maps scRNA-seq cells to layer-annotated spatial positions; provides spatial_sc initialization and transport_plan solver) — https://github.com/zcang/SpaOTsc
- **Seurat** (Its 'cca' utility computes a low-dimensional common space between the 1020-gene STARmap data and the scRNA-seq data to reduce noise in cross-platform correlation)
- **pandas** (Loads the scRNA-seq exon matrix CSV and the STARmap data as DataFrames)
- **numpy** (Builds the spatial distance matrix D_spa and dissimilarity matrices (cost_matrix, sc_dmat) as numpy arrays)
- **scikit-learn** (Computes the multiclass micro F1 score of the L1–L6 layer predictions; also a SpaOTsc dependency)

## Examples

```
from spaotsc import SpaOTsc; spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat); spsc.transport_plan(cost_matrix)
```

## Evaluation signals

- Computed multiclass micro F1 of the L1–L6 spatial-origin assignment should be approximately 0.48 (Fig 2e,f)
- Baseline check: a majority-population predictor yields micro F1 of only 0.09, so the SpaOTsc mapping should clearly outperform it
- All 15413 scRNA-seq cells must receive a layer label among the six classes L1–L6
- The transport plan from spsc.transport_plan should distribute cells across the 1549 STARmap positions with no unassigned cells
- Output file spatial_origin_assignment.csv should contain per-cell layer predictions plus the micro F1 score

## Limitations

- Computationally intractable for excessively large datasets; cell-cell distance inference requires iterative optimal transport over all cell pairs
- Approximation is made by using only a small number of landmark positions
- The mapping depends on both gene expression profile similarity and the mapping of all other pairs, so cross-platform noise (mitigated here by the Seurat CCA step) affects accuracy
- No SpaOTsc software version is stated, complicating exact reproduction

## Evidence

- [results] SpaOTsc achieves a micro F1 score of 0.48 for predicting the spatial region of mouse visual cortex scRNA-seq cells, compared to 0.09 for a baseline model: [evidence span withheld — anchor status: not_found]
- [methods] CCA for common low-dimensional space: [evidence span withheld — anchor status: not_found]
- [abstract] SpaOTsc abstract claim: "We present SpaOTsc, a method relying on structured optimal transport to recover spatial properties of scRNA-seq data"
- [readme] README minimal usage example: "spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat); get the mapping between spatial data and scRNA-seq data; spsc.transport_plan(cost_matrix)"
- [methods] STARmap data provenance: "We used the data named “20180505_BY3_1kgenes” from the folder “visual_1020”."
