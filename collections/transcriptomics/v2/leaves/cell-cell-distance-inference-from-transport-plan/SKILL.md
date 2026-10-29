---
name: cell-cell-distance-inference-from-transport-plan
description: Use when you have a fitted SpaOTsc transport plan γ* (n cells × m spatial
  positions) plus a spatial position distance matrix, and you need a pairwise spatial
  cell-cell distance matrix for downstream signaling or clustering analyses.
when_to_use_negative:
- Input is spatial transcriptomics data where true physical cell coordinates are already
  measured directly — no transport-based distance inference is needed
- No upstream SpaOTsc transport plan γ* exists yet; transport_plan(cost_matrix) must
  be run first
- The goal is time-resolved cell-cell communication, since SpaOTsc does not consider
  time delay (ligand diffusion or intracellular cascade time)
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: SpaOTsc
  role: Provides the spatial_sc wrapper whose cell_cell_distance(use_landmark=True)
    method computes the n × n spatial cell-cell distance matrix from the fitted transport
    plan
  repo: https://github.com/zcang/SpaOTsc
- name: POT
  role: Optimal transport solver backend used by SpaOTsc to minimize <γ, Dspa>_F over
    the coupling set Γ for each cell pair
- name: numpy
  role: Holds and manipulates the γ* transport plan and Dspa spatial distance matrices
- name: pandas
  role: Loads coordinate/geometry files and writes the resulting cell-cell distance
    matrix (e.g., cell_cell_distance.csv)
- name: scipy
  role: SpaOTsc dependency supporting numerical routines in the distance computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:e8c6f2c31b8952b6962e08eb85f6a17c1649c4c0b8073b682e68b9b31661da03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zcang/SpaOTsc
  tools:
  - POT
  - SpaOTsc
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cell-cell-distance-inference-from-transport-plan

## Summary

Computes a single-cell resolution spatial distance matrix (D_sc_hat) for scRNA-seq cells by evaluating the optimal transport distance between each cell pair's spatial probability distributions derived from the SpaOTsc transport plan γ*, using the POT-wrapped solver and a landmark approximation for tractability.

## When to use

Use when you have a fitted SpaOTsc transport plan γ* (n cells × m spatial positions) plus a spatial position distance matrix, and you need a pairwise spatial cell-cell distance matrix for downstream signaling or clustering analyses.

## When NOT to use

- Input is spatial transcriptomics data where true physical cell coordinates are already measured directly — no transport-based distance inference is needed
- No upstream SpaOTsc transport plan γ* exists yet; transport_plan(cost_matrix) must be run first
- The goal is time-resolved cell-cell communication, since SpaOTsc does not consider time delay (ligand diffusion or intracellular cascade time)

## Inputs

- SpaOTsc transport plan γ* (n × m numpy array) mapping n scRNA-seq cells to m spatial positions
- Spatial distance matrix Dspa (m × m) from spatial position coordinates (e.g., geometry.txt from Synapse syn15665609 Drosophila data)
- pandas/numpy-readable single-cell and spatial data as consumed by SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat)

## Outputs

- n × n symmetric spatial cell-cell distance matrix D_sc_hat with nonnegative entries (e.g., cell_cell_distance.csv)
- Fitted SpaOTsc spatial_sc object with computed cell-cell distances available for clustering or spatial_signaling_ot

## How to apply

Load the transport plan γ* (n × m) from the upstream SpaOTsc mapping and the spatial distance matrix Dspa (m × m) built from per-cell coordinates (e.g., 'geometry.txt' from the Drosophila Dream Challenge data, Synapse syn15665609). For each pair of cells (i, j), form the two spatial probability marginals from the normalized rows and columns of γ*, matching the feasible set Γ of Eq. (2) in Methods. Solve, for each cell pair, the optimal transport problem minimizing the Frobenius inner product <γ, Dspa>_F over the coupling set Γ, using the POT library as wrapped by SpaOTsc. Because pairwise transport over all cell pairs becomes computationally intractable for excessively large datasets, apply the landmark-based approximation via cell_cell_distance(use_landmark=True), which uses only a small number of landmark positions. Assemble the resulting n × n spatial cell-cell distance matrix D_sc_hat and write it to cell_cell_distance.csv.

## Related tools

- **SpaOTsc** (Provides the spatial_sc wrapper whose cell_cell_distance(use_landmark=True) method computes the n × n spatial cell-cell distance matrix from the fitted transport plan) — https://github.com/zcang/SpaOTsc
- **POT** (Optimal transport solver backend used by SpaOTsc to minimize <γ, Dspa>_F over the coupling set Γ for each cell pair)
- **numpy** (Holds and manipulates the γ* transport plan and Dspa spatial distance matrices)
- **pandas** (Loads coordinate/geometry files and writes the resulting cell-cell distance matrix (e.g., cell_cell_distance.csv))
- **scipy** (SpaOTsc dependency supporting numerical routines in the distance computation)

## Examples

```
from spaotsc import SpaOTsc; spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat); spsc.transport_plan(cost_matrix); spsc.cell_cell_distance(use_landmark=True)
```

## Evaluation signals

- Output cell_cell_distance.csv exists and is exactly n × n, where n is the number of scRNA-seq cells
- All entries of D_sc_hat are nonnegative, as required for a distance matrix
- D_sc_hat is symmetric (D_sc_hat[i, j] == D_sc_hat[j, i])
- Diagonal entries are zero (or near-zero) since a cell's distance to itself should vanish
- Landmark approximation runs without exhausting memory on datasets where full pairwise transport would be intractable, while reproducing the ordering of distances on a small subset

## Limitations

- Computation can become intractable when the dataset is excessively large, since cell-cell distance inference requires iterative optimal transport over all cell pairs
- Landmark-based approximation trades exactness for tractability by using only a small number of landmark positions
- The approach does not consider time delay in cell-cell communication (ligand diffusion time, intracellular cascade reacting time)

## Evidence

- [methods] The spatial cell–cell distance ... is then computed based on γ* using the optimal transport distance: [evidence span withheld — anchor status: not_found]
- [discussion] landmark approximation supported by discussion: "Although effective approximation was made by only using a small number of landmark positions"
- [readme] landmark flag usage in README: "spsc.cell_cell_distance(use_landmark=True)"
- [readme] setup of the spatial_sc object from README: "spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat)"
