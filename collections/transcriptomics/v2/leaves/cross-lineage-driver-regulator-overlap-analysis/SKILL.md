---
name: cross-lineage-driver-regulator-overlap-analysis
description: Use when CEFCON has been run independently per lineage on scRNA-seq trajectory
  data and you need to determine which driver regulators (e.g., top-20 per lineage)
  are common versus lineage-specific, and how overlapping sets affect control metrics.
when_to_use_negative:
- When only a single cell fate trajectory is available — CEFCON can only handle one
  cell lineage at a time and the cross-lineage overlap is undefined without at least
  two independently inferred driver sets
- When cell fate trajectories bifurcate within one dataset and must be processed simultaneously,
  since CEFCON cannot process multiple fate trajectories with bifurcations at once
  and each lineage must be analyzed separately first
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CEFCON
  role: Infers each cell-lineage-specific GRN and identifies per-lineage driver regulators
    via driver_regulators() whose sets are then compared across lineages
  repo: https://github.com/WPZgithub/CEFCON
- name: gurobipy
  role: Solves the integer linear programming problem underlying driver_regulators()
    for each lineage
- name: Python
  role: Implementation language for the CEFCON package used to run per-lineage analyses
    and compute overlap metrics
  repo: https://github.com/WPZgithub/CEFCON
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:3e681a8395ceb2ff52bd410b369be1e7e376a6887be9ceb3935e49a679398b9d
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
  - scanpy
  - torch-geometric
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# cross-lineage-driver-regulator-overlap-analysis

## Summary

Compares the top-N driver regulator sets identified by CEFCON across separate cell fate trajectories (erythroid, granulocyte-monocyte, lymphoid in mouse hematopoietic stem cell differentiation) to find shared and lineage-specific regulators, quantified via controllability score, Jaccard index, and coverage metrics (Fig. 6c-i).

## When to use

Use when CEFCON has been run independently per lineage on scRNA-seq trajectory data and you need to determine which driver regulators (e.g., top-20 per lineage) are common versus lineage-specific, and how overlapping sets affect control metrics.

## When NOT to use

- When only a single cell fate trajectory is available — CEFCON can only handle one cell lineage at a time and the cross-lineage overlap is undefined without at least two independently inferred driver sets
- When cell fate trajectories bifurcate within one dataset and must be processed simultaneously, since CEFCON cannot process multiple fate trajectories with bifurcations at once and each lineage must be analyzed separately first

## Inputs

- Per-lineage scRNA-seq expression matrices (from Zenodo record 7564872) prepared with cefcon.data_preparation
- Mouse prior gene interaction network in edgelist format
- Gene differential expression log fold changes
- Per-lineage inferred cell-lineage-specific GRNs (get_cefcon_results with edge_threshold_avgDegree=8)
- Per-lineage driver regulator ranked lists (top-20 sets)

## Outputs

- Intersection of top-20 driver regulator sets across lineages (six common regulators: Meis1, Gata2, Jun, Fos, Mycn, Dusp1)
- Lineage-specific driver regulators (e.g., Klf1, Mpo, Mef2c)
- Controllability scores per lineage driver set
- Jaccard index between lineage driver sets
- Coverage metrics corresponding to Fig. 6c-i

## How to apply

First run CEFCON separately for each lineage: train the graph attention model with NetModel(epochs=350, repeats=3, cuda='0'), extract each GRN with get_cefcon_results(edge_threshold_avgDegree=8), compute gene_influence_score(), and identify driver regulators per lineage with driver_regulators() solved as an integer linear programming problem using the GUROBI solver. Then take the top-20 driver regulator list from each lineage and intersect them to find common regulators; in the mouse HSC analysis six genes with similar expression trends along pseudotime — Meis1, Gata2, Jun, Fos, Mycn, and Dusp1 — were shared across all three fate directions, while Klf1, Mpo, and Mef2c appeared as lineage-specific. Finally recompute the controllability score for each top-N set and quantify set similarity with the Jaccard index and coverage to reproduce the per-lineage comparison corresponding to Fig. 6c-i.

## Related tools

- **CEFCON** (Infers each cell-lineage-specific GRN and identifies per-lineage driver regulators via driver_regulators() whose sets are then compared across lineages) — https://github.com/WPZgithub/CEFCON
- **gurobipy** (Solves the integer linear programming problem underlying driver_regulators() for each lineage)
- **Python** (Implementation language for the CEFCON package used to run per-lineage analyses and compute overlap metrics) — https://github.com/WPZgithub/CEFCON

## Examples

```
cefcon_results.driver_regulators()
```

## Evaluation signals

- The six regulators Meis1, Gata2, Jun, Fos, Mycn, and Dusp1 appear in the top-20 driver regulator list of all three lineages (erythroid, granulocyte-monocyte, lymphoid)
- Per-lineage top-20 driver regulator lists reproduce the published sets in Fig. 6c-i
- Lineage-specific regulators such as Klf1, Mpo, and Mef2c are recovered in the expected lineage only
- Recomputed controllability score, Jaccard index, and coverage metrics match the reported values for the per-lineage comparison
- The six common regulators show similar expression trends along pseudotime in each lineage

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously, so each trajectory must be analyzed independently before overlap comparison
- Identified driver regulators reflect only intracellular factors; cell-cell communication and spatial location are not modeled
- Epigenetic information controlling cell fate decisions is not incorporated, which may affect which regulators are ranked as drivers

## Evidence

- [abstract] driver regulators for three developmental lineages: "When applied to the mouse hematopoietic stem cell differentiation data, CEFCON successfully identified driver regulators for three developmental lineages"
- [intro] control-based identification of driver regulators: "CEFCON characterizes the cell fate dynamics based on the network control theory and identifies the driver regulators through combining the control-based methods with our proposed influence score"
- [methods] driver_regulators call: "# Identify dirver regulators
cefcon_results.driver_regulators()"
- [readme] RGM activity via AUCell: "CEFCON detects gene regulatory modules (i.e., RGMs) involving the identified driver regulators and measure 
their activities using [AUCell](https://github.com/aertslab/AUCell)"
