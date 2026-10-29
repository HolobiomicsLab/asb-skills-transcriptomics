---
name: network-control-theory-driver-regulator-identification
description: Use when scRNA-seq data along a single developmental lineage plus a prior
  gene interaction network are available and the goal is to identify the driver regulators
  (e.g., top-20 per lineage) steering that cell fate trajectory.
when_to_use_negative:
- The trajectory contains multiple fate branches or bifurcations that need simultaneous
  analysis — CEFCON can only handle one cell lineage at a time
- The question involves extrinsic control such as cell–cell communication, spatial
  location, or epigenetic regulation — CEFCON models only intracellular RNA-level
  factors
- Supervised training labels (ground-truth GRN edges) are available for the cell type
  — a supervised framework such as GENELink may yield superior results
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0749
tools:
- name: CEFCON
  role: Framework whose graph attention network infers the lineage-specific GRN and
    whose driver_regulators(), gene_influence_score(), and RGM_activity() functions
    implement control-theoretic driver regulator identification
  repo: https://github.com/WPZgithub/CEFCON
- name: gurobipy
  role: Solves the integer linear programming problem underlying driver regulator
    selection
  repo: https://pypi.org/project/gurobipy/
- name: cvxpy
  role: Dependency for the network-control-theory computations (minimum control energy
    calculations)
- name: AUCell
  role: Measures the activity of regulon-like gene modules involving identified driver
    regulators
  repo: https://github.com/aertslab/AUCell
- name: Palantir
  role: Optional trajectory tool for pseudotime/branch analysis of differentiation
    data in related workflows
  repo: https://github.com/dpeerlab/Palantir
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:a40188d7226aa36072ec6f4e8fbc2cc205b160efd2ed58cd6d0f0f29736585b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WPZgithub/CEFCON
  tools:
  - AUCell
  - CEFCON
  - Palantir
  - PyTorch
  - Python
  - cvxpy
  - gurobipy
  - networkx
  - scanpy
  - torch-geometric
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# network-control-theory-driver-regulator-identification

## Summary

Identifies driver regulators controlling cell fate decisions by modeling cell fate dynamics with network control theory on a cell-lineage-specific GRN inferred by CEFCON's graph attention network, combining control-based methods with an influence score to rank candidate driver regulators per lineage.

## When to use

Use when scRNA-seq data along a single developmental lineage plus a prior gene interaction network are available and the goal is to identify the driver regulators (e.g., top-20 per lineage) steering that cell fate trajectory.

## When NOT to use

- The trajectory contains multiple fate branches or bifurcations that need simultaneous analysis — CEFCON can only handle one cell lineage at a time
- The question involves extrinsic control such as cell–cell communication, spatial location, or epigenetic regulation — CEFCON models only intracellular RNA-level factors
- Supervised training labels (ground-truth GRN edges) are available for the cell type — a supervised framework such as GENELink may yield superior results

## Inputs

- Lineage-specific scRNA-seq expression matrix (AnnData) prepared with cefcon.data_preparation
- Prior gene interaction network in edgelist format (e.g., mouse prior network from Zenodo record 7564872)
- Gene differential expression log fold changes
- Inferred cell-lineage-specific GRN from cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)

## Outputs

- Ranked driver regulators per lineage (e.g., top-20 driver regulator sets)
- Per-gene influence scores
- Controllability score, Jaccard index, and coverage metrics comparing driver regulator sets across lineages
- Regulon-like gene modules (RGMs) and their AUCell activity matrix

## How to apply

First infer the cell-lineage-specific GRN by training CEFCON's graph attention network under contrastive learning with cf.NetModel(epochs=350, repeats=3, cuda='0'), then extract the network via get_cefcon_results(edge_threshold_avgDegree=8). Compute per-gene influence scores with gene_influence_score(), then call driver_regulators(), which formulates driver node selection as an integer linear programming problem solved with the GUROBI solver, combining control-energy minimization (network control theory) with the influence score. Optionally detect regulon-like gene modules (RGMs) containing the driver regulators and their AUCell activity matrix with RGM_activity(). For multi-lineage data (e.g., mouse HSC erythroid, granulocyte-monocyte, and lymphoid trajectories), run the procedure once per lineage and compare top-20 driver regulator sets using controllability score, Jaccard index, and coverage metrics to find common regulators (e.g., Meis1, Gata2, Jun, Fos, Mycn, Dusp1) versus lineage-specific ones (e.g., Klf1, Mpo, Mef2c).

## Related tools

- **CEFCON** (Framework whose graph attention network infers the lineage-specific GRN and whose driver_regulators(), gene_influence_score(), and RGM_activity() functions implement control-theoretic driver regulator identification) — https://github.com/WPZgithub/CEFCON
- **gurobipy** (Solves the integer linear programming problem underlying driver regulator selection) — https://pypi.org/project/gurobipy/
- **cvxpy** (Dependency for the network-control-theory computations (minimum control energy calculations))
- **AUCell** (Measures the activity of regulon-like gene modules involving identified driver regulators) — https://github.com/aertslab/AUCell
- **Palantir** (Optional trajectory tool for pseudotime/branch analysis of differentiation data in related workflows) — https://github.com/dpeerlab/Palantir

## Examples

```
cefcon_results = cf.NetModel(epochs=350, repeats=3, cuda='0').get_cefcon_results(edge_threshold_avgDegree=8); cefcon_results.gene_influence_score(); cefcon_results.driver_regulators()
```

## Evaluation signals

- Per-lineage top-20 driver regulator lists for the mouse HSC data reproduce the reported sets, with the six common regulators Meis1, Gata2, Jun, Fos, Mycn, and Dusp1 shared across erythroid, granulocyte-monocyte, and lymphoid lineages (Fig. 6c-i)
- Lineage-specific regulators such as Klf1 (erythroid), Mpo, and Mef2c appear in the expected lineage's driver set
- Controllability score, Jaccard index, and coverage metrics recomputed across the three lineages match the published Fig. 6c-i values
- The GRN edge extraction uses edge_threshold_avgDegree=8 and the ILP solves without GUROBI licensing errors
- RGM activity matrix is produced with plausible AUCell scores for the identified driver-regulator modules

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously
- Only intracellular factors are modeled; extrinsic factors such as cell–cell communication and spatial location are not considered
- Epigenetic information controlling cell fate decisions is not incorporated
- Limited to RNA-seq; extension to other modalities (spatial transcriptome, chromatin accessibility, multi-omics) remains future work
- Ground-truth label datasets for supervised alternatives are often lacking, especially for specific cell types or lineages

## Evidence

- [intro] Network control theory identifies driver regulators via influence score: "CEFCON characterizes the cell fate dynamics based on the network control theory and identifies the driver regulators through combining the control-based methods with our proposed influence score"
- [methods] Driver regulator identification call in methods: "# Identify dirver regulators
cefcon_results.driver_regulators()"
- [methods] GRN extraction threshold: "cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)"
- [discussion] Single-lineage limitation: "First, CEFCON can only handle one cell lineage at a time."
- [readme] RGM activity measured with AUCell: "Moreover, CEFCON detects gene regulatory modules (i.e., RGMs) involving the identified driver regulators and measure their activities using [AUCell]"
- [abstract] Application to mouse HSC lineages: "When applied to the mouse hematopoietic stem cell differentiation data, CEFCON successfully identified driver regulators for three developmental lineages"
