---
name: masked-neural-network-architecture-construction
description: Use when bulk RNA-seq samples (rows = human gene symbols, columns = samples)
  must be deconvolved into >400 cell-type abundances and an interpretable, marker-gene-masked
  network is needed instead of black-box deconvolution methods.
when_to_use_negative:
- The goal is estimating a small predefined panel of cell types (e.g. 6–7 immune subsets)
  where CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER or xCell already suffice
- Input expression rows are not human gene symbols or the matrix cannot be aligned
  to the 9078 marker-gene index in index.pkl
- No gene–cell relation matrix is available, since masking each hidden neuron to its
  marker genes is the core interpretability mechanism
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: DCNet
  role: The masked deconvolution network being constructed; embeds 434 cell–marker-gene
    relationships in the ANN structure
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Gluon framework used to code the DCNet model, apply the weight mask, and train
    with adam/ReLU/L2loss settings
- name: Python
  role: Implementation language; pickle-loading of index.pkl, relation_matrix.pkl
    and demo_data.pkl, and saving the model definition
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:4ca4c057887cdf4a26a9c4be97e10b533f09cb24229c2e121e60b76c812f337e
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  - mxnet
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# masked-neural-network-architecture-construction

## Summary

Constructs DCNet, an explainable three-layer mxnet/gluon neural network (9078 input genes → 434 cell-type hidden neurons → 21136 output genes) whose input-to-hidden weights are element-wise masked by a gene–cell zero-one relation matrix, so the latent layer's activation directly encodes 434 cell-type abundances from bulk RNA-seq.

## When to use

Use when bulk RNA-seq samples (rows = human gene symbols, columns = samples) must be deconvolved into >400 cell-type abundances and an interpretable, marker-gene-masked network is needed instead of black-box deconvolution methods.

## When NOT to use

- The goal is estimating a small predefined panel of cell types (e.g. 6–7 immune subsets) where CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER or xCell already suffice
- Input expression rows are not human gene symbols or the matrix cannot be aligned to the 9078 marker-gene index in index.pkl
- No gene–cell relation matrix is available, since masking each hidden neuron to its marker genes is the core interpretability mechanism

## Inputs

- relation_matrix.pkl: zero-one matrix of gene–cell (9078 marker genes × 434 cells) correspondence
- index.pkl: set of genes corresponding to input and output neurons
- demo_data.pkl: gene expression matrix, each column a sample, each row a human gene symbol, log2(x+1)-transformed
- net_M434_adam_T600_DCNetBnoise-D1.params: pretrained DCNet weights/biases

## Outputs

- dcnet_model.py: mxnet network definition with 9078/434/21136 nodes and masked input-to-hidden connectivity
- Latent-layer activations giving 434 cell-type abundance values per bulk RNA-seq sample

## How to apply

Load the one-zero gene–cell correspondence matrix (relation_matrix.pkl / rix.pkl) and the gene-to-neuron index mappings (index.pkl) with Python pickle. Define an mxnet (gluon) three-layer network with input dimension 9078, hidden dimension 434, and output dimension 21136, then element-wise mask the input-to-hidden weights with the relation matrix P so each hidden neuron (cell type) connects only to its marker-gene input neurons. Apply ReLU activations on the hidden and output layers and configure L2 loss, matching the published training setup (adam optimizer, learning rate 1e-4, 600 iterations, 256 samples per batch, CPU training). Pretrained weights are available as net_M434_adam_T600_DCNetBnoise-D1.params, saved via net.save_parameters(file_name). Verify a forward pass on demo_data.pkl (samples as columns, human gene symbols as rows, log2(x+1)-transformed) to confirm the 9078/434/21136 layer dimensions and that masked connectivity matches the relation matrix before saving as dcnet_model.py.

## Related tools

- **DCNet** (The masked deconvolution network being constructed; embeds 434 cell–marker-gene relationships in the ANN structure) — https://github.com/xindd/DCNet
- **mxnet** (Gluon framework used to code the DCNet model, apply the weight mask, and train with adam/ReLU/L2loss settings)
- **Python** (Implementation language; pickle-loading of index.pkl, relation_matrix.pkl and demo_data.pkl, and saving the model definition)

## Examples

```
import pickle, mxnet as mx; P = pickle.load(open('relation_matrix.pkl','rb')); net = build_dcnet(9078, 434, 21136, P); net.load_parameters('net_M434_adam_T600_DCNetBnoise-D1.params')
```

## Evaluation signals

- The constructed network exposes layers of exactly 9078 (input), 434 (hidden/latent), and 21136 (output) nodes
- Element-wise masked input-to-hidden connectivity equals the one-zero relation matrix P loaded from relation_matrix.pkl
- A forward pass on demo_data.pkl (log2(x+1)-transformed, samples as columns) completes without dimension errors
- Pretrained parameters net_M434_adam_T600_DCNetBnoise-D1.params load cleanly via net.load_parameters
- Latent-layer activation values are interpretable as 434 cell-type abundance estimates per sample

## Limitations

- Trained model weights or checkpoints availability is only documented via the net_M434_adam_T600_DCNetBnoise-D1.params file; no random seed or reproducibility controls for training are mentioned
- Which dropout setting (0.1/0.3/0.5) was used for the final model is not stated, and the fine-tuning procedure for the recommended dropout-0.1 DCNet on user data is unspecified
- Training was performed on CPU only with no hardware specification or runtime reported, and mxnet version numbers are not reported

## Evidence
