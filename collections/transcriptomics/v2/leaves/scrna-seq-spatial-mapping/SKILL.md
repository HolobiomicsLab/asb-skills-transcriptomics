---
name: scrna-seq-spatial-mapping
description: Use when you need to link scRNA-seq cells to spatial coordinates/positions
  via optimal transport — e.g., mapping 1297 Drosophila embryo cells onto 3039 seqFISH/in
  situ spatial positions from geometry.txt and dge_normalized.txt/bdtnp.txt inputs.
when_to_use_negative:
- The data are already spatially resolved single-cell measurements with coordinates
  per cell — SpaOTsc can be applied directly but the cell↔position mapping step is
  unnecessary.
- You only need spatial gene-expression prediction at positions without a cell-to-position
  coupling map.
- The dataset is so large that pairwise structured transport becomes computationally
  intractable even with landmark approximations.
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: SpaOTsc
  role: Implements the unbalanced + structured optimal transport solver; SpaOTsc.spatial_sc
    initializes the model and transport_plan(cost_matrix) returns γ*.
  repo: https://github.com/zcang/SpaOTsc
- name: pandas
  role: Loads 'dge_normalized.txt', 'bdtnp.txt', and 'geometry.txt' into DataFrames
    for model input.
- name: numpy
  role: Builds is_dmat, sc_dmat, and cost_matrix numpy arrays consumed by SpaOTsc.spatial_sc.
- name: POT
  role: Optimal transport backend wrapped by SpaOTsc for solving the transport problem.
- name: Scanpy
  role: Selects highly variable genes before computing dissimilarity matrices.
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:116decc92a40ef09b270eaf0e78048eac958857984a7d369142a6d1c59543920
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
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# scrna-seq-spatial-mapping

## Summary

Maps n single cells from scRNA-seq data to m spatial positions (e.g., Drosophila embryo scRNA-seq vs. in situ hybridization from Synapse syn15665609) by solving an unbalanced, structured optimal transport problem with SpaOTsc, yielding an optimal transport plan γ* that connects the two datasets.

## When to use

Use when you need to link scRNA-seq cells to spatial coordinates/positions via optimal transport — e.g., mapping 1297 Drosophila embryo cells onto 3039 seqFISH/in situ spatial positions from geometry.txt and dge_normalized.txt/bdtnp.txt inputs.

## When NOT to use

- The data are already spatially resolved single-cell measurements with coordinates per cell — SpaOTsc can be applied directly but the cell↔position mapping step is unnecessary.
- You only need spatial gene-expression prediction at positions without a cell-to-position coupling map.
- The dataset is so large that pairwise structured transport becomes computationally intractable even with landmark approximations.

## Inputs

- scRNA-seq expression matrix (e.g., 'dge_normalized.txt', rows = cells, columns = genes)
- spatial in situ hybridization expression matrix (e.g., 'bdtnp.txt')
- per-cell/per-position spatial coordinates file (e.g., 'geometry.txt') used to build Dspa
- gene-expression dissimilarity matrix M (cells × spatial positions)
- single-cell dissimilarity matrix Dsc and spatial distance matrix Dspa (numpy arrays)

## Outputs

- Optimal transport plan γ* as an n × m matrix (e.g., 1297 × 3039) of nonnegative couplings between cells and spatial positions
- Exported transport_plan.csv containing γ*

## How to apply

Load the scRNA-seq matrix (e.g., 'dge_normalized.txt'), the spatial in situ matrix ('bdtnp.txt'), and per-cell spatial coordinates ('geometry.txt') with pandas; select highly variable genes using Scanpy, then build the gene-expression dissimilarity matrix M (cells × positions), the cell–cell dissimilarity Dsc, and the spatial distance matrix Dspa. Initialize the model with SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat) and solve the unbalanced + structured optimal transport problem of Eq. (1) — with major cost term <γ, M>_F, KL weight-conservation penalties ρ·(KL(γ 1^m | ω₁)+KL(γᵀ 1ⁿ | ω₂)), and a structured distance-preserving term α·Σ L(Dsc(i,k), Dspa(j,l)) γ_ij γ_kl — via spsc.transport_plan(cost_matrix). The returned transport plan γ* (here 1297 × 3039) both encodes the cell-to-position map and is the input for downstream spatial cell–cell distance, subclustering, and signaling inference steps.

## Related tools

- **SpaOTsc** (Implements the unbalanced + structured optimal transport solver; SpaOTsc.spatial_sc initializes the model and transport_plan(cost_matrix) returns γ*.) — https://github.com/zcang/SpaOTsc
- **pandas** (Loads 'dge_normalized.txt', 'bdtnp.txt', and 'geometry.txt' into DataFrames for model input.)
- **numpy** (Builds is_dmat, sc_dmat, and cost_matrix numpy arrays consumed by SpaOTsc.spatial_sc.)
- **POT** (Optimal transport backend wrapped by SpaOTsc for solving the transport problem.)
- **Scanpy** (Selects highly variable genes before computing dissimilarity matrices.)

## Examples

```
from spaotsc import SpaOTsc; spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat); spsc.transport_plan(cost_matrix)
```

## Evaluation signals

- Output transport_plan.csv exists and has the expected n × m shape (e.g., 1297 × 3039 for the Drosophila embryo challenge data).
- All entries of γ* are nonnegative, as required for a transport plan.
- Row and column marginals of γ* approximately match the weight vectors ω₁ and ω₂, as enforced by the KL penalties in Eq. (1).
- Reconstructed spatial gene expression at positions qualitatively matches the measured in situ hybridization patterns from 'bdtnp.txt'.

## Limitations

- Computationally intractable for excessively large datasets; cell–cell distance inference requires iterative optimal transport over all cell pairs.
- Effective approximation uses only a small number of landmark positions, which trades accuracy for tractability.
- The mapping of a cell–position pair depends not only on gene expression similarity for that pair but also on the mapping of all other pairs.

## Evidence

- [methods] SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem: "SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem"
- [readme] transport_plan call in README: "# get the mapping between spatial data and scRNA-seq data
spsc.transport_plan(cost_matrix)"
- [methods] Synapse dataset provenance: "the scRNA-seq data and the spatial data (in situ hybridization) were downloaded from the Dream Single cell Transcriptomics Challenge through Synapse ID (syn15665609)10."
- [abstract] structured optimal transport in abstract: "We present SpaOTsc, a method relying on structured optimal transport to recover spatial properties of scRNA-seq data"
