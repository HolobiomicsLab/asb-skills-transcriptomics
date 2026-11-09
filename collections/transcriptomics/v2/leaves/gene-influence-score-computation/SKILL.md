---
name: gene-influence-score-computation
description: Use when you have a CEFCON-constructed cell-lineage-specific GRN with
  learned attention coefficients and need to rank MFVS/MDS driver gene candidates
  by importance to output driver regulators.
when_to_use_negative:
- Input is raw scRNA-seq data without a CEFCON-constructed GRN — run data_preparation
  and GRN construction first
- The goal is identifying regulon-like gene modules (RGMs) and their activity rather
  than ranking driver regulators
- Multiple fate trajectories with bifurcations must be processed simultaneously —
  CEFCON handles one cell lineage at a time
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0605
tools:
- name: CEFCON
  role: Framework whose cefcon_results.gene_influence_score() and driver_regulators()
    methods compute attention-derived influence scores and rank driver regulators
  repo: https://github.com/WPZgithub/CEFCON
- name: gurobipy
  role: Solves the MFVS and MDS integer linear programs that produce control-based
    driver gene candidates on the GRN
- name: networkx
  role: Represents the cell-lineage-specific GRN as a directed graph for MFVS/MDS
    analysis
- name: cvxpy
  role: Optimization backend used in the driver regulator identification module
- name: PyTorch
  role: Backend for the graph attention network whose coefficients feed the influence
    score
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:50c71cef03a557b12eb362614e735ec81907136f6494108a339c2962a461fdb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WPZgithub/CEFCON
  tools:
  - CEFCON
  - PyTorch
  - Python
  - cvxpy
  - gurobipy
  - networkx
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# gene-influence-score-computation

## Summary

Computes a gene influence score from the attention coefficients learned by CEFCON's graph attention network when constructing the cell-lineage-specific GRN; log-transformed, scaled scores (lambda=0.8) rank candidate driver regulators obtained from MFVS and MDS network-control solutions.

## When to use

Use when you have a CEFCON-constructed cell-lineage-specific GRN with learned attention coefficients and need to rank MFVS/MDS driver gene candidates by importance to output driver regulators.

## When NOT to use

- Input is raw scRNA-seq data without a CEFCON-constructed GRN — run data_preparation and GRN construction first
- The goal is identifying regulon-like gene modules (RGMs) and their activity rather than ranking driver regulators
- Multiple fate trajectories with bifurcations must be processed simultaneously — CEFCON handles one cell lineage at a time

## Inputs

- cell_lineage_GRN.csv — directed cell-lineage-specific GRN inferred by CEFCON's graph attention network
- gene_embs.csv — gene embeddings learned by CEFCON
- learned attention coefficients from the CEFCON GRN model (cefcon_results object)

## Outputs

- gene influence scores per gene (log-transformed, scaled with lambda=0.8)
- driver_regulators.csv — ranked driver regulator list with influence score values
- MFVS and MDS candidate node sets from gurobipy ILP solutions

## How to apply

After CEFCON's graph attention network (PyTorch/torch-geometric, 350 epochs, 3 repeats) constructs the cell-lineage-specific GRN, load it with networkx/pandas (cell_lineage_GRN.csv, gene_embs.csv). Solve the minimum feedback vertex set (MFVS) and minimum dominating set (MDS) problems on the directed GRN as integer linear programs using gurobipy to obtain control-based driver gene candidates. Call cefcon_results.gene_influence_score() to derive each gene's influence score from the learned attention coefficients, applying log transformation and scaling with lambda=0.8. Then combine the MFVS/MDS node sets with these influence scores via cefcon_results.driver_regulators() to produce the final ranked driver regulator list, writing driver_regulators.csv with genes and influence score values.

## Related tools

- **CEFCON** (Framework whose cefcon_results.gene_influence_score() and driver_regulators() methods compute attention-derived influence scores and rank driver regulators) — https://github.com/WPZgithub/CEFCON
- **gurobipy** (Solves the MFVS and MDS integer linear programs that produce control-based driver gene candidates on the GRN)
- **networkx** (Represents the cell-lineage-specific GRN as a directed graph for MFVS/MDS analysis)
- **cvxpy** (Optimization backend used in the driver regulator identification module)
- **PyTorch** (Backend for the graph attention network whose coefficients feed the influence score)

## Examples

```
cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8); cefcon_results.gene_influence_score(); cefcon_results.driver_regulators()
```

## Evaluation signals

- driver_regulators.csv exists in the output directory and contains genes paired with influence score values
- Gurobi reports the MFVS and MDS ILP solutions as optimal
- Influence scores are log-transformed and scaled with lambda=0.8 (values consistent with scaled attention coefficients)
- Top-ranked driver regulators overlap the union of MFVS and MDS solution node sets from the GRN
- Identified regulators are biologically plausible for the lineage (e.g., known drivers of the three mouse hematopoietic stem cell developmental lineages)

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously
- Only intracellular factors are modeled; cell-cell communication and spatial location are not considered
- Gurobi requires a license for large-scale ILP solving (gurobipy >=10.0.0 dependency)

## Evidence

- [intro] Driver regulator identification via MFVS/MDS and influence score: "CEFCON characterizes the cell fate dynamics based on the network control theory and identifies the driver regulators through combining the control-based methods with our proposed influence score"
- [methods] driver_regulators API call: "# Identify dirver regulators
cefcon_results.driver_regulators()"
- [methods] gene_influence_score API call: "cefcon_results.gene_influence_score()"
- [methods] Solver dependencies for MFVS/MDS ILPs: "networkx(>=3.0)
- cvxpy(>=1.2.0)
- gurobipy(>=10.0.0)"
- [abstract] Attention-based GRN construction feeding influence scores: "first uses a graph neural network with attention mechanism to infer a cell-lineage-specific gene regulatory network (GRN) from single-cell RNA-sequencing data, and then models cell fate dynamics"
