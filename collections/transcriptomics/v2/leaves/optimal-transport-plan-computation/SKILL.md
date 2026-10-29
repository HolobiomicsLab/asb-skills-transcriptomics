---
name: optimal-transport-plan-computation
description: Use when integrating non-spatial scRNA-seq data with spatial transcriptomics
  (e.g., in situ hybridization, seqFISH+) and a cell-to-position mapping is needed
  to recover spatial properties of single cells.
when_to_use_negative:
- Input is a purely spatial single-cell dataset with coordinates for every cell already
  — SpaOTsc-integration or direct spatial analysis is more appropriate than solving
  the cross-modality transport problem
- The scRNA-seq and spatial datasets share no genes or measurable expression similarity,
  since the cost matrix M cannot be constructed
- Very large datasets where the iterative transport computation becomes intractable
  without landmark approximation
edam_operation: http://edamontology.org/operation_3898
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3316
tools:
- name: SpaOTsc
  role: Provides spatial_sc initialization and the transport_plan solver for the unbalanced
    structured optimal transport problem of Eq. (1)
  repo: https://github.com/zcang/SpaOTsc
- name: pandas
  role: Loads 'dge_normalized.txt', 'bdtnp.txt', and 'geometry.txt' into DataFrames
    used to construct the cost and dissimilarity matrices
- name: numpy
  role: Builds the numpy arrays for cost_matrix, is_dmat, and sc_dmat passed to SpaOTsc.spatial_sc
- name: POT
  role: Optimal transport backend that SpaOTsc depends on for solving the transport
    problem
- name: Scanpy
  role: Selects highly variable genes from the scRNA-seq data before computing dissimilarity
    matrices
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:0c48d6d1a6b7e83bb5448db81d0c597ddf7244f960d0bd6dee785b8e81701eac
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
  edam_operation: http://edamontology.org/operation_3898
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# optimal-transport-plan-computation

## Summary

Computes the unbalanced, structured optimal transport plan γ* that maps scRNA-seq cells to spatial measurement positions in SpaOTsc, forming the foundation for deriving spatial cell-cell distances and downstream signaling analyses.

## When to use

Use when integrating non-spatial scRNA-seq data with spatial transcriptomics (e.g., in situ hybridization, seqFISH+) and a cell-to-position mapping is needed to recover spatial properties of single cells.

## When NOT to use

- Input is a purely spatial single-cell dataset with coordinates for every cell already — SpaOTsc-integration or direct spatial analysis is more appropriate than solving the cross-modality transport problem
- The scRNA-seq and spatial datasets share no genes or measurable expression similarity, since the cost matrix M cannot be constructed
- Very large datasets where the iterative transport computation becomes intractable without landmark approximation

## Inputs

- scRNA-seq gene expression matrix (e.g., 'dge_normalized.txt', rows=cells, cols=genes) as a pandas DataFrame
- Spatial gene expression matrix (e.g., 'bdtnp.txt' in situ hybridization data) as a pandas DataFrame
- Per-position spatial coordinates (e.g., 'geometry.txt') used to build the spatial distance matrix Dspa (m x m)
- Gene-expression dissimilarity matrix M (n cells x m positions) serving as the transport cost matrix
- Within-scRNA-seq dissimilarity matrix Dsc (n x n)
- Weight vectors omega_1 and omega_2 for the unbalanced KL penalties

## Outputs

- Optimal transport plan gamma* (1297 x 3039 for the Drosophila embryo example) mapping scRNA-seq cells to spatial positions, exported as transport_plan.csv

## How to apply

Load the Drosophila embryo data from Synapse syn15665609 ('dge_normalized.txt', 'bdtnp.txt', 'geometry.txt') with pandas, select highly variable genes, then compute the gene-expression dissimilarity matrix M (cells vs. positions), the within-scRNA-seq dissimilarity matrix Dsc, and the spatial distance matrix Dspa from coordinates. Initialize the model with SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat) and solve Eq. (1) by calling spsc.transport_plan(cost_matrix): the objective minimizes the Frobenius cost <gamma, M>_F plus KL weight-conservation penalties rho*(KL(gamma 1^m | omega_1)+KL(gamma^T 1^n | omega_2)) plus the structured term alpha*sum L(Dsc(i,k), Dspa(j,l)) gamma_ij gamma_kl, so that a cell-position pair's mapping depends on both expression similarity and the mapping of all other pairs. The rationale for the unbalanced KL penalties is to allow partial mapping when the two datasets do not have exactly matching total masses, while the structured term preserves dissimilarity relationships between datasets.

## Related tools

- **SpaOTsc** (Provides spatial_sc initialization and the transport_plan solver for the unbalanced structured optimal transport problem of Eq. (1)) — https://github.com/zcang/SpaOTsc
- **pandas** (Loads 'dge_normalized.txt', 'bdtnp.txt', and 'geometry.txt' into DataFrames used to construct the cost and dissimilarity matrices)
- **numpy** (Builds the numpy arrays for cost_matrix, is_dmat, and sc_dmat passed to SpaOTsc.spatial_sc)
- **POT** (Optimal transport backend that SpaOTsc depends on for solving the transport problem)
- **Scanpy** (Selects highly variable genes from the scRNA-seq data before computing dissimilarity matrices)

## Examples

```
from spaotsc import SpaOTsc; spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat); spsc.transport_plan(cost_matrix)
```

## Evaluation signals

- Output transport_plan.csv exists and has shape 1297 x 3039 for the Drosophila embryo dataset (n cells x m positions)
- All entries of gamma* are nonnegative, as required for a transport plan
- Row and column marginals of gamma* approximately match the weight vectors omega_1 and omega_2, as required by the unbalanced KL penalties
- Reconstructed spatial gene expression from the plan qualitatively matches the measured in situ hybridization patterns reported in the paper

## Limitations

- Computation can become intractable when the dataset is excessively large; effective approximation uses only a small number of landmark positions
- The approach does not consider time delay in cell-cell communication (ligand diffusion time, intracellular cascade reacting time)
- Mapping of a cell-position pair depends on the mapping of all other pairs, so results are sensitive to the full cost and structured-distance inputs

## Evidence

- [methods] SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem: "SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem"
- [abstract] structured optimal transport with spatial measurements of a small number of genes: "a method relying on structured optimal transport to recover spatial properties of scRNA-seq data by utilizing spatial measurements of a relatively small number of genes"
- [discussion] the mapping of a cell-position pair depends on not only the gene expression profile similarity between this pair but also the mapping of all other pairs: "the mapping of a cell–position pair depends on not only the gene expression profile similarity between this pair but also the mapping of all other pairs"
- [readme] README usage showing transport_plan call: "spsc.transport_plan(cost_matrix)"
- [readme] SpaOTsc dependencies include POT and pandas: "SpaOTsc depends on the following packages: pandas, numpy, scipy, networkx, python-igraph, louvain, POT, dit, astropy, scikit-learn, matplotlib, umap-learn."
