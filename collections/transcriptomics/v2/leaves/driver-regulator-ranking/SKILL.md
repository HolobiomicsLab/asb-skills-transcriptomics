---
name: driver-regulator-ranking
description: Use when a cell-lineage-specific GRN has already been constructed and
  attention-based gene embeddings are available, and you must produce a ranked driver-regulator
  list using network control theory (MFVS + MDS candidates scored by influence score).
when_to_use_negative:
- The input spans multiple developmental fate trajectories with bifurcations — CEFCON
  can only handle one cell lineage at a time.
- The question requires extrinsic factors such as cell–cell communication, spatial
  location, or epigenetic regulation — the framework models only intracellular RNA-seq
  factors.
- No cell-lineage-specific GRN or attention coefficients are available yet (e.g.,
  raw scRNA-seq counts only) — run GRN construction first.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CEFCON
  role: provides the influence score computation and driver_regulators() API that
    combine MFVS/MDS candidates with attention-based scores
  repo: https://github.com/WPZgithub/CEFCON
- name: gurobipy
  role: solves the MFVS and MDS integer linear programs on the directed GRN to obtain
    the candidate node sets
- name: networkx
  role: represents the cell-lineage-specific GRN as a directed graph for the control-based
    analyses
- name: cvxpy
  role: dependency of CEFCON used for the control-theory optimizations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:0cd0cac883b063bc6547c3e80dd4e8298c3cd60b658dd28379a8a01e02b89555
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WPZgithub/CEFCON
  tools:
  - CEFCON
  - Python
  - cvxpy
  - gurobipy
  - networkx
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# driver-regulator-ranking

## Summary

Ranks candidate driver regulators of cell fate decisions by combining control-based node sets — minimum feedback vertex set (MFVS) and minimum dominating set (MDS) solutions on the cell-lineage-specific GRN — with a log-transformed, scaled influence score derived from the GNN attention coefficients (lambda=0.8).

## When to use

Use when a cell-lineage-specific GRN has already been constructed and attention-based gene embeddings are available, and you must produce a ranked driver-regulator list using network control theory (MFVS + MDS candidates scored by influence score).

## When NOT to use

- The input spans multiple developmental fate trajectories with bifurcations — CEFCON can only handle one cell lineage at a time.
- The question requires extrinsic factors such as cell–cell communication, spatial location, or epigenetic regulation — the framework models only intracellular RNA-seq factors.
- No cell-lineage-specific GRN or attention coefficients are available yet (e.g., raw scRNA-seq counts only) — run GRN construction first.

## Inputs

- cell_lineage_GRN.csv — cell-lineage-specific GRN inferred by the CEFCON graph attention network
- gene_embs.csv — gene embeddings carrying learned attention coefficients
- CEFCON results object (produced by get_cefcon_results(edge_threshold_avgDegree=8))

## Outputs

- driver_regulators.csv — ranked driver regulator list with influence score values per gene

## How to apply

Load the constructed cell-lineage-specific GRN (cell_lineage_GRN.csv) with networkx/pandas and represent it as a directed graph. Formulate the minimum feedback vertex set (MFVS) problem on the directed GRN as an integer linear program and solve it with gurobipy, then do the same for the minimum dominating set (MDS) problem; the union of these control-based node sets forms the candidate pool. For each gene, compute the influence score from the learned attention coefficients of the GRN constructor (cefcon_results.gene_influence_score()), applying log transformation and scaling with lambda=0.8. Rank the MFVS/MDS candidates by this influence score and export via cefcon_results.driver_regulators() to driver_regulators.csv in the output directory.

## Related tools

- **CEFCON** (provides the influence score computation and driver_regulators() API that combine MFVS/MDS candidates with attention-based scores) — https://github.com/WPZgithub/CEFCON
- **gurobipy** (solves the MFVS and MDS integer linear programs on the directed GRN to obtain the candidate node sets)
- **networkx** (represents the cell-lineage-specific GRN as a directed graph for the control-based analyses)
- **cvxpy** (dependency of CEFCON used for the control-theory optimizations)

## Examples

```
cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8); cefcon_results.gene_influence_score(); cefcon_results.driver_regulators()
```

## Evaluation signals

- driver_regulators.csv exists in the output directory and contains genes paired with influence score values
- The Gurobi ILP solver reports both the MFVS and MDS solutions as optimal
- Influence scores are log-transformed values derived from scaled attention coefficients with lambda=0.8
- The ranked list members are drawn from the union of the MFVS and MDS candidate node sets, ordered by influence score
- Identified driver regulators reproduce known lineage regulators on the mouse hematopoietic stem cell differentiation benchmark (three developmental lineages)

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously.
- Only intracellular factors are modeled; cell–cell communication, spatial location, and epigenetics are not incorporated.
- Limited to RNA-seq inputs; extension to spatial transcriptomics, chromatin accessibility, or multi-omics is future work.

## Evidence

- [intro] identify driver regulators by combining control-based node sets with influence scores: "identifies the driver regulators through combining the control-based methods with our proposed influence score"
- [methods] influence score API: "cefcon_results.gene_influence_score()"
- [methods] driver regulator API: "# Identify dirver regulators
cefcon_results.driver_regulators()"
- [readme] network control theory identification from README: "Subsequently, CEFCON characterizes gene regulatory dynamics from the perspective of network control theory and identifies the driver regulators that steer cell fate decisions (Fig. c)."
