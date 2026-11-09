---
name: network-cartography-classification
description: Use when cluster-specific GRN Links objects exist for a cell trajectory
  (e.g. fibroblast → reprogramming → iEP clusters) and you need degree/eigenvector
  centrality trajectories and cartography role classification to compare network configurations
  across clusters.
when_to_use_negative:
- Input is raw scRNA-seq count data rather than inferred CellOracle Links/GRN objects
  — GRN inference (base-GRN from scATAC plus transcriptome) must precede network analysis
- The goal is perturbation simulation of TF knockouts/overexpression rather than structural
  network comparison — use CellOracle's in silico perturbation module instead
- Networks are too sparse or disconnected for meaningful modularity detection, making
  rnetcarto cartography roles unstable
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: CellOracle
  role: Provides the Links objects, network_analysis module (get_network_score, centrality
    functions, cartography functions) that wrap igraph/rnetcarto
  repo: https://github.com/morris-lab/CellOracle
- name: igraph
  role: Graph library used by CellOracle's network_analysis to compute degree and
    eigenvector centrality per cluster
  repo: https://github.com/igraph/rigraph
- name: rnetcarto
  role: R backend computing network modularity and cartography roles (z-score, participation
    coefficient) via simulated annealing
  repo: https://github.com/cran/rnetcarto
- name: Python
  role: Runtime for CellOracle (developed/tested with Python 3.8 and 3.10) in which
    the network_analysis functions are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:af774d49baaf626429b9f80c2401c3119901155ccd909ed49ff044e8df4d4927
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
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# network-cartography-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classify each cluster's CellOracle-inferred gene regulatory network using igraph-based centrality metrics and rnetcarto network cartography roles, to identify hub/connector-hub configurations that distinguish fibroblast, early reprogramming, and iEP clusters during direct lineage reprogramming (Hnf4α-Foxa1 to iEP conversion).

## When to use

Use when cluster-specific GRN Links objects exist for a cell trajectory (e.g. fibroblast → reprogramming → iEP clusters) and you need degree/eigenvector centrality trajectories and cartography role classification to compare network configurations across clusters.

## When NOT to use

- Input is raw scRNA-seq count data rather than inferred CellOracle Links/GRN objects — GRN inference (base-GRN from scATAC plus transcriptome) must precede network analysis
- The goal is perturbation simulation of TF knockouts/overexpression rather than structural network comparison — use CellOracle's in silico perturbation module instead
- Networks are too sparse or disconnected for meaningful modularity detection, making rnetcarto cartography roles unstable

## Inputs

- Cluster-specific CellOracle Links objects (inferred GRNs) for each reprogramming trajectory cluster
- Cell/cluster metadata ordering clusters along the fibroblast-to-iEP reprogramming trajectory
- Python environment with celloracle installed and R with igraph and rnetcarto for the cartography backend

## Outputs

- Per-cluster network score table (e.g. Hnf4α-Foxa1 edge strength) exported from Links.get_network_score
- Per-cluster degree centrality and eigenvector centrality values and centrality trajectory plots
- Cartography role assignments (module, z-score, participation coefficient) per cluster network
- Early-cluster vs iEP-cluster and iEP vs dead-end cluster degree centrality comparison statistics

## How to apply

Load the cluster-specific GRN Links objects for each reprogramming trajectory cluster into CellOracle's network_analysis module. Compute per-cluster edge/network scores with Links.get_network_score (e.g. the Hnf4α-Foxa1 edge-strength score table) and export it. Compute degree centrality and eigenvector centrality per cluster using the igraph-backed CellOracle network_analysis functions, plotting centrality as trajectories over pseudotime/cluster order — in this study, Hes1, Eno1, Fos, Foxq1, and Zfp57 score high degree centrality in early reprogramming clusters while Klf2 and Egr1 rise in later transition stages. Then run the CellOracle cartography functions (igraph/rnetcarto backend, simulated-annealing modularity/role assignment) to assign each gene a cartography role per cluster and identify the early connector-hub configuration. Finally, compare degree centrality between early-cluster vs iEP-cluster and iEP-cluster vs dead-end clusters, exporting the comparison statistics; validation is reproducing the qualitative patterns of Figure 1G–K (edge strength, centrality trajectories, early connector-hub classification).

## Related tools

- **CellOracle** (Provides the Links objects, network_analysis module (get_network_score, centrality functions, cartography functions) that wrap igraph/rnetcarto) — https://github.com/morris-lab/CellOracle
- **igraph** (Graph library used by CellOracle's network_analysis to compute degree and eigenvector centrality per cluster) — https://github.com/igraph/rigraph
- **rnetcarto** (R backend computing network modularity and cartography roles (z-score, participation coefficient) via simulated annealing) — https://github.com/cran/rnetcarto
- **Python** (Runtime for CellOracle (developed/tested with Python 3.8 and 3.10) in which the network_analysis functions are executed)

## Examples

```
from celloracle.applications import Links; sc.plot_degree_centrality_and_cartography(links_per_cluster)  # per-cluster network_analysis: get_network_score, centrality, cartography via igraph/rnetcarto
```

## Evaluation signals

- Score table from Links.get_network_score reproduces the reported Hnf4α-Foxa1 edge-strength pattern across fibroblast, early reprogramming, and iEP clusters
- Centrality trajectories show Hes1, Eno1, Fos, Foxq1, and Zfp57 with relatively high degree centrality in early reprogramming clusters, and rising Klf2/Egr1 degree centrality in later transition stages
- Cartography classification of the early cluster identifies a connector-hub configuration as in Figure 1G–K
- Degree centrality comparisons (early vs iEP, iEP vs dead-end clusters) produce exported statistics consistent with the paper's reported network decommissioning (e.g. Mef2a/Klf6 highly connected in fibroblasts, connections decommissioned in converting cells)
- Each cluster's network yields valid cartography output (module, z-score, participation) without backend errors from igraph/rnetcarto

## Limitations

- rnetcarto uses simulated annealing, so cartography role assignments (module membership, z-score, participation) can vary between runs
- The CellOracle Python package requires matching R libraries (igraph) to be installed and checked for the network/cartography backends to function
- Conversion outcome is established shortly after TF overexpression, so cluster ordering along the trajectory directly affects which network state is labeled 'early'

## Evidence

- [readme] Fast network modularity and roles computation by simulated annealing (rgraph C library wrapper for R).: [evidence span withheld — anchor status: external_doc]
