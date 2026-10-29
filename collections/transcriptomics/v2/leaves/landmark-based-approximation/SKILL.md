---
name: landmark-based-approximation
description: Use when the scRNA-seq dataset has many cells so that solving the optimal
  transport problem (Eq. 2) for every cell pair over all m spatial positions is computationally
  intractable, and an approximate n x n spatial distance matrix is acceptable.
when_to_use_negative:
- The dataset is small enough that the exact pairwise optimal transport over all m
  spatial positions is computationally feasible, so the exact Eq. (2) distances can
  be used instead of the landmark approximation
- No upstream cell–position transport plan gamma* exists yet — the approximation requires
  the SpaOTsc mapping to be solved first
- The analysis requires modeling time delay in cell–cell communication (e.g. ligand
  diffusion or intracellular cascade times), which SpaOTsc's distance-based approach
  does not consider
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SpaOTsc
  role: Provides the cell_cell_distance routine with use_landmark=True that implements
    the landmark approximation of the pairwise optimal transport distance over spatial
    positions
  repo: github.com/zcang/SpaOTsc
- name: POT
  role: Optimal transport backend used to solve each reduced coupling problem minimizing
    <gamma, Dspa>_F over the feasible set Gamma
- name: numpy
  role: Constructs the marginals from normalized rows/columns of gamma* and assembles
    the n x n distance matrix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:261099fd9a2132fa47c4c34c8cc0229a1edd62da238701e4dea000cea42e15fb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - POT
  - SpaOTsc
  - numpy
  - pandas
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# landmark-based-approximation

## Summary

Approximates the n x n spatial cell–cell distance matrix in SpaOTsc by restricting the pairwise optimal transport computation over spatial locations to a small set of landmark positions, because the full cell-pair-wise OT computation becomes intractable for excessively large datasets.

## When to use

Use when the scRNA-seq dataset has many cells so that solving the optimal transport problem (Eq. 2) for every cell pair over all m spatial positions is computationally intractable, and an approximate n x n spatial distance matrix is acceptable.

## When NOT to use

- The dataset is small enough that the exact pairwise optimal transport over all m spatial positions is computationally feasible, so the exact Eq. (2) distances can be used instead of the landmark approximation
- No upstream cell–position transport plan gamma* exists yet — the approximation requires the SpaOTsc mapping to be solved first
- The analysis requires modeling time delay in cell–cell communication (e.g. ligand diffusion or intracellular cascade times), which SpaOTsc's distance-based approach does not consider

## Inputs

- Optimal transport plan gamma* (n x m) mapping n scRNA-seq cells to m spatial positions (from spsc.transport_plan)
- Spatial distance matrix Dspa (m x m) among spatial locations, built from per-cell coordinates (e.g. 'geometry.txt' from Synapse syn15665609)
- Landmark spatial positions (small subset of the m locations used to reduce the coupling set)

## Outputs

- Approximate n x n spatial cell–cell distance matrix D_sc_hat (cell_cell_distance.csv)
- SpaOTsc object with the distance matrix available for downstream clustering (spsc.clustering()) and spatial signaling inference (spsc.spatial_signaling_ot)

## How to apply

After SpaOTsc constructs the cell–position mapping via spsc.transport_plan(cost_matrix), each cell i is represented by its spatial probability marginal derived from the rows of the transport plan gamma* (feasible set Gamma of Eq. 2), and the cell–cell distance is the optimal transport distance between these marginals under the m x m spatial distance matrix Dspa (built from per-cell coordinates, e.g. geometry.txt from Synapse syn15665609). Instead of solving this OT problem for all n(n-1)/2 cell pairs against all m positions, the landmark variant (spsc.cell_cell_distance(use_landmark=True)) restricts the coupling to a small number of landmark spatial positions, which keeps the computation tractable while effectively approximating the distances. Solve each reduced OT problem by minimizing the Frobenius inner product <gamma, Dspa>_F over the feasible coupling set, using the POT library as wrapped by SpaOTsc. Assemble the resulting distances into the n x n matrix D_sc_hat; the decision point is dataset size — for very large n this approximation is used, otherwise the exact computation can be run.

## Related tools

- **SpaOTsc** (Provides the cell_cell_distance routine with use_landmark=True that implements the landmark approximation of the pairwise optimal transport distance over spatial positions) — github.com/zcang/SpaOTsc
- **POT** (Optimal transport backend used to solve each reduced coupling problem minimizing <gamma, Dspa>_F over the feasible set Gamma)
- **numpy** (Constructs the marginals from normalized rows/columns of gamma* and assembles the n x n distance matrix)

## Examples

```
spsc.transport_plan(cost_matrix); spsc.cell_cell_distance(use_landmark=True)
```

## Evaluation signals

- Output cell_cell_distance.csv exists and is an n x n matrix (n equals the number of scRNA-seq cells)
- All entries of D_sc_hat are nonnegative, as required for a transport distance
- The matrix is symmetric, as required for a pairwise distance matrix
- Distances are bounded by the spatial scale of Dspa (values should not exceed the diameter of the spatial domain used to build Dspa)
- On a small subset of cell pairs, landmark-approximated distances should closely match the exact Eq. (2) OT distances computed without landmarks

## Limitations

- The computation becomes intractable when the dataset is excessively large, motivating the landmark approximation
- The approximation by only using a small number of landmark positions introduces error relative to the exact cell–cell distances
- The underlying approach does not consider time delay in cell–cell communication (ligand diffusion time, intracellular cascade reacting time), so distances reflect spatial proximity only

## Evidence

- [discussion] Landmark approximation necessity: "the computation can become intractable when the dataset is excessively large"
- [discussion] Landmark approximation described: "Although effective approximation was made by only using a small number of landmark positions"
- [methods] Cell–cell distance computed from transport plan: [evidence span withheld — anchor status: not_found]
- [readme] API usage of the landmark flag: "spsc.cell_cell_distance(use_landmark=True)"
