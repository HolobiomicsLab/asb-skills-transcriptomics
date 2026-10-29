---
name: reprogramming-trajectory-network-comparison
description: Use when single-cell GRN Links objects exist for ordered cell-state clusters
  along a reprogramming trajectory and you need per-cluster topology metrics (centrality,
  cartography, specific TF edge strengths) to compare network configurations between
  states.
when_to_use_negative:
- When you only need to infer GRN edges from single-cell transcriptome and epigenome
  data (use CellOracle GRN inference instead) rather than compare topology across
  already-inferred cluster networks
- When the goal is in silico TF perturbation simulation (KO/overexpression) of gene
  expression changes rather than graph-theoretic network comparison
- When clusters do not form an ordered trajectory — centrality trajectory and early-vs-late
  comparisons require cluster ordering along reprogramming progress
edam_operation: http://edamontology.org/operation_0271
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0203
tools:
- name: CellOracle
  role: Provides the Links objects and network_analysis module (get_network_score,
    igraph-based centrality, cartography functions) used to compute and compare per-cluster
    GRN topology
  repo: https://github.com/morris-lab/CellOracle
- name: igraph
  role: R graph library backend used by CellOracle for degree/eigenvector centrality
    and network cartography computations
  repo: https://github.com/igraph/rigraph
- name: rnetcarto
  role: CRAN R package providing simulated-annealing modularity and role (cartography)
    computation used to classify connector hubs per cluster
  repo: https://github.com/cran/rnetcarto
- name: Python
  role: Runtime environment for CellOracle (developed with Python 3.8, also tested
    with 3.10)
- name: R
  role: Runtime for igraph/rnetcarto backend required by CellOracle network analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:c8dbae9057e4d4c72f2b1cb0c718ed582275ef165ec55cf13d6e0bea0ed79919
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/morris-lab/CellOracle
  repo_url: https://github.com/morris-lab/CellOracle
  tools:
  - CellOracle
  - Python
  - R
  - igraph
  - rnetcarto
  edam_operation: http://edamontology.org/operation_0271
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# reprogramming-trajectory-network-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares cluster-specific gene regulatory networks (GRNs) along a fibroblast-to-iEP reprogramming trajectory using CellOracle's igraph-based network analysis, computing Hnf4α-Foxa1 edge-strength scores, degree/eigenvector centrality trajectories, and network cartography to identify connector-hub configurations and candidate TFs such as Fos, Eno1, Hes1, Foxq1, Zfp57, Klf2, and Egr1.

## When to use

Use when single-cell GRN Links objects exist for ordered cell-state clusters along a reprogramming trajectory and you need per-cluster topology metrics (centrality, cartography, specific TF edge strengths) to compare network configurations between states.

## When NOT to use

- When you only need to infer GRN edges from single-cell transcriptome and epigenome data (use CellOracle GRN inference instead) rather than compare topology across already-inferred cluster networks
- When the goal is in silico TF perturbation simulation (KO/overexpression) of gene expression changes rather than graph-theoretic network comparison
- When clusters do not form an ordered trajectory — centrality trajectory and early-vs-late comparisons require cluster ordering along reprogramming progress

## Inputs

- Cluster-specific CellOracle Links objects (inferred GRNs) for each cluster along the fibroblast-to-iEP reprogramming trajectory
- Cluster ordering/metadata defining the reprogramming trajectory (fibroblast, early reprogramming clusters, transition stages, iEP, dead-end clusters)
- Base-GRN generated from the Mouse sci-ATAC-seq Atlas used for GRN inference

## Outputs

- Per-cluster Hnf4α-Foxa1 network score tables exported from Links.get_network_score
- Per-cluster degree centrality and eigenvector centrality values with trajectory plots
- Per-cluster network cartography classification (z-score vs participation coefficient roles, e.g. connector hubs)
- Pairwise degree-centrality comparison statistics (early cluster vs iEP; iEP vs dead-end clusters)

## How to apply

Load the cluster-specific CellOracle Links objects for each reprogramming trajectory cluster into CellOracle's network_analysis module, then export per-cluster network scores for the Hnf4α-Foxa1 edge via Links.get_network_score. Compute degree centrality and eigenvector centrality per cluster using the igraph-backed CellOracle network_analysis functions and plot centrality as a trajectory over cluster order. Run network cartography classification per cluster (igraph/rnetcarto backend) to detect the early connector-hub configuration and how it changes toward the iEP state. Finally, perform pairwise degree-centrality comparisons (early cluster vs iEP cluster, iEP cluster vs dead-end clusters) and export the statistics, validating against the qualitative patterns in Figure 1G–K where Mef2a and Klf6 connections are decommissioned in successfully converting cells.

## Related tools

- **CellOracle** (Provides the Links objects and network_analysis module (get_network_score, igraph-based centrality, cartography functions) used to compute and compare per-cluster GRN topology) — https://github.com/morris-lab/CellOracle
- **igraph** (R graph library backend used by CellOracle for degree/eigenvector centrality and network cartography computations) — https://github.com/igraph/rigraph
- **rnetcarto** (CRAN R package providing simulated-annealing modularity and role (cartography) computation used to classify connector hubs per cluster) — https://github.com/cran/rnetcarto
- **Python** (Runtime environment for CellOracle (developed with Python 3.8, also tested with 3.10))
- **R** (Runtime for igraph/rnetcarto backend required by CellOracle network analysis)

## Examples

```
score = links.get_network_score(); score.to_csv("hnf4a_foxa1_network_scores.csv")
```

## Evaluation signals

- Hnf4α-Foxa1 network score table is non-empty and reproducible per cluster via Links.get_network_score export
- Early reprogramming clusters show high degree centrality for Hes1, Eno1, Fos, Foxq1, and Zfp57, while Klf2 and Egr1 degree centrality increases in later transition stages
- Cartography classification labels the early cluster network with a connector-hub configuration consistent with Figure 1G–K
- Mef2a and Klf6 appear highly connected in fibroblasts with connections largely decommissioned in successfully converting (iEP) cells
- Pairwise comparison statistics (early vs iEP, iEP vs dead-end) are exported and qualitatively match the centrality trajectory plots

## Limitations

- Findings are specific to the fibroblast-to-iEP direct reprogramming system (MEFs converted to iEPs per Biddy et al. 2018); patterns such as connector-hub configurations may not transfer to other fate-conversion systems
- CellOracle was developed using Python 3.8 and tested with Python 3.10; the R igraph library must be installed and verified for the network analysis backend
- This skill compares network topology only; identifying causal TF regulators requires additional in silico perturbation simulation (e.g. Fos with Yap1) and experimental validation

## Evidence
