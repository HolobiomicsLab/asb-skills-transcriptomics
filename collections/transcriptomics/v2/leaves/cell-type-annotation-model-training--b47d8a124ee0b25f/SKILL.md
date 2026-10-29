---
name: cell-type-annotation-model-training
description: Use when confident cell type labels from a consensus of annotation tools
  must be spread to unlabeled cells in scRNA-seq data via a graph convolutional network,
  or when tuning GCN hyperparameters per dataset.
when_to_use_negative:
- When no confident labels exist yet — the GCN is semi-supervised and depends on the
  consensus step producing highly confident label assignments first
- When a simple nonparametric neighbor majority approach suffices — the paper shows
  it gives comparable results to the GCN
- When the input is already processed/normalized embeddings rather than raw counts,
  since the pipeline expects raw counts
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scSHARP
  role: Provides the GCNModel class with EdgeConv layers used to propagate labels
    from component tool predictions
  repo: https://github.com/lewinsohndp/scSHARP
- name: PyTorch
  role: Framework in which the end-to-end classification pipeline and GCN training
    are implemented
  repo: https://github.com/lewinsohndp/scSHARP
- name: pytorch-cluster
  role: Neighbor sampling used to construct the cell-neighbor graph for GCN training
- name: scSHARP_tool
  role: Python package distribution of the GCN label propagation tool
  repo: https://github.com/mperozek11/scSHARP_tool
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:cc6876f98efb28734d563dc31ec025efb293fe194620a5bfdf77db27e9a7462e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lewinsohndp/scSHARP
  tools:
  - PyTorch
  - pytorch-cluster
  - scSHARP
  - scSHARP_tool
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cell-type-annotation-model-training

## Summary

Train the scSHARP semi-supervised GCN (EdgeConv layers in PyTorch) to propagate confident consensus cell type labels across an scRNA-seq cell-neighbor graph, sweeping batch size b, neighbors k, layer count l, and embedding size e to select the highest-validation-accuracy architecture.

## When to use

Use when confident cell type labels from a consensus of annotation tools must be spread to unlabeled cells in scRNA-seq data via a graph convolutional network, or when tuning GCN hyperparameters per dataset.

## When NOT to use

- When no confident labels exist yet — the GCN is semi-supervised and depends on the consensus step producing highly confident label assignments first
- When a simple nonparametric neighbor majority approach suffices — the paper shows it gives comparable results to the GCN
- When the input is already processed/normalized embeddings rather than raw counts, since the pipeline expects raw counts

## Inputs

- Confident cell type labels from the upstream consensus step (task_001) over the scRNA-seq datasets (sim-0.7, sim-0.8-sim, testis, PBMC)
- Raw counts single-cell RNA-seq gene expression matrix (README: 'Please input raw counts.')
- Candidate GCN hyperparameter configurations: batch size b, neighbors k, EdgeConv layer count l, embedding size e

## Outputs

- Trained scSHARP GCNModel per dataset with propagated cell type labels for all cells
- selected_hyperparameters.csv listing the highest-validation-accuracy configuration per dataset
- Validation accuracy per candidate configuration

## How to apply

First obtain confident cell type labels from the upstream consensus step, then hold out 20% of confidently labeled cells per dataset as a validation split (subsampling PBMC to 5% of its confident cells). Build the cell-neighbor graph using pytorch-cluster neighbor sampling in PyTorch and train the scSHARP GCNModel with EdgeConv layers, sweeping batch size b, neighbor count k, EdgeConv layer count l, and embedding size e. Score each configuration by validation accuracy on the held-out confidently labeled cells and select the configuration with the highest validation accuracy per dataset. For simulated datasets the selected optimum was batch size 20, k=2 nearest neighbors, and 2 EdgeConv layers with a 25-dimensional embedding (batch 20/20/50 and embedding 25/40/25 across sim-0.7/sim-0.8-sim/testis/PBMC), giving a target against which new runs can be compared. Write the selected hyperparameters per dataset to selected_hyperparameters.csv for downstream reuse.

## Related tools

- **scSHARP** (Provides the GCNModel class with EdgeConv layers used to propagate labels from component tool predictions) — https://github.com/lewinsohndp/scSHARP
- **PyTorch** (Framework in which the end-to-end classification pipeline and GCN training are implemented) — https://github.com/lewinsohndp/scSHARP
- **pytorch-cluster** (Neighbor sampling used to construct the cell-neighbor graph for GCN training)
- **scSHARP_tool** (Python package distribution of the GCN label propagation tool) — https://github.com/mperozek11/scSHARP_tool

## Examples

```
conda create -n <env name> python=3.9; conda activate <env name>; pip install scSHARP
```

## Evaluation signals

- Selected hyperparameters for simulated datasets should match reported optima: batch size 20, k=2 nearest neighbors, 2 EdgeConv layers, embedding size 25 for the 0.7 de.facScale simulation
- Reported batch sizes 20/20/50 and embedding sizes 25/40/25 for sim-0.7/sim-0.8-sim/testis/PBMC respectively serve as reference targets
- Validation accuracy must be computed on the held-out 20% of confidently labeled cells (PBMC subsampled to 5% of confident cells)
- selected_hyperparameters.csv must contain one row per dataset with batch size, k, layer count, and embedding size
- GCN accuracy should improve over a simple consensus algorithm and the average of the underlying component tools

## Limitations

- Torch and torch geometric must be installed prior to pip install of scSHARP; additional PyTorch Geometric dependencies such as torch-cluster may need manual installation
- Software version numbers for scSHARP and its dependencies, and the R version requirement, are not specified in the methods
- R tools (R4scSHARP) are a necessary dependency for running scSHARP
- The R4scSHARP package was only available via GitHub installation (devtools::install_github) at time of writing

## Evidence

- [abstract] spread confident labels with a semi-supervised GCN: "using state-of-the-art tools to find cells with highly confident label assignments through consensus and spreading these confident labels with a semi-supervised GCN."
- [methods] GCN model class propagates labels from component tool predictions: "GCN model class propagates labels from component tool predictions"
- [methods] pytorch-cluster installation instruction: "``conda install pytorch-cluster -c pyg``"
- [readme] raw counts input requirement: "See demo.ipynb for example work flow.
Please input raw counts."
- [abstract] accuracy improvement claim: "we show that our method improves accuracy over a simple consensus algorithm and the average of the underlying tools."
