---
name: semi-supervised-label-propagation
description: Use when single-cell RNA-seq cells already have partially confident labels
  from a consensus of annotation tools (e.g. scPred, SingleR, ScType, SCINA, scSorter)
  and you need to propagate those labels to unconfident cells with a graph convolutional
  network.
when_to_use_negative:
- No confident seed labels exist yet — run the tool-consensus step first, since the
  GCN propagates labels from component tool predictions
- Input is already a fully annotated feature table where no unconfident cells remain
  to be classified
- Data is not single-cell transcriptomic data — the GCN operates on cell-neighbor
  graphs built from scRNA-seq profiles
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scSHARP
  role: GCN model class that propagates labels from component tool predictions; installed
    via pip or conda in a Python 3.9 environment
  repo: https://github.com/lewinsohndp/scSHARP
- name: mperozek11/scSHARP_tool
  role: Python package implementation of the GCN label propagation tool
  repo: https://github.com/mperozek11/scSHARP_tool
- name: PyTorch
  role: Framework for building and training the EdgeConv GCN model
- name: pytorch-cluster
  role: Neighbor sampling for constructing the cell-neighbor graph; installed from
    the pyg conda channel
- name: conda
  role: Environment management (Python 3.9 environment) for the scSHARP pipeline
- name: R4scSHARP
  role: Upstream R package running the five component annotation tools whose predictions
    feed the GCN
  repo: https://github.com/W-Holtz/R4scSHARP
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad360
    title: Consensus label propagation with graph convolutional networks for single-cell
      RNA sequencing cell type annotation
  source_content_hash: sha256:1969c4c9ca56f15cd67fef072cf5017033ed9a716b03371c8050586c4b5f71b1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lewinsohndp/scSHARP
  tools:
  - PyTorch
  - Python
  - R4scSHARP
  - conda
  - mperozek11/scSHARP_tool
  - pip
  - pytorch-cluster
  - scSHARP
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# semi-supervised-label-propagation

## Summary

Spread confident consensus cell type labels across the cell graph using a semi-supervised EdgeConv GCN (scSHARP), classifying remaining unconfident cells from their transcriptomic profiles. This improves annotation accuracy over a simple max consensus and the average of the component annotation tools.

## When to use

Use when single-cell RNA-seq cells already have partially confident labels from a consensus of annotation tools (e.g. scPred, SingleR, ScType, SCINA, scSorter) and you need to propagate those labels to unconfident cells with a graph convolutional network.

## When NOT to use

- No confident seed labels exist yet — run the tool-consensus step first, since the GCN propagates labels from component tool predictions
- Input is already a fully annotated feature table where no unconfident cells remain to be classified
- Data is not single-cell transcriptomic data — the GCN operates on cell-neighbor graphs built from scRNA-seq profiles

## Inputs

- Per-dataset confident cell type labels produced by the upstream tool-consensus step (task_001)
- Single-cell RNA-seq transcriptomic profiles (raw counts matrix) to be classified
- Dataset-specific GCN hyperparameters (batch size, neighbors k, EdgeConv layers l, embedding size e)

## Outputs

- Trained GCN model checkpoints (.pt files, one per random initialization)
- Cell type predictions for all cells (confident and unconfident)
- Total accuracy and unconfident-cell accuracy per dataset, comparable to Table 1

## How to apply

Load the per-dataset confident cell labels from the upstream consensus step together with the raw-count single-cell transcriptomic profiles. Build the cell-neighbor graph with pytorch-cluster neighbor sampling in PyTorch and train the scSHARP GCNModel with EdgeConv layers, using the dataset-specific optimal hyperparameters (batch sizes 20/20/50, neighbor count k=2, 2 layers, embedding dimensions 25/40/25 for sim-0.7/sim-0.8-sim/testis/PBMC). Train five random initializations per dataset for 150 epochs with the Adam optimizer at learning rate 0.0001 and cross-entropy loss on the confident labels, saving each model as a .pt checkpoint. Predict cell types for all cells with each initialization, then evaluate total accuracy and accuracy restricted to unconfident cells. Compare against the max consensus, nonparametric neighbor majority, the five component tools, and their average; on the testis dataset the GCN achieved a 14.1% improvement over the tool average.

## Related tools

- **scSHARP** (GCN model class that propagates labels from component tool predictions; installed via pip or conda in a Python 3.9 environment) — https://github.com/lewinsohndp/scSHARP
- **mperozek11/scSHARP_tool** (Python package implementation of the GCN label propagation tool) — https://github.com/mperozek11/scSHARP_tool
- **PyTorch** (Framework for building and training the EdgeConv GCN model)
- **pytorch-cluster** (Neighbor sampling for constructing the cell-neighbor graph; installed from the pyg conda channel)
- **conda** (Environment management (Python 3.9 environment) for the scSHARP pipeline)
- **R4scSHARP** (Upstream R package running the five component annotation tools whose predictions feed the GCN) — https://github.com/W-Holtz/R4scSHARP

## Examples

```
conda create -n scsharp python=3.9; conda activate scsharp; pip install scSHARP; conda install pytorch-cluster -c pyg
```

## Evaluation signals

- Accuracy table must show the GCN outperforming the simple max consensus and the component-tool average on total accuracy
- GCN accuracy must match or exceed the nonparametric neighbor majority baseline (within the model's standard deviation on the testis dataset)
- Testis dataset should reproduce the reported 14.1% improvement over the tool average
- All five random initializations train for 150 epochs with Adam at learning rate 0.0001 and cross-entropy loss, saving .pt checkpoints
- Optimal hyperparameters match reported values: batch 20/20/50, k=2, 2 EdgeConv layers, embedding size 25/40/25 for sim-0.7/sim-0.8-sim/testis/PBMC

## Limitations

- Nonparametric neighbor majority approach achieved accuracy within the model's standard deviation on the testis dataset, so the GCN is not always a clear improvement
- Missing software version numbers for scSHARP and its dependencies; R tools are a required dependency
- Installation of torch and torch geometric is required prior to pip install of scSHARP, and torch-cluster may need manual installation from the pyg channel
- README specifies raw counts must be input ('Please input raw counts')

## Evidence

- [abstract] using state-of-the-art tools to find cells with highly confident label assignments through consensus: "using state-of-the-art tools to find cells with highly confident label assignments through consensus"
- [abstract] spreading these confident labels with a semi-supervised GCN: "and spreading these confident labels with a semi-supervised GCN."
- [abstract] improves accuracy over simple consensus and tool average: "we show that our method improves accuracy over a simple consensus algorithm and the average of the underlying tools."
- [abstract] comparable to neighbor majority: "We also compare our method to a nonparametric neighbor majority approach, showing comparable results."
- [methods] conda pytorch-cluster install: "``conda install pytorch-cluster -c pyg``"
- [methods] conda Python 3.9 environment: "``conda create -n <env name> python=3.9``"
- [methods] pip install scSHARP: "Install with Pip if you plan on running R4scSHARP from Python. ``pip install scSHARP``"
- [methods] GCN propagates labels from component tool predictions: "GCN model class propagates labels from component tool predictions"
- [readme] README: raw counts input: "See demo.ipynb for example work flow.
Please input raw counts."
- [readme] README: torch-cluster manual install: [evidence span withheld — anchor status: external_doc]
