---
name: gene-regulatory-network-centrality-analysis
description: Use when cluster-specific GRN Links objects are available and you need
  to rank TFs by network position (degree/eigenvector centrality) or classify network
  configuration (cartography) across a reprogramming trajectory.
when_to_use_negative:
- The analysis goal is perturbation simulation of gene expression (in silico KO/overexpression)
  rather than structural network scoring — use CellOracle's perturbation simulation
  module instead.
- Input is raw scRNA-seq or clone barcode data that has not yet been converted into
  CellOracle Links/GRN objects.
- Only a single static network with no cluster or time-resolved comparison is needed,
  so trajectory-level centrality dynamics cannot be assessed.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3170
tools:
- name: CellOracle
  role: Provides the Links GRN objects, network_analysis module (get_network_score,
    igraph-based centrality and cartography functions)
  repo: https://github.com/morris-lab/CellOracle
- name: igraph
  role: Graph library backend used by CellOracle network_analysis for degree and eigenvector
    centrality computation
  repo: https://github.com/igraph/rigraph
- name: rnetcarto
  role: R backend for network cartography (modularity and role computation via simulated
    annealing) of each cluster's GRN
  repo: https://cran.r-project.org/package=rnetcarto
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:e41a6d5a7c12d377862c825509425da6af991ad102426848100869af46997621
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
  - R
  - igraph
  - newCloneCalling
  - rnetcarto
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# gene-regulatory-network-centrality-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute graph-theoretic scores (degree centrality, eigenvector centrality, network cartography roles) on CellOracle-inferred cluster-specific gene regulatory networks to identify which transcription factors occupy critical regulatory nodes during fibroblast-to-iEP reprogramming.

## When to use

Use when cluster-specific GRN Links objects are available and you need to rank TFs by network position (degree/eigenvector centrality) or classify network configuration (cartography) across a reprogramming trajectory.

## When NOT to use

- The analysis goal is perturbation simulation of gene expression (in silico KO/overexpression) rather than structural network scoring — use CellOracle's perturbation simulation module instead.
- Input is raw scRNA-seq or clone barcode data that has not yet been converted into CellOracle Links/GRN objects.
- Only a single static network with no cluster or time-resolved comparison is needed, so trajectory-level centrality dynamics cannot be assessed.

## Inputs

- CellOracle Links objects (cluster-specific GRN edge lists) for reprogramming trajectory clusters
- Sample-specific CellOracle Links/GRN objects for day-4 iEP-enriched, iEP-depleted, and native fibroblast state groups
- Cell metadata assigning cells to reprogramming clusters or fate groups

## Outputs

- Per-cluster TF degree and eigenvector centrality tables and centrality trajectory plots
- Hnf4α-Foxa1 network score table from Links.get_network_score
- Network cartography classification per cluster (module, z-score, participation coefficient)
- Degree-centrality comparison statistics and plots (e.g., Figure 1G–K, Figure 2H)

## How to apply

Load the cluster-specific CellOracle Links objects for the reprogramming trajectory clusters into the CellOracle network_analysis module. Compute per-cluster network scores (e.g., the Hnf4α-Foxa1 edge-strength score via Links.get_network_score) and export the score table. Use the igraph-based CellOracle network_analysis functions to compute degree centrality and eigenvector centrality per cluster and plot centrality trajectories along the reprogramming time course. Run the network cartography classification (igraph/rnetcarto backend, netcarto simulated-annealing module/role assignment) per cluster to detect configurations such as the early connector-hub state. Finally, compare degree centrality between key state pairs (e.g., early cluster vs iEP cluster, iEP cluster vs dead-end clusters) and export comparison statistics; for clone-resolved fate questions, build sample-specific Links objects for each day-4 state group (iEP-enriched 9 clones / iEP-depleted 43 clones vs native fibroblasts) before scoring.

## Related tools

- **CellOracle** (Provides the Links GRN objects, network_analysis module (get_network_score, igraph-based centrality and cartography functions)) — https://github.com/morris-lab/CellOracle
- **igraph** (Graph library backend used by CellOracle network_analysis for degree and eigenvector centrality computation) — https://github.com/igraph/rigraph
- **rnetcarto** (R backend for network cartography (modularity and role computation via simulated annealing) of each cluster's GRN) — https://cran.r-project.org/package=rnetcarto

## Examples

```
links.get_network_score(); score_table = links.entropy_score_df; net = oa.analyze_links(links); centrality = net.compute_degree_centrality()
```

## Evaluation signals

- Hes1, Eno1, Fos, Foxq1, and Zfp57 show relatively high degree centrality in the early reprogramming clusters, while Klf2 and Egr1 degree centrality increases in later transition stages.
- Cartography classification of the early reprogramming cluster returns a connector-hub configuration.
- Mef2a and Klf6 are highly connected in fibroblasts and these connections are largely decommissioned in successfully converting cells.
- Clone-resolved comparison (Figure 2H) shows Klf6, Tbx5, Tfap2b, and Foxs1 more highly connected in failing cells versus Fos, Klf5, and Junb more highly connected in iEP-destined cells.
- Exported score tables are complete for all clusters and centrality trajectory plots reproduce the qualitative patterns of Figure 1G–K.

## Limitations

- CellOracle requires Python 3.8 (also tested with Python 3.10) and the R igraph library must be installed separately for centrality/cartography backends.
- The rnetcarto cartography result depends on simulated-annealing modularity optimization, so module assignments may vary between runs.
- GRN inference is limited to regulators represented in the base-GRN (e.g., the Mouse sci-ATAC-seq Atlas), so TFs absent from that cis-regulatory map cannot be scored.

## Evidence

- [other] Cartography backend grounding: [evidence span withheld — anchor status: external_doc]
