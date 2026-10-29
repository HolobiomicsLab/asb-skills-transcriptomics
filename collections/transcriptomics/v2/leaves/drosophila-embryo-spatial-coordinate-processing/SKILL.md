---
name: drosophila-embryo-spatial-coordinate-processing
description: Use when mapping scRNA-seq cells to spatial positions of the Drosophila
  embryo (bdtnp.txt, geometry.txt, dge_normalized.txt) and you need the dissimilarity/distance
  matrices for an unbalanced structured optimal transport plan.
when_to_use_negative:
- The spatial data is already a high-resolution spatial transcriptomics assay (e.g.
  seqFISH+, STARmap, Slide-seq) processed with its own pipeline rather than the Dream
  Challenge in situ hybridization data with geometry.txt coordinates.
- The goal is downstream inference (cell-cell communication, signal range, spatial
  GRN) and a valid transport plan already exists — use the corresponding SpaOTsc steps
  instead of rebuilding the coordinate processing.
- The dataset is excessively large, since the paper notes the computation can become
  intractable and is approximated using only a small number of landmark positions.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: SpaOTsc
  role: Provides the spatial_sc initialization and transport_plan solver for the unbalanced
    structured optimal transport mapping built on the processed coordinate and dissimilarity
    matrices
  repo: https://github.com/zcang/SpaOTsc
- name: pandas
  role: Loads dge_normalized.txt, bdtnp.txt, and geometry.txt into DataFrames/arrays
    for matrix construction
- name: Scanpy
  role: Selects highly variable genes from the binarized scRNA-seq data before computing
    dissimilarity matrices
- name: numpy
  role: Holds the spatial distance matrix is_dmat and dissimilarity arrays passed
    to SpaOTsc
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:c2bd28f86dfb76778bcdb9b87e874abc810e451b7619cd9467500ad9c203be01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zcang/SpaOTsc
  tools:
  - POT
  - Scanpy
  - SpaOTsc
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# drosophila-embryo-spatial-coordinate-processing

## Summary

Load and process the Drosophila embryo spatial in situ hybridization data and cell/position coordinates from the Dream Single Cell Transcriptomics Challenge (Synapse syn15665609) to build the spatial distance matrix Dspa and gene-expression dissimilarity matrices (M, Dsc) required by SpaOTsc's structured optimal transport mapping.

## When to use

Use when mapping scRNA-seq cells to spatial positions of the Drosophila embryo (bdtnp.txt, geometry.txt, dge_normalized.txt) and you need the dissimilarity/distance matrices for an unbalanced structured optimal transport plan.

## When NOT to use

- The spatial data is already a high-resolution spatial transcriptomics assay (e.g. seqFISH+, STARmap, Slide-seq) processed with its own pipeline rather than the Dream Challenge in situ hybridization data with geometry.txt coordinates.
- The goal is downstream inference (cell-cell communication, signal range, spatial GRN) and a valid transport plan already exists — use the corresponding SpaOTsc steps instead of rebuilding the coordinate processing.
- The dataset is excessively large, since the paper notes the computation can become intractable and is approximated using only a small number of landmark positions.

## Inputs

- dge_normalized.txt: normalized Drosophila embryo scRNA-seq count matrix (pandas DataFrame, rows=cells, columns=genes)
- bdtnp.txt: spatial in situ hybridization gene expression matrix for measured positions
- geometry.txt: cell/position spatial coordinates from the Dream Challenge data
- Highly variable gene list selected from binarized scRNA-seq data (Scanpy)

## Outputs

- M: gene-expression dissimilarity matrix between scRNA-seq cells and spatial positions (cells vs. positions)
- Dsc: dissimilarity matrix among scRNA-seq cells
- Dspa: spatial distance matrix among positions derived from geometry.txt coordinates
- gamma*: the optimal transport plan (e.g. exported as transport_plan.csv, shape 1297 x 3039) mapping cells to spatial positions

## How to apply

Download the Drosophila embryo data from Synapse ID syn15665609 and load 'dge_normalized.txt' (scRNA-seq matrix), 'bdtnp.txt' (spatial in situ hybridization matrix), and 'geometry.txt' (cell coordinates) with pandas. Select highly variable genes from the binarized scRNA-seq data using Scanpy, as done in the paper's tutorial. Compute the gene-expression dissimilarity matrix M between cells and spatial positions, the cell-cell dissimilarity matrix Dsc within the scRNA-seq data, and the spatial distance matrix Dspa from the geometry.txt coordinates. Initialize SpaOTsc with SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat) and pass the cost matrix M into spsc.transport_plan(cost_matrix), which solves the unbalanced structured OT problem of Eq. (1) with KL weight penalties (rho) and the structured distance-preserving term (alpha) to return the optimal transport plan gamma*. The plan connects individual cells to spatial positions so a spatial metric can be equipped to the scRNA-seq cells.

## Related tools

- **SpaOTsc** (Provides the spatial_sc initialization and transport_plan solver for the unbalanced structured optimal transport mapping built on the processed coordinate and dissimilarity matrices) — https://github.com/zcang/SpaOTsc
- **pandas** (Loads dge_normalized.txt, bdtnp.txt, and geometry.txt into DataFrames/arrays for matrix construction)
- **Scanpy** (Selects highly variable genes from the binarized scRNA-seq data before computing dissimilarity matrices)
- **numpy** (Holds the spatial distance matrix is_dmat and dissimilarity arrays passed to SpaOTsc)

## Examples

```
from spaotsc import SpaOTsc; spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat); spsc.transport_plan(cost_matrix)
```

## Evaluation signals

- transport_plan.csv exists with shape 1297 x 3039 (cells x positions) and all entries nonnegative
- Row and column marginals of gamma* approximately match the weight vectors omega_1 and omega_2 as required by the unbalanced KL penalties rho*(KL(gamma 1^m | omega_1)+KL(gamma^T 1^n | omega_2))
- Reconstructed spatial gene expression at positions qualitatively matches the measured in situ hybridization patterns (bdtnp.txt) as reported in the paper's Drosophila validation
- The distance matrices M, Dsc, Dspa are finite and have consistent dimensions with the numbers of cells and positions in the loaded files

## Limitations

- Computationally intractable for excessively large datasets; the cell-cell distance inference requires iterative optimal transport over all cell pairs, approximated by using only a small number of landmark positions.
- The mapping of a cell-position pair depends not only on gene expression profile similarity but also on the mapping of all other pairs via the structured term.
- No software version of SpaOTsc is stated in the article.

## Evidence

- [methods] Synapse data loading and file formats: "the scRNA-seq data and the spatial data (in situ hybridization) were downloaded from the Dream Single cell Transcriptomics Challenge through Synapse ID (syn15665609)"
- [methods] OT construction of the mapping: "SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem"
- [methods] transport_plan API call: "spsc.transport_plan(cost_matrix)"
- [readme] README minimal usage example: "spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat)"
- [abstract] Structured OT rationale: "a method relying on structured optimal transport to recover spatial properties of scRNA-seq data by utilizing spatial measurements of a relatively small number of genes"
