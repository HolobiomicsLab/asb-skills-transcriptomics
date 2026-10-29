---
name: single-cell-cluster-specific-network-inference
description: Use when you have a scATAC-seq-derived base GRN edge list plus a scRNA-seq
  expression matrix (e.g. Mouse sci-ATAC-seq Atlas with the Biddy et al. 2018 MEF-to-iEP
  data) and need per-cluster regulatory networks.
when_to_use_negative:
- You only need the scATAC-seq base GRN itself (motif-level potential connections)
  and have no matched scRNA-seq expression to prune against — CellOracle's base GRN
  construction is the relevant skill instead
- The data lack defined cell clusters or trajectory states, so no cluster-specific
  context exists to condition the network on
- Your goal is cell type classification rather than network inference — Capybara (used
  for cell type classification in this paper) is the appropriate tool
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0602
tools:
- name: CellOracle
  role: Loads the base GRN and scRNA-seq data into an Oracle object, prunes inactive
    edges per cluster, and fits cluster-specific edge weights via the regularized-regression
    Links pipeline
  repo: https://github.com/morris-lab/CellOracle
- name: Python
  role: Runtime environment in which CellOracle is executed (CellOracle developed
    with Python 3.8, tested with Python 3.10)
- name: igraph
  role: Graph library backing the Louvain clustering of cells into trajectory-state
    clusters used to define each network context
  repo: https://github.com/igraph/rigraph
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:9971408346e88f5f2f09fd1611e64b2bd7897bee6e982736d2f239a606df4b95
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
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# single-cell-cluster-specific-network-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Converts a base GRN derived from scATAC-seq data into context-dependent, cluster-specific GRN configurations by pruning inactive TF-target edges against single-cell RNA-seq expression and refitting edge weights per cluster with CellOracle.

## When to use

Use when you have a scATAC-seq-derived base GRN edge list plus a scRNA-seq expression matrix (e.g. Mouse sci-ATAC-seq Atlas with the Biddy et al. 2018 MEF-to-iEP data) and need per-cluster regulatory networks.

## When NOT to use

- You only need the scATAC-seq base GRN itself (motif-level potential connections) and have no matched scRNA-seq expression to prune against — CellOracle's base GRN construction is the relevant skill instead
- The data lack defined cell clusters or trajectory states, so no cluster-specific context exists to condition the network on
- Your goal is cell type classification rather than network inference — Capybara (used for cell type classification in this paper) is the appropriate tool

## Inputs

- Base GRN edge list derived from the Mouse sci-ATAC-seq Atlas (TF-target motif connections)
- Single-cell RNA-seq expression matrix (e.g. Biddy et al. 2018 MEF-to-iEP scRNA-seq data)
- Cell clustering assignment (Louvain clusters over the trajectory states)

## Outputs

- One CellOracle Links object per cell cluster containing the pruned, weighted cluster-specific GRN
- Per-cluster edge-weight tables
- Pickled per-cluster Links objects saved to disk (e.g. cluster_grn_links.pkl)

## How to apply

Load the base GRN edge list and the scRNA-seq expression matrix into a CellOracle Oracle object, then cluster the cells with Louvain clustering via the scanpy/igraph graph interface so each trajectory state (e.g. MEF-to-iEP reprogramming states) forms a cell cluster. For each cluster, filter the base GRN to retain only TF-target edges whose genes are detectably expressed in that cluster, removing connections inactive in the cluster's transcriptional context. Fit cluster-specific edge weights for the retained edges with CellOracle's regularized regression network-inference step (the Links pipeline) applied to the cluster's expression matrix. Assemble one CellOracle Links object per cluster containing the pruned, weighted GRN and export the corresponding edge-weight tables, saving everything (e.g. cluster_grn_links.pkl) to disk for downstream perturbation simulation.

## Related tools

- **CellOracle** (Loads the base GRN and scRNA-seq data into an Oracle object, prunes inactive edges per cluster, and fits cluster-specific edge weights via the regularized-regression Links pipeline) — https://github.com/morris-lab/CellOracle
- **Python** (Runtime environment in which CellOracle is executed (CellOracle developed with Python 3.8, tested with Python 3.10))
- **igraph** (Graph library backing the Louvain clustering of cells into trajectory-state clusters used to define each network context) — https://github.com/igraph/rigraph

## Examples

```
from celloracle.applications import Oracle; oracle = Oracle(); oracle.import_sc_data(dataframe=expr_matrix); oracle.import_TFdata(TF_info_matrix=base_grn); oracle.perform_PCA(); oracle.cluster_cells(cluster_field_name='cluster')
```

## Evaluation signals

- Exactly one Links object and one edge-weight table exists for every cell cluster identified in the Oracle object
- Each saved Links object is a loadable pickled CellOracle Links object with a non-empty pruned edge list
- No edge in a cluster-specific GRN targets a gene absent from that cluster's expression matrix (all retained edges map to detectably expressed genes)
- Edge weights are finite fitted values from the regularized regression step, not placeholder or base-GRN motif scores

## Limitations

- The pruned network is only as good as the base GRN; connections not present in the scATAC-seq-derived base GRN can never be recovered, and pruning can drop edges that are real but lowly expressed
- Results depend on the Louvain clustering resolution chosen via the scanpy/igraph interface; different clusterings yield different network configurations
- Cluster-specific inference requires adequate cell numbers and gene detection per cluster; sparse clusters may yield unreliable edge weights

## Evidence

- [methods] Base GRN pruning into cluster-specific networks: [evidence span withheld — anchor status: not_found]
