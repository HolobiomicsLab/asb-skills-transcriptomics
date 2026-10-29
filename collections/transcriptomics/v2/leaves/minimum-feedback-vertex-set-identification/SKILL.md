---
name: minimum-feedback-vertex-set-identification
description: Use when a directed gene regulatory network (e.g. a lineage-specific
  GRN from CEFCON) must be decomposed with network control theory to find a minimum
  set of genes whose removal breaks feedback cycles, as candidates for driver regulators.
when_to_use_negative:
- The network is undirected or not a regulatory graph — MFVS is defined on the directed
  GRN and would be meaningless on undirected co-expression graphs.
- The goal is to analyze multiple fate trajectories with bifurcations simultaneously
  — CEFCON's workflow can only handle one cell lineage at a time.
- You need to model extrinsic factors such as cell–cell communication or spatial location,
  which CEFCON's intracellular network-control framework does not consider.
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: CEFCON
  role: framework whose driver-regulator identification module solves MFVS (and MDS)
    on the constructed GRN and ranks candidates by influence score via cefcon_results.driver_regulators()
  repo: https://github.com/WPZgithub/CEFCON
- name: gurobipy
  role: integer linear programming solver used to formulate and solve the minimum
    feedback vertex set optimization problem on the directed GRN
  repo: https://pypi.org/project/gurobipy/
- name: networkx
  role: loads and represents the cell-lineage-specific GRN as a directed graph before
    ILP formulation
- name: cvxpy
  role: alternative optimization dependency listed by CEFCON for the network-control
    computations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:73e95cda887449185dd0258ce4f72f3e0296877bb2627515eecd507ae7861884
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# minimum-feedback-vertex-set-identification

## Summary

Identifies candidate driver regulators of cell fate decisions by solving the minimum feedback vertex set (MFVS) problem on a cell-lineage-specific GRN, one of the two network-control methods CEFCON uses alongside MDS to obtain control-based driver node candidates.

## When to use

Use when a directed gene regulatory network (e.g. a lineage-specific GRN from CEFCON) must be decomposed with network control theory to find a minimum set of genes whose removal breaks feedback cycles, as candidates for driver regulators.

## When NOT to use

- The network is undirected or not a regulatory graph — MFVS is defined on the directed GRN and would be meaningless on undirected co-expression graphs.
- The goal is to analyze multiple fate trajectories with bifurcations simultaneously — CEFCON's workflow can only handle one cell lineage at a time.
- You need to model extrinsic factors such as cell–cell communication or spatial location, which CEFCON's intracellular network-control framework does not consider.

## Inputs

- cell-lineage-specific GRN as a directed graph (e.g. cell_lineage_GRN.csv / networkx DiGraph)
- gene embeddings / learned attention coefficients (e.g. gene_embs.csv) for downstream influence scoring
- gene expression data from single-cell RNA-seq along a developmental trajectory (upstream GRN construction)

## Outputs

- minimum feedback vertex set: a minimal set of genes whose removal renders the directed GRN acyclic
- candidate driver regulator set (combined with MDS candidates) ranked by influence score
- driver_regulators.csv listing driver regulators with their influence scores

## How to apply

Load the constructed cell-lineage-specific GRN (e.g. cell_lineage_GRN.csv) with networkx and represent it as a directed graph. Formulate the minimum feedback vertex set problem on this directed graph as an integer linear program using gurobipy (cvxpy is an alternative solver dependency listed by CEFCON). Solve the ILP to optimality; the solution node set is the feedback-vertex-based candidate set. In CEFCON these MFVS candidates are combined with MDS candidates and then ranked by a log-transformed influence score computed from the GRN constructor's scaled attention coefficients (lambda=0.8), via cefcon_results.driver_regulators(), to produce the final ranked driver regulator list written to driver_regulators.csv.

## Related tools

- **CEFCON** (framework whose driver-regulator identification module solves MFVS (and MDS) on the constructed GRN and ranks candidates by influence score via cefcon_results.driver_regulators()) — https://github.com/WPZgithub/CEFCON
- **gurobipy** (integer linear programming solver used to formulate and solve the minimum feedback vertex set optimization problem on the directed GRN) — https://pypi.org/project/gurobipy/
- **networkx** (loads and represents the cell-lineage-specific GRN as a directed graph before ILP formulation)
- **cvxpy** (alternative optimization dependency listed by CEFCON for the network-control computations)

## Examples

```
import cefcon as cf, networkx as nx; G = nx.read_edgelist('cell_lineage_GRN.csv', create_using=nx.DiGraph); scores = cf.driver_regulators()
```

## Evaluation signals

- The ILP solver (gurobipy) reports the MFVS solution as optimal.
- driver_regulators.csv exists in the output directory and contains genes with influence score values.
- Removing the returned vertex set from the directed GRN eliminates all directed cycles (the set is a feedback vertex set).
- The candidate set is non-empty and its members appear in the final ranked driver regulator list produced by cefcon_results.driver_regulators().
- Influence scores attached to candidates are log-transformed values derived from scaled attention coefficients (lambda=0.8), consistent with the described ranking scheme.

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously.
- The framework only considers intracellular factors; extrinsic factors such as cell–cell communication and spatial location are not modeled.
- Epigenetic information controlling cell fate decisions is not incorporated, and the analysis is limited to RNA-seq data.
- MFVS alone yields only candidate drivers; ranking depends on the attention-derived influence score, which inherits any inaccuracy of the inferred GRN.

## Evidence

- [methods] MFVS+MDS candidate generation: [evidence span withheld — anchor status: not_found]
- [methods] influence score ranking: [evidence span withheld — anchor status: not_found]
- [methods] API step: "# Identify dirver regulators
cefcon_results.driver_regulators()"
- [intro] combining control methods with influence score: "CEFCON characterizes the cell fate dynamics based on the network control theory and identifies the driver regulators through combining the control-based methods with our proposed influence score"
- [readme] README framing of network control step: "CEFCON characterizes gene regulatory dynamics from the perspective of network control theory and identifies the driver regulators that steer cell fate decisions"
