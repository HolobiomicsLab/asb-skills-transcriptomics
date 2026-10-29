---
name: base-grn-integration-with-scrna-seq
description: Use when you have a base GRN edge list derived from scATAC-seq (e.g.,
  the Mouse sci-ATAC-seq Atlas) plus an scRNA-seq expression matrix (e.g., Biddy et
  al. 2018 MEF-to-iEP) and need cluster-specific pruned GRNs.
when_to_use_negative:
- Input is only an scRNA-seq dataset without a prebuilt base GRN — pruning requires
  epigenome-derived candidate edges; without them use de novo GRN inference instead
- The goal is perturbation simulation or cell-state transition prediction rather than
  producing context-dependent GRN configurations
- Expression matrix contains genes absent from the base GRN edge list and you need
  to extend the base GRN rather than prune it
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: CellOracle
  role: 'Core pipeline: loads the base GRN and scRNA-seq expression into an Oracle
    object, prunes inactive edges per cluster, and fits regularized-regression edge
    weights via the Links pipeline'
  repo: https://github.com/morris-lab/CellOracle
- name: igraph
  role: Graph library underlying the Louvain clustering used to define cell clusters
    for per-cluster GRN pruning
  repo: https://github.com/igraph/rigraph
- name: Python
  role: Runtime environment for the CellOracle Links pipeline and pickling of per-cluster
    Links objects
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:c0745c63a2a7054ddd113e606e7e9004b4379b2193f359fc1d90e8eb193c9d6b
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
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# base-grn-integration-with-scrna-seq

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates an epigenome-derived base GRN with single-cell RNA-seq expression data using CellOracle to prune inactive TF-target edges and fit cluster-specific, weighted GRN configurations. This converts a static scATAC-seq-based edge list into context-dependent regulatory networks per cell state.

## When to use

Use when you have a base GRN edge list derived from scATAC-seq (e.g., the Mouse sci-ATAC-seq Atlas) plus an scRNA-seq expression matrix (e.g., Biddy et al. 2018 MEF-to-iEP) and need cluster-specific pruned GRNs.

## When NOT to use

- Input is only an scRNA-seq dataset without a prebuilt base GRN — pruning requires epigenome-derived candidate edges; without them use de novo GRN inference instead
- The goal is perturbation simulation or cell-state transition prediction rather than producing context-dependent GRN configurations
- Expression matrix contains genes absent from the base GRN edge list and you need to extend the base GRN rather than prune it

## Inputs

- Base GRN edge list derived from the Mouse sci-ATAC-seq Atlas (TF-target pairs from scATAC-seq)
- scRNA-seq expression matrix (e.g., the Biddy et al. (2018) MEF-to-iEP expression data)
- Cell cluster assignments per cell (e.g., Louvain clusters from scanpy/igraph)

## Outputs

- One pickled CellOracle Links object per cluster containing the pruned, weighted cluster-specific GRN (cluster_grn_links.pkl)
- Per-cluster edge-weight tables of retained TF-target regulatory connections
- A mapping of cluster identity to its active, filtered GRN configuration

## How to apply

Load the base GRN edge list and the scRNA-seq expression matrix (e.g., the Biddy et al. (2018) MEF-to-iEP matrix) into a CellOracle Oracle object, then define cell clusters with Louvain clustering via the scanpy/igraph interface so each MEF-to-iEP trajectory state forms a cluster. For each cluster, prune the base GRN by retaining only TF-target edges whose genes are detectably expressed in that cluster's transcriptional context, removing inactive connections. Fit edge weights for the surviving edges using CellOracle's regularized regression network-inference step (the Links pipeline) applied to each cluster's expression matrix. Assemble one CellOracle Links object per cluster containing the pruned, weighted cluster-specific GRN, export the edge-weight tables, and persist everything (e.g., as cluster_grn_links.pkl). The rationale is that CellOracle integrates different single-cell data modalities — chromatin accessibility defines candidate edges, transcriptomics filters and weights them per cell identity.

## Related tools

- **CellOracle** (Core pipeline: loads the base GRN and scRNA-seq expression into an Oracle object, prunes inactive edges per cluster, and fits regularized-regression edge weights via the Links pipeline) — https://github.com/morris-lab/CellOracle
- **igraph** (Graph library underlying the Louvain clustering used to define cell clusters for per-cluster GRN pruning) — https://github.com/igraph/rigraph
- **Python** (Runtime environment for the CellOracle Links pipeline and pickling of per-cluster Links objects)

## Examples

```
import celloracle as co; links = co.links.load_links_object(path="cluster_grn_links.pkl")
```

## Evaluation signals

- One loadable pickled Links object and one edge-weight table exist for every defined cell cluster
- Each Links object has a non-empty pruned edge list (no cluster yields a completely empty GRN)
- No retained edge targets a gene absent from the corresponding cluster's expression matrix
- Every retained TF-target edge in the cluster-specific GRN traces back to an edge in the original scATAC-seq-derived base GRN
- Edge weights differ across clusters for the same base edge, confirming context-dependent reconfiguration rather than a single shared network

## Limitations

- Pruning depends on cluster definitions: cells misassigned by Louvain clustering will inherit an inappropriate active-edge set
- The base GRN caps candidate edges — regulation not supported by the Mouse sci-ATAC-seq Atlas cannot be recovered by this skill
- Gene detectability thresholds for 'detectably expressed' determine which edges survive; very low-expressed regulators may be wrongly pruned

## Evidence

- [abstract] Base GRN built from scATAC-seq data is the substrate for pruning: "We previously developed CellOracle, a computational method to infer GRNs from single-cell transcriptome and epigenome data."
- [intro] CellOracle is explicitly a multi-modal integration pipeline: "We previously described CellOracle, a computational pipeline for GRN inference via integrating different single-cell data modalities (Kamimoto et al., 2020)."
- [intro] The output of this skill is high-resolution cell-type-specific GRN configurations: "Together, these results demonstrate the efficacy of CellOracle to infer and interpret cell-type-specific GRN configurations at high resolution"
- [methods] The base GRN input dataset is the Mouse sci-ATAC-seq Atlas: "Base-GRN: Base-GRN generated from `the Mouse sci-ATAC-seq Atlas"
- [methods] Python version support for running the CellOracle pipeline: "CellOracle was developed using python 3.8. It is also tested with python 3.10."
