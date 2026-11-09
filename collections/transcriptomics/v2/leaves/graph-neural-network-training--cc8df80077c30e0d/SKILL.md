---
name: graph-neural-network-training
description: Use when constructing a lineage-specific GRN from scRNA-seq data (e.g.,
  h5ad/adata with 1,000 HVGs) given a prior gene interaction network, and downstream
  driver-regulator analysis requires reliable inferred regulatory edges.
when_to_use_negative:
- No prior gene interaction network is available and the goal is to infer a GRN from
  scRNA-seq data alone
- Multiple fate trajectories with bifurcations must be processed simultaneously —
  CEFCON can only handle one cell lineage at a time
- The analysis requires extrinsic factors such as cell–cell communication, spatial
  location, or epigenetics, which CEFCON does not model
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CEFCON
  role: Provides the graph attention neural network (NetModel) trained under a contrastive
    learning framework to construct the cell-lineage-specific GRN
  repo: https://github.com/WPZgithub/CEFCON
- name: PyTorch
  role: Deep-learning backend used to train the GNN (>=1.13.0,<2.0), with CUDA GPU
    acceleration
- name: torch-geometric
  role: Graph neural network library (>=2.1.0) supporting CEFCON's graph attention
    layers
- name: networkx
  role: Loads and manipulates the prior gene interaction network edgelist before GNN
    training
- name: scanpy
  role: Prepares the scRNA-seq expression input (adata) for data_preparation (>=1.8.2)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-44103-3
    title: Deciphering driver regulators of cell fate decisions from single-cell transcriptomics
      data with CEFCON
  source_content_hash: sha256:95c35218497e5966287aa65b1f6e4dafe9d62dd2edec07604949134b8951be0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WPZgithub/CEFCON
  tools:
  - CEFCON
  - PRROC
  - PyTorch
  - Python
  - networkx
  - pandas
  - scanpy
  - scikit-learn
  - torch-geometric
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# graph-neural-network-training

## Summary

Trains a graph attention neural network under a contrastive learning framework (CEFCON) to infer a cell-lineage-specific gene regulatory network from single-cell RNA-seq expression plus a prior gene interaction network.

## When to use

Use when constructing a lineage-specific GRN from scRNA-seq data (e.g., h5ad/adata with 1,000 HVGs) given a prior gene interaction network, and downstream driver-regulator analysis requires reliable inferred regulatory edges.

## When NOT to use

- No prior gene interaction network is available and the goal is to infer a GRN from scRNA-seq data alone
- Multiple fate trajectories with bifurcations must be processed simultaneously — CEFCON can only handle one cell lineage at a time
- The analysis requires extrinsic factors such as cell–cell communication, spatial location, or epigenetics, which CEFCON does not model

## Inputs

- scRNA-seq expression matrix (scanpy adata, e.g., hESC dataset with 1,000 highly variable genes)
- Prior gene interaction network edgelist (human/mouse, loadable with networkx)
- Cell lineage/trajectory assignment for the developmental lineage under study

## Outputs

- Cell-lineage-specific GRN (thresholded edge list with edge weights, ~average degree 8)
- CEFCON results object supporting gene_influence_score(), driver_regulators(), and RGM_activity() downstream
- AUPRC evaluation results against ground-truth edges (e.g., perturbed_auprc_results.csv per dataset)

## How to apply

Load the prior gene interaction network and scRNA-seq expression (e.g., the hESC dataset with 1,000 highly variable genes from Zenodo record 7564872) and prepare inputs with cf.data_preparation(adata, prior_network). Instantiate the GNN as cf.NetModel(epochs=350, repeats=3, cuda='0') and train per cell lineage using PyTorch (>=1.13.0,<2.0) and torch-geometric (>=2.1.0), preferably on an NVIDIA GPU with CUDA as the README recommends. Retrieve inferred GRNs via get_cefcon_results(edge_threshold_avgDegree=8), which thresholds predicted edges to a network of roughly average-degree 8. Rationale: the attention-based contrastive framework leverages the prior network topology rather than expression alone, because GRNs inferred from scRNA-seq data alone are not accurate enough. Evaluate edge predictions against ground-truth regulatory edges by computing AUPRC with the PRROC R package.

## Related tools

- **CEFCON** (Provides the graph attention neural network (NetModel) trained under a contrastive learning framework to construct the cell-lineage-specific GRN) — https://github.com/WPZgithub/CEFCON
- **PyTorch** (Deep-learning backend used to train the GNN (>=1.13.0,<2.0), with CUDA GPU acceleration)
- **torch-geometric** (Graph neural network library (>=2.1.0) supporting CEFCON's graph attention layers)
- **networkx** (Loads and manipulates the prior gene interaction network edgelist before GNN training)
- **scanpy** (Prepares the scRNA-seq expression input (adata) for data_preparation (>=1.8.2))

## Examples

```
cefcon_GRN_model = cf.NetModel(epochs=350, repeats=3, cuda='0')
```

## Evaluation signals

- AUPRC of inferred edges vs. ground-truth regulatory edges computed with the PRROC R package should exceed baseline methods
- Robustness check: randomly shuffling 20% of prior-network edges should drop AUPRC only ~10% on average across benchmark datasets (Supplementary Fig. 3)
- Training config matches the paper: epochs=350, repeats=3, cuda='0'; results retrieved with get_cefcon_results(edge_threshold_avgDegree=8)
- Perturbed-run output file perturbed_auprc_results.csv exists with AUPRC columns for each dataset
- Environment satisfies README constraints: Python 3.9~3.10, pytorch(>=1.13.0,<2.0), torch-geometric(>=2.1.0)

## Limitations

- CEFCON can only handle one cell lineage at a time and cannot process multiple fate trajectories with bifurcations simultaneously
- Only intracellular factors are modeled; cell–cell communication, spatial location, and epigenetics are not incorporated
- Requires specific environment: Python (3.9~3.10), pytorch(>=1.13.0,<2.0), torch-geometric(>=2.1.0); GPU with CUDA is recommended
- Results depend on prior-network quality: 20% random edge shuffling caused ~10% average AUPRC drop

## Evidence

- [intro] GNN contrastive framework: "CEFCON first employs a graph attention neural network under a contrastive learning framework to construct a cell-lineage-specific GRN."
- [methods] GNN model instantiation: "# Construct cell-lineage-specific GRN
cefcon_GRN_model = cf.NetModel(epochs=350, repeats=3, cuda='0')"
- [methods] Result retrieval and thresholding: "cefcon_results = cefcon_GRN_model.get_cefcon_results(edge_threshold_avgDegree=8)"
- [readme] Environment constraints: "We recommend running CEFCON on CUDA if possible."
- [methods] Python version tested: "CEFCON was originally tested on Ubuntu 20.04 with Python (3.9~3.10)"
- [methods] Installation command: "pip install git+https://github.com/WPZgithub/CEFCON.git"
