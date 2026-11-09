---
name: attention-based-edge-weight-aggregation
description: Use when building a lineage-specific GRN from scRNA-seq expression (e.g.,
  .h5ad) plus a prior gene interaction edgelist, where edge weights should derive
  from GNN attention rather than co-expression correlations.
when_to_use_negative:
- Multiple fate trajectories with bifurcations must be modeled simultaneously — CEFCON
  handles only one cell lineage at a time
- Cell–cell communication, spatial location, or epigenetic regulation must be modeled
  — CEFCON only considers intracellular factors from RNA-seq
- A supervised GRN method is feasible with abundant ground-truth labels (e.g., GENELink-style)
  — labels for specific lineages are usually lacking, but if present, supervised frameworks
  may yield superior results
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CEFCON
  role: Provides cf.data_preparation, cf.NetModel (graph attention GNN trained with
    DGI contrastive learning), and get_cefcon_results for attention-based edge weight
    aggregation and top-kd edge selection
  repo: https://github.com/WPZgithub/CEFCON
- name: PyTorch
  role: Deep-learning backend (>=1.13.0,<2.0) for training the graph attention network
  repo: https://pytorch.org/get-started/locally/
- name: torch-geometric
  role: Graph neural network layer implementations (>=2.1.0) underlying the CEFCON
    attention model
  repo: https://pytorch-geometric.readthedocs.io/en/latest/notes/installation.html
- name: scanpy
  role: Reads and preprocesses the scRNA-seq .h5ad expression input (>=1.8.2)
  repo: https://scanpy.readthedocs.io/en/stable/installation.html
- name: networkx
  role: Handles the prior gene interaction network edgelist and graph structure (>=3.0)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:09e5b08ed4a053ebd9c169a86a40d4f26e5f8ed8dc64e9959dcf51bd7fed594c
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# attention-based-edge-weight-aggregation

## Summary

Reconstructs a cell-lineage-specific gene regulatory network (GRN) by training CEFCON's graph attention network under a deep graph infomax (DGI) contrastive objective, then converting degree-scaled attention coefficients into an edge-weighted network.

## When to use

Use when building a lineage-specific GRN from scRNA-seq expression (e.g., .h5ad) plus a prior gene interaction edgelist, where edge weights should derive from GNN attention rather than co-expression correlations.

## When NOT to use

- Multiple fate trajectories with bifurcations must be modeled simultaneously — CEFCON handles only one cell lineage at a time
- Cell–cell communication, spatial location, or epigenetic regulation must be modeled — CEFCON only considers intracellular factors from RNA-seq
- A supervised GRN method is feasible with abundant ground-truth labels (e.g., GENELink-style) — labels for specific lineages are usually lacking, but if present, supervised frameworks may yield superior results

## Inputs

- scRNA-seq expression matrix (.csv or .h5ad) for the target cell lineage
- prior gene interaction network in edgelist format
- gene log fold-change table (differential-expression encoding)

## Outputs

- cell_lineage_GRN.csv — edge list of the cell-lineage-specific GRN
- gene_embs.csv — learned GNN node (gene) representations

## How to apply

First combine the scRNA-seq matrix, prior network edgelist, and gene log fold-change table via cf.data_preparation for the target lineage. Instantiate cf.NetModel(epochs=350, repeats=3, cuda='0'), selecting the cosine attention variant (attention={COS,AD,SD}) that encodes gene differential expression into the attention computation. Train the two-layer graph attention network under the DGI contrastive objective, repeating training 3 times, then average attention-derived edge weights across repeats. Select edges with get_cefcon_results(edge_threshold_avgDegree=8) so the final network's average node degree matches the configured threshold. Rationale: attention coefficients learned by the GAT are scaled by each central node's degree, making degree-scaled coefficients the basis for cell-lineage-specific edge weights.

## Related tools

- **CEFCON** (Provides cf.data_preparation, cf.NetModel (graph attention GNN trained with DGI contrastive learning), and get_cefcon_results for attention-based edge weight aggregation and top-kd edge selection) — https://github.com/WPZgithub/CEFCON
- **PyTorch** (Deep-learning backend (>=1.13.0,<2.0) for training the graph attention network) — https://pytorch.org/get-started/locally/
- **torch-geometric** (Graph neural network layer implementations (>=2.1.0) underlying the CEFCON attention model) — https://pytorch-geometric.readthedocs.io/en/latest/notes/installation.html
- **scanpy** (Reads and preprocesses the scRNA-seq .h5ad expression input (>=1.8.2)) — https://scanpy.readthedocs.io/en/stable/installation.html
- **networkx** (Handles the prior gene interaction network edgelist and graph structure (>=3.0))

## Examples

```
cefcon_GRN_model = cf.NetModel(epochs=350, repeats=3, cuda='0'); cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)
```

## Evaluation signals

- cell_lineage_GRN.csv exists and parses as a valid edge-list CSV
- Resulting network's average node degree matches the configured edge_threshold_avgDegree (8)
- Attention-derived edge weights are averaged consistently across repeats=3 training runs
- gene_embs.csv contains one learned representation per gene in the input network
- GRN structure matches the benchmark example (hESC dataset, 1,000 highly variable genes)

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously
- Only intracellular factors are modeled; cell–cell communication and spatial location are not included
- Epigenetic information controlling cell fate decisions is not incorporated; RNA-seq only — spatial transcriptome, chromatin accessibility, and multi-omics extensions are future work

## Evidence

- [abstract] abstract: "first uses a graph neural network with attention mechanism to infer a cell-lineage-specific gene regulatory network (GRN) from single-cell"
- [intro] intro: "CEFCON first employs a graph attention neural network under a contrastive learning framework to construct a cell-lineage-specific GRN."
- [methods] methods: "cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)"
- [readme] readme: "CEFCON initially employs the graph attention neural networks under a contrastive learning framework to construct reliable GRNs"
