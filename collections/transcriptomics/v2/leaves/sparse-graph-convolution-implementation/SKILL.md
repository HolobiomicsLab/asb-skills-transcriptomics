---
name: sparse-graph-convolution-implementation
description: Use when you have a directed gene regulatory network (e.g. DawnNet edge
  list) plus a gene feature matrix and must compute directed graph convolution outputs
  while keeping all adjacency matrices sparse.
when_to_use_negative:
- The network is undirected or edge direction/regulation information is absent — DGCN's
  second-order in-/out-degree proximity matrices require directed edges
- The goal is node-level gene feature learning that must weigh multi-omics features
  over topology; DGMP augments DGCN with an MLP branch precisely because DGCN alone
  biases toward topological features
- Input is already a dense precomputed embedding rather than a sparse graph — sparse
  graph libraries add overhead with no benefit
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
tools:
- name: DGMP
  role: Reference implementation whose DGCN module defines the three proximity convolutions
    and output schema
  repo: https://github.com/NWPU-903PR/DGMP
- name: PyTorch 1.7.1
  role: Backend for computing proximity matrices and trainable convolution weights
    Θ
- name: torch-sparse
  role: Sparse graph handling for the AF/Ain/Aout matrices and their multiplications
    with X
- name: torch-scatter
  role: Required scatter operations for sparse-graph message passing in the DGCN layer
- name: torch-geometric 1.6.3
  role: Graph convolution framework (with DiGC template) used to implement the directed
    convolution layers
  repo: https://github.com/flyingtango/DiGC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:319873ff576ac1fc468abcb7c62b5a19c3b5db65503468e5b73b9e2f450f1629
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NWPU-903PR/DGMP
  tools:
  - DGMP
  - PyTorch 1.7.1
  - Python
  - torch-geometric
  - torch-geometric 1.6.3
  - torch-scatter
  - torch-sparse
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# sparse-graph-convolution-implementation

## Summary

Reconstructs the DGCN module from DGMP, which applies three sparse proximity convolutions (first-order ZF and second-order in-/out-degree Zin, Zout) over a directed gene regulatory network, per Equations (1)-(2): Z = σ(ÂXΘ).

## When to use

Use when you have a directed gene regulatory network (e.g. DawnNet edge list) plus a gene feature matrix and must compute directed graph convolution outputs while keeping all adjacency matrices sparse.

## When NOT to use

- The network is undirected or edge direction/regulation information is absent — DGCN's second-order in-/out-degree proximity matrices require directed edges
- The goal is node-level gene feature learning that must weigh multi-omics features over topology; DGMP augments DGCN with an MLP branch precisely because DGCN alone biases toward topological features
- Input is already a dense precomputed embedding rather than a sparse graph — sparse graph libraries add overhead with no benefit

## Inputs

- Directed gene regulatory network edge list (e.g. DawnNet, KEGG, or RegNetwork)
- Gene feature matrix X (multi-omics features averaged across samples: SNVs, CNAs, DNA methylation, gene expression)
- Trainable weight matrix Θ (DGCN layer parameters)

## Outputs

- First-order proximity matrix AF (normalized, sparse)
- Second-order in-degree proximity matrix Ain and out-degree proximity matrix Aout (normalized, sparse)
- Three convolution outputs ZF, Zin, Zout stored as named arrays in dgcn_proximity_outputs.npz

## How to apply

Load the directed GRN edge list and build a sparse adjacency in PyTorch using torch-sparse and torch-scatter, which DGMP requires for sparse graphs. Compute the first-order proximity matrix AF by symmetric normalization of the directed edge structure, then compute second-order in-degree (Ain) and out-degree (Aout) proximity matrices from in-/out-edge co-occurrence per Equations 1-3, keeping all matrices sparse. Apply the three convolutions ZF = σ(ÂF X Θ), Zin = σ(Âin X Θ), and Zout = σ(Âout X Θ), where X is the gene feature matrix and Θ is a trainable weight matrix, using sparse matrix multiplication via torch-geometric. Persist AF, Ain, Aout, ZF, Zin, and Zout as named arrays in dgcn_proximity_outputs.npz. Target the DGMP conda environment (Python 3.7, PyTorch 1.7.1, torch-geometric 1.6.3) to match the DGMP runtime.

## Related tools

- **DGMP** (Reference implementation whose DGCN module defines the three proximity convolutions and output schema) — https://github.com/NWPU-903PR/DGMP
- **PyTorch 1.7.1** (Backend for computing proximity matrices and trainable convolution weights Θ)
- **torch-sparse** (Sparse graph handling for the AF/Ain/Aout matrices and their multiplications with X)
- **torch-scatter** (Required scatter operations for sparse-graph message passing in the DGCN layer)
- **torch-geometric 1.6.3** (Graph convolution framework (with DiGC template) used to implement the directed convolution layers) — https://github.com/flyingtango/DiGC

## Examples

```
conda create -n DGMP python=3.7; conda activate DGMP; conda install pytorch==1.7.1 torchvision==0.8.2 cudatoolkit=11.0; pip install torch-sparse==0.6.8 -f https://pytorch-geometric.com/whl/torch-1.7.1+cu110.html
```

## Evaluation signals

- dgcn_proximity_outputs.npz exists and contains all six named arrays (AF, Ain, Aout, ZF, Zin, Zout)
- All proximity matrix entries fall within the valid normalized range after symmetric/degree normalization
- ZF, Zin, Zout shapes match the gene count in the input GRN × Θ's output dimension
- Script runs end-to-end under the DGMP conda environment (Python 3.7, PyTorch 1.7.1, torch-geometric 1.6.3) without dense-matrix blowups
- All intermediate proximity matrices remain sparse (torch_sparse/torch_scatter types) rather than densified

## Limitations

- DGCN alone shows performance degradation; in DGMP it must be joined with an MLP to offset this bias toward topological features
- Averaging omics features across samples to form X ignores individual patient-specific characteristics
- Version pinning is strict: torch-sparse 0.6.8 and torch-scatter 2.0.6 wheels are built for torch-1.7.1+cu110

## Evidence

- [other] DGCN computes ZF = σ(ÂF X Θ), Zin = σ(Âin X Θ), and Zout = σ(Âout X Θ): [evidence span withheld — anchor status: not_found]
- [readme] torch-scatter and torch-sparse are required for dealing with sparse graph: "Besides, torch-scatter and torch-sparse are required for dealing with sparse graph"
- [intro] We use the DGCN model to perform the graph convolutional operation on the directed GRN for capturing the directed information: "We use the DGCN model to perform the graph convolutional operation on the directed GRN for capturing the directed information (i.e., regulation information)"
- [methods] Our project is developed using PyTorch 1.7.1 in Python 3.7.: "Our project is developed using PyTorch 1.7.1 in Python 3.7."
- [methods] python DGMP.py --gpu-no 0 --dataset DawnNet: "python DGMP.py --gpu-no 0 --dataset DawnNet"
