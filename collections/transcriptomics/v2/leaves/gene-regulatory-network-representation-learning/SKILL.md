---
name: gene-regulatory-network-representation-learning
description: Use when the input is a directed GRN edge list (e.g. DawnNet, KEGG, RegNetwork)
  plus a gene feature matrix, and the goal is to capture directed regulation information
  and second-order neighbor structure as embeddings.
when_to_use_negative:
- The network is undirected — DGCN's in-/out-degree second-order proximity matrices
  require directed regulation edges
- You only need topological structure without gene features — the convolution ZF =
  sigma(AF X Theta) requires the gene feature matrix X
- You need patient-specific predictions — DGMP averages the SNVs, CNAs, DNA methylation,
  and gene expression of all samples, ignoring individual cancer patients
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_3474
tools:
- name: DGMP
  role: Framework containing the DGCN module that performs graph convolution on the
    directed GRN
  repo: https://github.com/NWPU-903PR/DGMP
- name: PyTorch 1.7.1
  role: Backend for computing normalized proximity matrices and convolution operations
- name: torch-sparse
  role: Sparse graph handling and sparse matrix multiplication for proximity convolutions
- name: torch-scatter
  role: Required for dealing with sparse graphs during adjacency construction
- name: torch-geometric 1.6.3
  role: Graph convolution library under which DGCN (adapted from DiGC) is implemented
  repo: https://github.com/flyingtango/DiGC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2022.11.004
    title: 'DGMP: Identifying Cancer Driver Genes by Jointing DGCN and MLP from Multi-Omics
      Genomic Data'
  source_content_hash: sha256:630db3590f83d7634f81914684e3fdaef3a856bc263cd951b9585545fbb29a53
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
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# gene-regulatory-network-representation-learning

## Summary

Learns node embeddings on a directed gene regulatory network (GRN) by applying the DGCN module of DGMP, which performs graph convolution with three proximity matrices (first-order AF and second-order in-/out-degree Ain, Aout) over multi-omics gene features to produce ZF, Zin, and Zout.

## When to use

Use when the input is a directed GRN edge list (e.g. DawnNet, KEGG, RegNetwork) plus a gene feature matrix, and the goal is to capture directed regulation information and second-order neighbor structure as embeddings.

## When NOT to use

- The network is undirected — DGCN's in-/out-degree second-order proximity matrices require directed regulation edges
- You only need topological structure without gene features — the convolution ZF = sigma(AF X Theta) requires the gene feature matrix X
- You need patient-specific predictions — DGMP averages the SNVs, CNAs, DNA methylation, and gene expression of all samples, ignoring individual cancer patients

## Inputs

- Directed gene regulatory network edge list (e.g. DawnNet, KEGG, RegNetwork)
- Gene feature matrix X from multi-omics data (SNVs, CNAs, DNA methylation, gene expression)
- Trained/untrained trainable weight matrix Theta

## Outputs

- First-order proximity matrix AF (sparse, normalized)
- Second-order proximity matrices Ain and Aout (sparse, normalized)
- Three proximity convolution outputs ZF, Zin, Zout (DGCN embedding vectors)
- Named array file dgcn_proximity_outputs.npz containing AF, Ain, Aout, ZF, Zin, Zout

## How to apply

Build a sparse adjacency tensor from the directed GRN edge list using torch-sparse and torch-scatter (required by DGMP for dealing with sparse graphs). Compute the first-order proximity matrix AF via symmetric normalization of the directed adjacency, and the second-order in-degree and out-degree proximity matrices Ain and Aout by normalizing in- and out-edge co-occurrence structure, keeping all matrices sparse. Apply the three proximity convolutions per Equations (1)-(2): ZF = sigma(AF X Theta), Zin = sigma(Ain X Theta), Zout = sigma(Aout X Theta), where X is the multi-omics gene feature matrix and Theta is a trainable weight matrix. Run end-to-end under the DGMP conda environment (Python 3.7, PyTorch 1.7.1, torch-geometric 1.6.3), persisting the six named arrays (AF, Ain, Aout, ZF, Zin, Zout) for downstream concatenation with the MLP embedding in DGMP.

## Related tools

- **DGMP** (Framework containing the DGCN module that performs graph convolution on the directed GRN) — https://github.com/NWPU-903PR/DGMP
- **PyTorch 1.7.1** (Backend for computing normalized proximity matrices and convolution operations)
- **torch-sparse** (Sparse graph handling and sparse matrix multiplication for proximity convolutions)
- **torch-scatter** (Required for dealing with sparse graphs during adjacency construction)
- **torch-geometric 1.6.3** (Graph convolution library under which DGCN (adapted from DiGC) is implemented) — https://github.com/flyingtango/DiGC

## Examples

```
conda activate DGMP && cd code && python DGMP.py --gpu-no 0 --dataset DawnNet
```

## Evaluation signals

- dgcn_proximity_outputs.npz exists and contains the six named arrays (AF, Ain, Aout, ZF, Zin, Zout)
- Proximity matrix entries fall within the valid normalized range after symmetric/degree normalization
- Script runs end-to-end under the DGMP conda environment (Python 3.7, PyTorch 1.7.1, torch-geometric 1.6.3)
- All three proximity matrices remain sparse throughout computation
- ZF, Zin, Zout are the expected shapes given X and Theta under sigma(AF X Theta), sigma(Ain X Theta), sigma(Aout X Theta)

## Limitations

- Second-order proximity convolutions expand the receptive field but DGCN alone suffers performance degradation that MLP introduction is needed to offset
- Aggregating omics across all samples ignores the specific characteristics of individual cancer patients
- DGCN's learning process can be biased toward topological features over gene features

## Evidence

- [other] DGCN computes ZF = σ(ÂF X Θ), Zin = σ(Âin X Θ), and Zout = σ(Âout X Θ): [evidence span withheld — anchor status: not_found]
- [readme] torch-scatter and torch-sparse are required for dealing with sparse graph: "Besides, torch-scatter and torch-sparse are required for dealing with sparse graph"
- [intro] We use the DGCN model to perform the graph convolutional operation on the directed GRN for capturing the directed information: "We use the DGCN model to perform the graph convolutional operation on the directed GRN for capturing the directed information (i.e., regulation information)"
- [methods] Our project is developed using PyTorch 1.7.1 in Python 3.7.: "Our project is developed using PyTorch 1.7.1 in Python 3.7."
