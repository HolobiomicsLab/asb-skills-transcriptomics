---
name: graph-attention-model-training
description: Use when constructing a lineage-specific gene regulatory network from
  scRNA-seq expression (h5ad/csv), a prior interaction network, and differential expression,
  and benchmarking GRN quality with AUPRC/EPR against ChIP-seq ground truth.
when_to_use_negative:
- The goal is to process multiple fate trajectories with bifurcations at once — CEFCON
  can only handle one cell lineage at a time
- Ground-truth label datasets exist for a supervised approach — supervised frameworks
  such as GENELink may yield superior results
- Extrinsic factors (cell–cell communication, spatial location) or epigenetic regulation
  must be modeled — CEFCON only considers intracellular RNA-seq factors
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: CEFCON
  role: Python package providing data_preparation, NetModel (graph attention + DGI
    training), and get_cefcon_results for GRN construction
  repo: https://github.com/WPZgithub/CEFCON
- name: PyTorch
  role: Deep learning backend for training the graph attention network (>=1.13.0,<2.0)
- name: torch-geometric
  role: Graph neural network library used to implement the attention layers (>=2.1.0)
- name: scanpy
  role: Loads and prepares the scRNA-seq AnnData input (>=1.8.2)
- name: networkx
  role: Handles the prior interaction network edgelist and GRN graph structure (>=3.0)
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:16f4e05a2f8ee9b03a8cdd8a7b4d7072c1ffe5726bef3c89274500e0202bba8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WPZgithub/CEFCON
  tools:
  - CEFCON
  - PyTorch
  - Python
  - networkx
  - scanpy
  - torch-geometric
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# graph-attention-model-training

## Summary

Train CEFCON's two-layer graph attention neural network under a deep graph infomax (DGI) contrastive learning framework to construct a cell-lineage-specific GRN from single-cell RNA-seq data, converting degree-scaled attention coefficients into an edge-weighted network.

## When to use

Use when constructing a lineage-specific gene regulatory network from scRNA-seq expression (h5ad/csv), a prior interaction network, and differential expression, and benchmarking GRN quality with AUPRC/EPR against ChIP-seq ground truth.

## When NOT to use

- The goal is to process multiple fate trajectories with bifurcations at once — CEFCON can only handle one cell lineage at a time
- Ground-truth label datasets exist for a supervised approach — supervised frameworks such as GENELink may yield superior results
- Extrinsic factors (cell–cell communication, spatial location) or epigenetic regulation must be modeled — CEFCON only considers intracellular RNA-seq factors

## Inputs

- scRNA-seq expression matrix for one cell lineage (.csv or .h5ad AnnData)
- prior gene interaction network in edgelist format
- gene log fold-change (differential expression) table
- BEELINE lineage datasets, ChIP-seq prior networks and ground-truth networks (Zenodo record 7564872) for benchmarking

## Outputs

- cell_lineage_GRN.csv: edge-list CSV of the inferred cell-lineage-specific GRN
- gene_embs.csv: learned node (gene) representations
- AUPRC and EPR metrics versus baseline methods (SCINET, NetREX, CellOracle, GRNBoost2, DeepSEM, Random_NicheNet)

## How to apply

Combine the scRNA-seq expression matrix, prior gene interaction network (edgelist), and gene log fold-change table with cf.data_preparation, then instantiate the GRN model with cf.NetModel(epochs=350, repeats=3, cuda='0') on PyTorch and torch-geometric. The graph attention layer uses a cosine attention variant ({COS,AD,SD} flags) that incorporates the gene differential-expression encoding into the attention computation, and the network is trained under the DGI contrastive objective. Because attention weights vary across runs, repeat training (repeats=3) and average attention-derived edge weights across repeats. Convert averaged coefficients to a final GRN with get_cefcon_results(edge_threshold_avgDegree=8), which selects top-kd edges scaled by each central node's degree so the output network keeps a target average degree. Save the edge list as cell_lineage_GRN.csv and node embeddings as gene_embs.csv.

## Related tools

- **CEFCON** (Python package providing data_preparation, NetModel (graph attention + DGI training), and get_cefcon_results for GRN construction) — https://github.com/WPZgithub/CEFCON
- **PyTorch** (Deep learning backend for training the graph attention network (>=1.13.0,<2.0))
- **torch-geometric** (Graph neural network library used to implement the attention layers (>=2.1.0))
- **scanpy** (Loads and prepares the scRNA-seq AnnData input (>=1.8.2))
- **networkx** (Handles the prior interaction network edgelist and GRN graph structure (>=3.0))

## Examples

```
cefcon_GRN_model = cf.NetModel(epochs=350, repeats=3, cuda='0'); cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)
```

## Evaluation signals

- cell_lineage_GRN.csv exists as a valid edge-list CSV whose network's average node degree matches edge_threshold_avgDegree (8)
- AUPRC against the lineage's ChIP-seq ground-truth network should approach CEFCON's reported benchmark (64% above DeepSEM, 92% above NetREX)
- EPR should be best or tied-best relative to published baseline results on the tested BEELINE datasets
- Training completes repeats=3 runs at epochs=350 on CUDA and edge weights are averaged across repeats
- gene_embs.csv is written with learned node representations of the expected dimension for all genes

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously
- Only intracellular factors are considered; cell–cell communication, spatial location, and epigenetic information are not modeled
- Limited to RNA-seq; extension to spatial transcriptome, chromatin accessibility, and multi-omics is future work
- Ground-truth label datasets for supervised alternatives are often lacking, especially for specific cell types or lineages

## Evidence

- [intro] CEFCON first employs a graph attention neural network under a contrastive learning framework to construct a cell-lineage-specific GRN.: "CEFCON first employs a graph attention neural network under a contrastive learning framework to construct a cell-lineage-specific GRN."
- [methods] degree-scaled attention coefficients: "data = cf.data_preparation(adata, prior_network)"
- [methods] edge threshold: "cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)"
- [methods] model instantiation: "cefcon_GRN_model = cf.NetModel(epochs=350, repeats=3, cuda='0')"
- [readme] readme method overview: "CEFCON initially employs the graph attention neural networks under a contrastive learning framework to construct reliable GRNs for specific developmental cell lineages"
