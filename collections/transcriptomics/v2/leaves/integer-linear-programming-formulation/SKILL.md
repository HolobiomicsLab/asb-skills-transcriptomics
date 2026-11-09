---
name: integer-linear-programming-formulation
description: Use when a directed gene regulatory network must be reduced to control-theoretic
  driver-node candidates, i.e., an MFVS that breaks all directed cycles and an MDS
  that covers every node are needed as optimization problems.
when_to_use_negative:
- The input graph is already acyclic (DAG), making the MFVS formulation trivial/unnecessary
- The GRN has not yet been constructed — CEFCON must first build the cell-lineage-specific
  GRN with the graph attention network before control-based candidate extraction
- The analysis targets multiple fate trajectories with bifurcations at once — CEFCON
  can only handle one cell lineage at a time
tools:
- name: gurobipy
  role: Solves the MFVS and MDS integer linear programs on the directed GRN to optimality
  repo: https://pypi.org/project/gurobipy/
- name: cvxpy
  role: Alternative optimization backend for formulating the network-control problems
    listed among CEFCON's requirements
- name: networkx
  role: Loads and represents the cell_lineage_GRN as a directed graph on which the
    ILPs are defined
- name: CEFCON
  role: Framework whose driver_regulators() function combines the ILP-derived control
    node sets with influence scores to rank driver regulators
  repo: https://github.com/WPZgithub/CEFCON
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:ca8355b30a7f402287448e499f1abce2fa1ec4d99105f95f30de27200a419a4c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CEFCON
  - Python
  - cvxpy
  - gurobipy
  - networkx
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# integer-linear-programming-formulation

## Summary

Within CEFCON's driver-regulator module, network-control candidate sets are obtained by formulating and solving the minimum feedback vertex set (MFVS) and minimum dominating set (MDS) problems on the directed cell-lineage-specific GRN as integer linear programs (ILPs) with gurobipy/cvxpy.

## When to use

Use when a directed gene regulatory network must be reduced to control-theoretic driver-node candidates, i.e., an MFVS that breaks all directed cycles and an MDS that covers every node are needed as optimization problems.

## When NOT to use

- The input graph is already acyclic (DAG), making the MFVS formulation trivial/unnecessary
- The GRN has not yet been constructed — CEFCON must first build the cell-lineage-specific GRN with the graph attention network before control-based candidate extraction
- The analysis targets multiple fate trajectories with bifurcations at once — CEFCON can only handle one cell lineage at a time

## Inputs

- cell_lineage_GRN.csv — directed cell-lineage-specific GRN (nodes = genes, edges = learned regulatory interactions)
- gene_embs.csv — gene embeddings / learned attention coefficients from the CEFCON GRN constructor, used to compute influence scores for ranking

## Outputs

- MFVS vertex set (optimal solution of the feedback vertex set ILP on the directed GRN)
- MDS vertex set (optimal solution of the dominating set ILP on the directed GRN)
- Ranked driver regulator list with influence scores (driver_regulators.csv)

## How to apply

Load the constructed cell-lineage-specific GRN (cell_lineage_GRN.csv) as a directed graph with networkx. Formulate MFVS as an integer linear program minimizing the number of selected vertices whose removal makes the directed graph acyclic, and MDS as a second ILP minimizing a vertex subset such that every node is either selected or adjacent to a selected node. Solve both ILPs with gurobipy (>=10.0.0; cvxpy >=1.2.0 is also supported by CEFCON) and confirm the solver returns optimal solutions. The MFVS and MDS solutions form the candidate driver-gene pool, which is then combined with the log-transformed, scaled attention-based influence scores (lambda=0.8) and ranked via cefcon_results.driver_regulators(). Rationale: control-based node sets from the directed GRN capture structural controllability, complementing the attention-derived influence score to identify driver regulators steering cell fate decisions.

## Related tools

- **gurobipy** (Solves the MFVS and MDS integer linear programs on the directed GRN to optimality) — https://pypi.org/project/gurobipy/
- **cvxpy** (Alternative optimization backend for formulating the network-control problems listed among CEFCON's requirements)
- **networkx** (Loads and represents the cell_lineage_GRN as a directed graph on which the ILPs are defined)
- **CEFCON** (Framework whose driver_regulators() function combines the ILP-derived control node sets with influence scores to rank driver regulators) — https://github.com/WPZgithub/CEFCON

## Examples

```
import gurobipy as gp; m = gp.Model("MFVS"); m.setObjective(gp.quicksum(x[v] for v in grn.nodes), gp.GRB.MINIMIZE); m.optimize()
```

## Evaluation signals

- The gurobipy solver reports optimal status for both the MFVS and MDS ILPs
- Removing the MFVS solution vertices from the directed GRN leaves an acyclic graph (no directed cycles remain)
- Every node of the GRN is covered by the MDS solution (dominated by itself or an adjacent selected node)
- driver_regulators.csv exists and contains genes with non-empty influence score values
- Ranked list can be reproduced via cefcon_results.driver_regulators() after cefcon_results.gene_influence_score()

## Limitations

- gurobipy (>=10.0.0) requires a license for larger models; ILP solve time can grow with GRN size
- The ILP-based control analysis is applied to a single cell lineage at a time; multiple fate trajectories with bifurcations cannot be processed simultaneously
- Only intracellular factors are modeled — cell–cell communication and spatial location are outside the scope of the control-based candidate extraction

## Evidence

- [intro] CEFCON characterizes cell fate dynamics via network control theory: "CEFCON characterizes the cell fate dynamics based on the network control theory and identifies the driver regulators through combining the control-based methods with our proposed influence score"
- [abstract] CEFCON models cell fate dynamics via network control theory to identify driver regulators: "then models cell fate dynamics through network control theory to identify driver regulators and the associated gene modules"
- [readme] gurobipy and cvxpy are required optimization dependencies: "networkx(>=3.0)
- cvxpy(>=1.2.0)
- gurobipy(>=10.0.0)"
- [readme] CEFCON combines control-based methods with the influence score for driver regulator identification: "CEFCON characterizes gene regulatory dynamics from the perspective of network control theory and identifies the driver regulators"
- [intro] GRN construction precedes the control-based analysis: "CEFCON first employs a graph attention neural network under a contrastive learning framework to construct a cell-lineage-specific GRN."
