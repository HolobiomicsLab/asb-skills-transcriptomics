---
name: mxnet-symbolic-model-definition
description: Use when building an 'explainable' neural deconvolution model in mxnet
  where each hidden neuron must map to a defined biological entity (cell type) via
  a masked gene–cell relation matrix, trained with adam/ReLU/L2loss on bulk RNA-seq.
when_to_use_negative:
- The input is not bulk RNA-seq gene expression (e.g. already a per-cell single-cell
  matrix or a precomputed cell abundance table) — DCNet is designed to deconvolve
  bulk samples into cell-type content
- A non-transparent, freely learned network is acceptable and the masked, biologically
  constrained connectivity is not needed — the mask is the core interpretability mechanism
  here
- Marker genes and cell types are not from the human CellMarker landscape (9078 genes
  / 434 cells) and the gene-to-neuron index mappings cannot be reused
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3170
tools:
- name: mxnet
  role: Gluon framework used to code and define the masked three-layer DCNet model
    in Python
- name: Python
  role: Language for defining the network, loading relation_matrix.pkl/index.pkl via
    pickle, and saving parameters
- name: DCNet
  role: The deep learning frame being defined — embeds the relationships between 434
    cell types and 9078 marker genes in the network structure
  repo: https://github.com/xindd/DCNet
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:a116dbeee95247e65c22d45f2206b7e2eb4c9cfa6e05be539cdad7aeacaa977f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  - mxnet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# mxnet-symbolic-model-definition

## Summary

Define an interpretable three-layer mxnet (gluon) neural network with a 9078–434–21136 topology whose hidden-layer neurons correspond one-to-one to 434 cell types, embedding the gene–cell marker relation matrix as a mask on the input-to-hidden weights so latent activations are directly interpretable as cell abundances.

## When to use

Use when building an 'explainable' neural deconvolution model in mxnet where each hidden neuron must map to a defined biological entity (cell type) via a masked gene–cell relation matrix, trained with adam/ReLU/L2loss on bulk RNA-seq.

## When NOT to use

- The input is not bulk RNA-seq gene expression (e.g. already a per-cell single-cell matrix or a precomputed cell abundance table) — DCNet is designed to deconvolve bulk samples into cell-type content
- A non-transparent, freely learned network is acceptable and the masked, biologically constrained connectivity is not needed — the mask is the core interpretability mechanism here
- Marker genes and cell types are not from the human CellMarker landscape (9078 genes / 434 cells) and the gene-to-neuron index mappings cannot be reused

## Inputs

- One-zero gene–cell relation matrix (relation_matrix.pkl) linking 434 cell types to 9078 marker genes
- Gene-to-neuron index mappings (index.pkl) for input and output neurons
- Gene expression matrix with columns = samples and rows = human gene symbols, log2(x+1)-transformed (demo_data.pkl as example)
- Preprocessed CellMarker marker gene data (13,605 marker genes across 467 cell types before filtering)
- TCGA RNA-seq expression profiles split 80/20 into train/test sets

## Outputs

- dcnet_model.py / gluon network definition exposing layers of 9078/434/21136 nodes with masked input-to-hidden connectivity
- Saved parameter file (e.g. net_M434_adam_T600_DCNetBnoise-D1.params) containing weights and biases via net.save_parameters
- Latent-layer activation values representing inferred abundances of 434 cell types per sample

## How to apply

In Python with mxnet/gluon, define a three-layer network with input dimension 9078, hidden (latent) dimension 434, and output dimension 21136. Load the one-zero gene–cell correspondence matrix (relation_matrix.pkl) and the gene-to-neuron index mappings (index.pkl) via pickle, then element-wise mask the input-to-hidden weight connections with the relation matrix so each hidden neuron (cell type) connects only to its marker-gene input neurons. Apply ReLU activations on both the hidden and output layers and configure L2loss, per the paper's mxnet parameter settings. Train with the adam optimizer, learning rate 1e-4, 600 iterations, 256 samples per batch, CPU training, and save parameters via net.save_parameters (e.g. net_M434_adam_T600_DCNetBnoise-D1.params). The latent layer activation values are the inferred abundances of the 434 cell types.

## Related tools

- **mxnet** (Gluon framework used to code and define the masked three-layer DCNet model in Python)
- **Python** (Language for defining the network, loading relation_matrix.pkl/index.pkl via pickle, and saving parameters)
- **DCNet** (The deep learning frame being defined — embeds the relationships between 434 cell types and 9078 marker genes in the network structure) — https://github.com/xindd/DCNet

## Examples

```
import mxnet as mx; from mxnet import gluon; net = gluon.nn.Sequential(); net.add(gluon.nn.Dense(434, activation='relu'), gluon.nn.Dense(21136, activation='relu')); net.initialize(); net.save_parameters('net_M434_adam_T600_DCNetBnoise-D1.params')
```

## Evaluation signals

- The constructed network exposes layers of 9078/434/21136 nodes matching the index.pkl gene-to-neuron mappings
- The input-to-hidden weight mask matches the one-zero relation matrix from relation_matrix.pkl (each hidden neuron connects only to its marker-gene inputs)
- A forward pass on the demo expression matrix (samples as columns, gene symbols as rows, log2(x+1)-transformed) completes without dimension errors
- net.save_parameters writes a params file (e.g. net_M434_adam_T600_DCNetBnoise-D1.params) that reloads into the same architecture
- Training configuration matches the published setup: adam optimizer, relu activation, L2loss, learning rate 1e-4, 600 iterations, 256 samples per batch, CPU

## Limitations

- No software version numbers are reported for mxnet, and training was CPU-only with no runtime reported
- No random seed or other reproducibility controls for model training are mentioned
- Trained model weights exist as downloadable params but fine-tuning the recommended dropout-0.1 DCNet on user sample data is not specified
- Random gene dropout probabilities (0.1/0.3/0.5) were used for augmentation, but which setting was used for the final model is not stated

## Evidence
