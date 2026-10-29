---
name: gene-regulatory-network-edge-pruning
description: Use when a base GRN built from single-cell epigenomic data (e.g., Mouse
  sci-ATAC-seq Atlas peaks) must be converted into cluster-specific, weighted GRNs
  using a matched scRNA-seq expression matrix with defined cell clusters.
when_to_use_negative:
- No base GRN from scATAC-seq (or equivalent epigenomic source) is available — pruning
  needs candidate TF-target edges to filter
- Cells are unclustered or no cell-type/trajectory-state structure exists to condition
  edge activity on
- The goal is in silico TF perturbation simulation downstream — this skill builds
  the cluster-specific Links objects that simulation consumes, it does not itself
  simulate KO/overexpression
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: CellOracle
  role: Prunes the base GRN against cluster expression and fits cluster-specific edge
    weights via its regularized regression Links pipeline, producing pickled Links
    objects
  repo: https://github.com/morris-lab/CellOracle
- name: igraph
  role: Graph interface used for Louvain clustering of cells so each trajectory state
    forms a cluster prior to edge pruning
  repo: https://github.com/igraph/rigraph
- name: Python
  role: Implementation language for the CellOracle pipeline (developed with Python
    3.8, tested with 3.10)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:895d1fe792bddc5acb9c68e2dd11df113d4a885b783791f59cf9522deef3cf55
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
  - igraph
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# gene-regulatory-network-edge-pruning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prunes a scATAC-seq-derived base GRN using cluster-specific scRNA-seq expression data to yield context-dependent GRN configurations, as done with CellOracle during MEF-to-iEP reprogramming analysis.

## When to use

Use when a base GRN built from single-cell epigenomic data (e.g., Mouse sci-ATAC-seq Atlas peaks) must be converted into cluster-specific, weighted GRNs using a matched scRNA-seq expression matrix with defined cell clusters.

## When NOT to use

- No base GRN from scATAC-seq (or equivalent epigenomic source) is available — pruning needs candidate TF-target edges to filter
- Cells are unclustered or no cell-type/trajectory-state structure exists to condition edge activity on
- The goal is in silico TF perturbation simulation downstream — this skill builds the cluster-specific Links objects that simulation consumes, it does not itself simulate KO/overexpression

## Inputs

- Base GRN edge list derived from scATAC-seq data (Mouse sci-ATAC-seq Atlas)
- Single-cell RNA-seq expression matrix (e.g., Biddy et al. 2018 MEF-to-iEP scRNA-seq data)
- Cell cluster assignments from Louvain clustering of the single-cell data

## Outputs

- One pickled CellOracle Links object per cell cluster containing the pruned, weighted cluster-specific GRN
- Per-cluster edge-weight tables
- Combined save file (e.g., cluster_grn_links.pkl)

## How to apply

Load the base GRN edge list (derived from the Mouse sci-ATAC-seq Atlas) and the scRNA-seq expression matrix (e.g., the Biddy et al. 2018 MEF-to-iEP data) into a CellOracle Oracle object. Cluster cells with Louvain clustering via the scanpy/igraph interface so each trajectory state (e.g., MEF-to-iEP states) forms a cluster. For each cluster, filter the base GRN to retain only TF-target edges whose genes are detectably expressed in that cluster, removing inactive connections; then fit cluster-specific edge weights with CellOracle's regularized regression network-inference step (Links pipeline) on the cluster's expression matrix. Assemble one CellOracle Links object per cluster and export edge-weight tables (e.g., cluster_grn_links.pkl).

## Related tools

- **CellOracle** (Prunes the base GRN against cluster expression and fits cluster-specific edge weights via its regularized regression Links pipeline, producing pickled Links objects) — https://github.com/morris-lab/CellOracle
- **igraph** (Graph interface used for Louvain clustering of cells so each trajectory state forms a cluster prior to edge pruning) — https://github.com/igraph/rigraph
- **Python** (Implementation language for the CellOracle pipeline (developed with Python 3.8, tested with 3.10))

## Examples

```
oracle.get_links(model_method='BayesianRidge'); links.per_cluster_to_pickle('cluster_grn_links.pkl')
```

## Evaluation signals

- Exactly one Links object and one edge-weight table exist per cell cluster
- Every Links object loads as a valid pickled CellOracle Links object with a non-empty pruned edge list
- No retained edge targets a gene absent from the corresponding cluster's expression matrix
- Each edge in a cluster's Links object is a subset of the original base GRN edge list (pruning removes edges, never adds them)

## Limitations

- Edge activity is judged by detectable expression in the cluster — genes absent or dropouts in scRNA-seq can cause true regulatory edges to be pruned
- Cluster-specific GRNs depend on the clustering resolution (Louvain); different clusterings yield different pruned configurations
- CellOracle's efficacy was demonstrated on specific systems (MEF-to-iEP reprogramming, hematopoiesis from Paul et al. 2015); behavior on other tissues/trajectories requires its own validation

## Evidence
