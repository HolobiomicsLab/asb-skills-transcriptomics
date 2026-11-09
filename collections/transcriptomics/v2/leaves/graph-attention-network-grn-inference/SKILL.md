---
name: graph-attention-network-grn-inference
description: Use when you have scRNA-seq expression data for one developmental lineage
  plus a prior gene interaction network (edgelist) and need a lineage-specific GRN,
  e.g. for downstream driver regulator identification.
when_to_use_negative:
- The dataset contains multiple fate trajectories with bifurcations that must be analyzed
  simultaneously — CEFCON can only handle one cell lineage at a time
- You need to model extrinsic factors such as cell-cell communication, spatial location,
  or epigenetic regulation — CEFCON only considers intracellular RNA-seq factors
- No prior gene interaction network is available for the organism or lineage of interest
edam_operation: http://edamontology.org/operation_3766
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CEFCON
  role: Graph attention neural network under contrastive learning framework used to
    infer the cell-lineage-specific GRN (NetModel, get_cefcon_results)
  repo: https://github.com/WPZgithub/CEFCON
- name: PyTorch
  role: Deep learning backend (>=1.13.0,<2.0) for training the graph attention network
- name: torch-geometric
  role: Graph neural network operations (>=2.1.0) underlying the attention model
- name: scanpy
  role: Loading and preprocessing the scRNA-seq AnnData input (>=1.8.2)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:cde56637acc9f0a22349251669e87ee184235a7aac8d13598cb29f920c492f43
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
  edam_operation: http://edamontology.org/operation_3766
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# graph-attention-network-grn-inference

## Summary

Train CEFCON's graph attention neural network under a contrastive learning framework to infer a cell-lineage-specific gene regulatory network (GRN) from single-cell RNA-seq data and a prior gene interaction network, as done for mouse hematopoietic stem cell (mHSC) differentiation lineages.

## When to use

Use when you have scRNA-seq expression data for one developmental lineage plus a prior gene interaction network (edgelist) and need a lineage-specific GRN, e.g. for downstream driver regulator identification.

## When NOT to use

- The dataset contains multiple fate trajectories with bifurcations that must be analyzed simultaneously — CEFCON can only handle one cell lineage at a time
- You need to model extrinsic factors such as cell-cell communication, spatial location, or epigenetic regulation — CEFCON only considers intracellular RNA-seq factors
- No prior gene interaction network is available for the organism or lineage of interest

## Inputs

- Single-cell RNA-seq expression matrix (AnnData/adata) for one cell lineage with pseudotime ordering
- Prior gene interaction network in edgelist format (e.g., mouse prior network)
- Gene differential expression log fold changes
- Lineage-specific data object produced by cefcon.data_preparation

## Outputs

- Cell-lineage-specific inferred GRN (weighted edge list, thresholded with edge_threshold_avgDegree=8)
- Trained CEFCON NetModel with attention weights over prior-network edges

## How to apply

Prepare lineage-specific inputs with cefcon.data_preparation(adata, prior_network), supplying the scRNA-seq matrix, mouse prior gene interaction network in edgelist format, and gene differential expression log fold changes from Zenodo record 7564872. Train the CEFCON graph attention network under the contrastive learning framework with NetModel(epochs=350, repeats=3, cuda='0') on a GPU, running each lineage (erythroid, granulocyte-monocyte, lymphoid) separately. Extract the inferred GRN with get_cefcon_results(edge_threshold_avgDegree=8), which thresholds edges by average degree. The attention mechanism weights prior-network edges using expression dynamics along pseudotime, producing a reliable cell-lineage-specific GRN suitable for control-theoretic driver regulator analysis.

## Related tools

- **CEFCON** (Graph attention neural network under contrastive learning framework used to infer the cell-lineage-specific GRN (NetModel, get_cefcon_results)) — https://github.com/WPZgithub/CEFCON
- **PyTorch** (Deep learning backend (>=1.13.0,<2.0) for training the graph attention network)
- **torch-geometric** (Graph neural network operations (>=2.1.0) underlying the attention model)
- **scanpy** (Loading and preprocessing the scRNA-seq AnnData input (>=1.8.2))

## Examples

```
cefcon_GRN_model = cf.NetModel(epochs=350, repeats=3, cuda='0'); cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)
```

## Evaluation signals

- The GRN is extracted only after get_cefcon_results(edge_threshold_avgDegree=8) returns a valid edge set over genes present in both adata and the prior network
- Training converges with NetModel(epochs=350, repeats=3); repeat runs should yield stable high-weight attention edges
- The inferred GRN supports recovery of lineage-specific regulators (e.g., Klf1 for erythroid, Mpo for granulocyte-monocyte, Mef2c for lymphoid) in downstream driver_regulators() analysis
- Per-lineage top-20 driver regulator lists reproduce the published sets, including the six common regulators Meis1, Gata2, Jun, Fos, Mycn, Dusp1 matching Fig. 6c-i
- Environment matches requirements: Python 3.9–3.10, pytorch >=1.13.0,<2.0, torch-geometric >=2.1.0, CUDA GPU recommended

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously
- Only intracellular factors are modeled; cell-cell communication and spatial location are not included
- Epigenetic information controlling cell fate decisions is not incorporated; the framework is limited to RNA-seq, with extension to spatial transcriptome and chromatin accessibility left as future work
- Requires a prior gene interaction network and, ideally, an NVIDIA CUDA GPU; pytorch must be pinned to >=1.13.0,<2.0

## Evidence

- [abstract] GRN via graph attention + contrastive learning: "first uses a graph neural network with attention mechanism to infer a cell-lineage-specific gene regulatory network (GRN) from single-cell RNA-sequencing data"
- [methods] data preparation step: "# Data preparation
data = cf.data_preparation(adata, prior_network)"
- [methods] model construction parameters: "# Construct cell-lineage-specific GRN
cefcon_GRN_model = cf.NetModel(epochs=350, repeats=3, cuda='0')"
- [methods] GRN extraction threshold: "cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)"
- [methods] input data source: "All the input data mentioned in the paper can be downloaded from [here](https://zenodo.org/record/7564872)"
- [discussion] lineage-at-a-time limitation: "First, CEFCON can only handle one cell lineage at a time."
- [readme] recommended hardware: "We recommend running CEFCON on CUDA if possible."
- [results] validation of driver regulators: "When applied to the mouse hematopoietic stem cell differentiation data, CEFCON successfully identified driver regulators for three developmental lineages"
