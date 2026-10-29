---
name: cell-type-marker-gene-matrix-handling
description: Use when reconstructing or fine-tuning the DCNet network and you need
  to wire 9078 marker genes to 434 cell-type hidden neurons via the zero-one relation
  matrix and align input/output neuron ordering with index.pkl.
when_to_use_negative:
- Input gene set does not match the 9078 marker genes in index.pkl (e.g. non-human
  gene symbols or unaligned identifiers), since the mask connectivity would be meaningless
- Expression values are not log2(x+1)-transformed — the paper recommends this transformation
  before feeding the matrix
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_0203
tools:
- name: DCNet
  role: The masked neural network whose gene–cell relation matrix and neuron index
    files are handled by this skill
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Gluon framework in which the three-layer masked network (9078/434/21136) is
    defined and the relation matrix is applied element-wise to input-hidden weights
- name: Python
  role: Language used to unpickle relation_matrix.pkl and index.pkl and build the
    masked network
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:0f524c29177168e4e022e46904d49097f73a07c40bbde45b3987d5beb61e2bc5
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
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cell-type-marker-gene-matrix-handling

## Summary

Load and apply DCNet's zero-one gene–cell relation matrix (relation_matrix.pkl) and gene-to-neuron index mappings (index.pkl) so that each of the 434 hidden-layer cell-type neurons connects only to its marker-gene input neurons and each of the 21136 output neurons maps to a specific gene.

## When to use

Use when reconstructing or fine-tuning the DCNet network and you need to wire 9078 marker genes to 434 cell-type hidden neurons via the zero-one relation matrix and align input/output neuron ordering with index.pkl.

## When NOT to use

- Input gene set does not match the 9078 marker genes in index.pkl (e.g. non-human gene symbols or unaligned identifiers), since the mask connectivity would be meaningless
- Expression values are not log2(x+1)-transformed — the paper recommends this transformation before feeding the matrix

## Inputs

- relation_matrix.pkl / rix.pkl — zero-one matrix of gene–cell correspondence
- index.pkl — set of genes corresponding to input and output neurons
- Gene expression matrix (rows = human gene symbols, columns = samples), log2(x+1)-transformed

## Outputs

- Masked mxnet network definition with 9078/434/21136 node layers and input-to-hidden connectivity matching the relation matrix
- Ordered input/output gene index mappings for the network
- Latent-layer activation vector of 434 cell-type abundances per sample

## How to apply

Load the zero-one gene–cell correspondence matrix (relation_matrix.pkl, referred to as rix.pkl in the source card) and the gene-to-neuron index mappings (index.pkl) with Python pickle. Element-wise mask the input-to-hidden weight connections of the mxnet/gluon three-layer network (9078 input, 434 hidden, 21136 output nodes) with the relation matrix P so each hidden neuron (cell type) connects only to its marker-gene input neurons. Use index.pkl to order input rows (human gene symbols, log2(x+1)-transformed, columns = samples) and output neurons so cell-type abundances are read from the 434-neuron latent layer. Verify the masked connectivity matches the relation matrix before training (adam, lr 1e-4, 600 iterations, batch 256, ReLU, L2 loss).

## Related tools

- **DCNet** (The masked neural network whose gene–cell relation matrix and neuron index files are handled by this skill) — https://github.com/xindd/DCNet
- **mxnet** (Gluon framework in which the three-layer masked network (9078/434/21136) is defined and the relation matrix is applied element-wise to input-hidden weights)
- **Python** (Language used to unpickle relation_matrix.pkl and index.pkl and build the masked network)

## Examples

```
import pickle; P = pickle.load(open('relation_matrix.pkl','rb')); idx = pickle.load(open('index.pkl','rb')); net = build_masked_dcnet(9078, 434, 21136, mask=P)
```

## Evaluation signals

- Constructed network exposes layers of exactly 9078 input, 434 hidden, and 21136 output nodes
- Masked input-to-hidden connectivity matches the zero-one relation matrix loaded from relation_matrix.pkl element-wise
- Forward pass on the demo expression matrix (samples as columns, human gene symbols as rows, log2(x+1)-transformed) runs without dimension errors
- Latent layer produces 434 activation values per sample, interpretable as cell-type abundances
- Output neuron ordering is consistent with the gene-to-neuron mappings in index.pkl

## Limitations

- Marker genes not detected in TCGA gene sets and 33 cell types with no detected marker genes were discarded during matrix construction, so the relation matrix reflects 434 of the original 467 CellMarker cell types
- No software version numbers are reported for mxnet or Python, which may affect pickle compatibility
- Trained model weights are not guaranteed in the code repo alone; the params file (net_M434_adam_T600_DCNetBnoise-D1.params) must be fetched from DCNet-Use for inference without retraining

## Evidence

- [full_text] latent layer = cell abundance: [evidence span withheld — anchor status: not_found]
