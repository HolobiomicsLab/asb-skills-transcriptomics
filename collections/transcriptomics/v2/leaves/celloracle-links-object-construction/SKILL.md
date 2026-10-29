---
name: celloracle-links-object-construction
description: Use when you have a base GRN edge list (e.g. from the Mouse sci-ATAC-seq
  Atlas) plus a single-cell expression matrix (e.g. Biddy et al. 2018 MEF-to-iEP scRNA-seq)
  and need per-cluster context-dependent GRN configurations.
when_to_use_negative:
- You need de novo TF-target inference without an epigenome-derived base GRN — CellOracle's
  pruning assumes a scATAC-seq-derived base GRN as the candidate edge universe
- You want in silico TF perturbation (KO/overexpression) simulations rather than cluster-specific
  GRN construction — that is a downstream CellOracle step performed on the Links objects
- Your data are bulk transcriptomics rather than single-cell; cluster-level pruning
  and regularized regression on single-cell expression matrices do not apply
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: CellOracle
  role: 'Core pipeline: Oracle object construction, base-GRN filtering per cluster,
    and regularized regression Links inference'
  repo: https://github.com/morris-lab/CellOracle
- name: Python
  role: Execution environment; CellOracle was developed using python 3.8 and tested
    with python 3.10
- name: igraph
  role: Graph interface used with scanpy for Louvain clustering of cells into trajectory-state
    clusters before pruning
  repo: https://github.com/igraph/rigraph
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:ec4baaaca7faf056cfd6507cdf5e5165d3ad24b51316570d652798cdea989f33
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# celloracle-links-object-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs cluster-specific CellOracle Links objects by pruning a scATAC-seq-derived base GRN against scRNA-seq expression per cell cluster (Louvain via scanpy/igraph) and fitting regularized-regression edge weights, yielding one pruned, weighted GRN per cluster.

## When to use

Use when you have a base GRN edge list (e.g. from the Mouse sci-ATAC-seq Atlas) plus a single-cell expression matrix (e.g. Biddy et al. 2018 MEF-to-iEP scRNA-seq) and need per-cluster context-dependent GRN configurations.

## When NOT to use

- You need de novo TF-target inference without an epigenome-derived base GRN — CellOracle's pruning assumes a scATAC-seq-derived base GRN as the candidate edge universe
- You want in silico TF perturbation (KO/overexpression) simulations rather than cluster-specific GRN construction — that is a downstream CellOracle step performed on the Links objects
- Your data are bulk transcriptomics rather than single-cell; cluster-level pruning and regularized regression on single-cell expression matrices do not apply

## Inputs

- Base GRN edge list (TF-target pairs) derived from scATAC-seq data, e.g. the Mouse sci-ATAC-seq Atlas
- Single-cell RNA-seq expression matrix, e.g. the Biddy et al. (2018) MEF-to-iEP scRNA-seq matrix
- Cell cluster assignments from Louvain clustering (scanpy/igraph) of the scRNA-seq data

## Outputs

- One pickled CellOracle Links object per cell cluster (e.g. cluster_grn_links.pkl) containing the pruned, weighted cluster-specific GRN
- Per-cluster edge-weight tables for the retained TF-target edges

## How to apply

Load the base GRN edge list and the expression matrix (e.g. the Biddy et al. (2018) MEF-to-iEP matrix) into a CellOracle Oracle object, then cluster cells with Louvain clustering using the scanpy/igraph graph interface so each trajectory state (e.g. MEF-to-iEP states) forms a cluster. For each cluster, filter the base GRN to retain only TF-target edges whose genes are detectably expressed in that cluster, removing inactive connections that do not fit the cluster's transcriptional context. Fit cluster-specific edge weights for the retained edges using CellOracle's regularized regression network-inference step (the Links pipeline) on the cluster's expression matrix. Assemble one CellOracle Links object per cluster containing the pruned, weighted GRN, export the edge-weight tables, and save all per-cluster Links objects and tables to disk (e.g. cluster_grn_links.pkl). The rationale is that scATAC-seq motifs define potential regulatory edges, while scRNA-seq activity selects the active edges that represent regulation in a specific cell type or state.

## Related tools

- **CellOracle** (Core pipeline: Oracle object construction, base-GRN filtering per cluster, and regularized regression Links inference) — https://github.com/morris-lab/CellOracle
- **Python** (Execution environment; CellOracle was developed using python 3.8 and tested with python 3.10)
- **igraph** (Graph interface used with scanpy for Louvain clustering of cells into trajectory-state clusters before pruning) — https://github.com/igraph/rigraph

## Examples

```
import celloracle as co; oracle = co.load_hdf5('oracle.object'); links = co.links_from_selected_GRNs(oracle=oracle, cluster_name='cluster_0'); links.to_hdf5('cluster_grn_links.pkl')
```

## Evaluation signals

- One Links object and one edge-weight table exist for every cell cluster in the dataset
- Each saved Links object is a loadable pickled CellOracle Links object (e.g. from cluster_grn_links.pkl) with a non-empty pruned edge list
- No edge in any cluster's GRN targets genes absent from that cluster's expression matrix
- Retained edges are a subset of the scATAC-seq base GRN edge list, and inactive edges (non-expressed genes) have been removed per cluster
- Edge weights are finite values fitted by the regularized regression step on the cluster's expression matrix

## Limitations

- Pruning removes only connections inactive in the cluster's transcriptional context; edges retained are candidate interactions whose activity depends on the quality of the base GRN and expression detectability thresholds
- CellOracle code requires specific runtime versions (developed with python 3.8, tested with python 3.10) and R dependencies (e.g. igraph), which can complicate installation
- The context provides no explicit changelog for the repository, so version-specific behavior changes in the Links pipeline are not documented here

## Evidence

- [methods] Base-GRN: Base-GRN generated from `the Mouse sci-ATAC-seq Atlas: "Base-GRN: Base-GRN generated from `the Mouse sci-ATAC-seq Atlas"
- [abstract] CellOracle, a computational method to infer GRNs from single-cell transcriptome and epigenome data: "CellOracle, a computational method to infer GRNs from single-cell transcriptome and epigenome data"
- [intro] these results demonstrate the efficacy of CellOracle to infer and interpret cell-type-specific GRN configurations at high resolution: "these results demonstrate the efficacy of CellOracle to infer and interpret cell-type-specific GRN configurations at high resolution"
- [methods] CellOracle code is open source and available on GitHub: (https://github.com/morris-lab/CellOracle): "CellOracle code is open source and available on GitHub: (https://github.com/morris-lab/CellOracle)"
