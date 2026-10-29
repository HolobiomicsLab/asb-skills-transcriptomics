---
name: single-cell-grn-visualization
description: Use when cluster-specific GRN Links objects must be scored and compared
  across a reprogramming or differentiation trajectory, and centrality/cartography
  structure (e.g., Hnf4α-Foxa1 edge strength, connector-hub detection) needs to be
  visualized.
when_to_use_negative:
- You only have raw scRNA-seq counts and no inferred cluster-specific GRN Links objects
  — first run CellOracle GRN inference rather than network scoring/visualization
- The goal is in silico TF perturbation simulation (KO/overexpression) of gene expression
  rather than structural network characterization
- The network of interest is not a TF regulatory network with a meaningful weighted
  adjacency — igraph/rnetcarto centrality and cartography roles are not interpretable
  on arbitrary graphs
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: CellOracle
  role: 'Provides the network_analysis module: Links.get_network_score for per-cluster
    edge/network scores plus igraph-based centrality and cartography wrapper functions'
  repo: https://github.com/morris-lab/CellOracle
- name: igraph
  role: R graph library backend used by CellOracle for computing degree and eigenvector
    centrality on cluster-specific GRNs; must be installed for the cartography functions
  repo: https://github.com/igraph/rigraph
- name: rnetcarto
  role: R backend for network cartography — computes network modularity and node roles
    (z-score, participation) by simulated annealing for the connector-hub classification
  repo: cran/rnetcarto
- name: Python
  role: Runtime environment for CellOracle (developed with python 3.8, tested with
    python 3.10) for network scoring and export
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:4790636807c10576a5eb4ae9b2457596d454fde83642df796386d9e9c5069415
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# single-cell-grn-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Method for scoring, characterizing, and visualizing cluster-specific gene regulatory networks (GRNs) along a cell-fate trajectory using CellOracle's network_analysis module, computing per-cluster edge/network scores, degree and eigenvector centrality, and network cartography roles to identify connector hubs and regulatory reconfiguration.

## When to use

Use when cluster-specific GRN Links objects must be scored and compared across a reprogramming or differentiation trajectory, and centrality/cartography structure (e.g., Hnf4α-Foxa1 edge strength, connector-hub detection) needs to be visualized.

## When NOT to use

- You only have raw scRNA-seq counts and no inferred cluster-specific GRN Links objects — first run CellOracle GRN inference rather than network scoring/visualization
- The goal is in silico TF perturbation simulation (KO/overexpression) of gene expression rather than structural network characterization
- The network of interest is not a TF regulatory network with a meaningful weighted adjacency — igraph/rnetcarto centrality and cartography roles are not interpretable on arbitrary graphs

## Inputs

- Cluster-specific CellOracle GRN Links objects (one per reprogramming trajectory cluster, e.g., fibroblast, early reprogramming, iEP, dead-end clusters)
- Cluster assignment / trajectory metadata for cells (e.g., from scRNA-seq processed with Cell Ranger 6.0.1)
- Base-GRN (e.g., generated from the Mouse sci-ATAC-seq Atlas) underlying the inferred Links

## Outputs

- Per-cluster network score table (e.g., Hnf4α-Foxa1 edge-strength/network scores via Links.get_network_score)
- Degree and eigenvector centrality values per cluster with plotted centrality trajectories across the reprogramming trajectory
- Network cartography classification per cluster (node roles: module, z-score, participation coefficient) identifying connector hubs
- Degree-centrality comparison statistics (early cluster vs iEP cluster; iEP vs dead-end clusters)

## How to apply

Load the cluster-specific GRN Links objects for each trajectory cluster (e.g., fibroblast, early reprogramming, iEP, dead-end clusters) into the CellOracle network_analysis module. Compute per-cluster edge-strength and network scores with Links.get_network_score and export the score table (e.g., the Hnf4α-Foxa1 edge score). Compute degree centrality and eigenvector centrality per cluster using the igraph-based CellOracle network_analysis functions, plotting centrality trajectories along the reprogramming progression. Run network cartography classification per cluster via the CellOracle cartography functions backed by R igraph and rnetcarto (which computes modularity roles — module, z-score, participation — by simulated annealing) to identify the early connector-hub configuration. Compare degree centrality between selected cluster pairs (early cluster vs iEP, iEP vs dead-end clusters) and export comparison statistics. Rationale: graph-theoretic node identification in the Kamimoto et al. reprogramming study revealed that Mef2a and Klf6 connections are decommissioned in successfully converting cells, and centrality trajectories distinguish early hubs (Hes1, Eno1, Fos, Foxq1, Zfp57) from later-transition hubs (Klf2, Egr1).

## Related tools

- **CellOracle** (Provides the network_analysis module: Links.get_network_score for per-cluster edge/network scores plus igraph-based centrality and cartography wrapper functions) — https://github.com/morris-lab/CellOracle
- **igraph** (R graph library backend used by CellOracle for computing degree and eigenvector centrality on cluster-specific GRNs; must be installed for the cartography functions) — https://github.com/igraph/rigraph
- **rnetcarto** (R backend for network cartography — computes network modularity and node roles (z-score, participation) by simulated annealing for the connector-hub classification) — cran/rnetcarto
- **Python** (Runtime environment for CellOracle (developed with python 3.8, tested with python 3.10) for network scoring and export)

## Examples

```
scores = links.get_network_score(); scores.to_csv("hnf4a_foxa1_network_scores.csv")
```

## Evaluation signals

- Links.get_network_score returns a non-empty score table per cluster and exports without error (e.g., Hnf4α-Foxa1 edge score present in each cluster's table)
- Qualitative centrality trajectory pattern matches Figure 1G–K: Hes1, Eno1, Fos, Foxq1, and Zfp57 show relatively high degree centrality in early reprogramming clusters; Klf2 and Egr1 degree centrality increases in later transition stages
- Cartography classification labels the early reprogramming cluster's configuration as a connector-hub topology, consistent with the reported early connector-hub classification
- Degree-centrality comparison statistics show the expected ordering: early cluster vs iEP cluster and iEP cluster vs dead-end clusters differ as reported
- R dependency check passes: igraph and rnetcarto libraries are installed and reachable from CellOracle before running cartography

## Limitations

- Centrality and cartography findings are qualitative pattern-level comparisons (e.g., against Figure 1G–K), not exact numeric reproductions; stochastic backends like rnetcarto's simulated annealing can vary module assignments between runs
- The R igraph library must be installed separately per the CellOracle instructions; cartography functions fail without it
- Interpretation is tied to the specific fibroblast-to-iEP reprogramming system (Hnf4α-Foxa1 cocktail); trajectories in other lineages may not share the same hub ordering

## Evidence

- [readme] rnetcarto performs the cartography role computation behind CellOracle: [evidence span withheld — anchor status: external_doc]
