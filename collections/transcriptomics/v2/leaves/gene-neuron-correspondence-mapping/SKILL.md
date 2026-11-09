---
name: gene-neuron-correspondence-mapping
description: 'Use when reconstructing or retraining DCNet: you need to wire the 9078x434
  zero-one gene-cell matrix into the network''s input-to-hidden weights and align
  input/output neuron ordering with the gene index.'
when_to_use_negative:
- Input expression rows are not human gene symbols or cannot be aligned to the index.pkl
  gene ordering (e.g. probe IDs, Ensembl IDs without symbol mapping)
- The analysis targets a different deconvolution method (CIBERSORTx, EPIC, MCP-counter,
  quanTIseq, TIMER, xCell) rather than the DCNet masked-autoencoder architecture
- Only pre-trained DCNet weights are needed for scoring with no architecture reconstruction
  or re-masking required
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3308
tools:
- name: DCNet
  role: Masked autoencoder architecture whose 434 hidden neurons are mapped to cell
    types via the relation matrix
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Gluon framework used to define the masked 9078/434/21136 network and apply
    ReLU/adam/L2loss settings
- name: Python
  role: Language for pickle-loading rix.pkl/index.pkl and building the network
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:9231708b1e6a1dad45e433d13ba3820ef62853610f450950149abd072e751154
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  - mxnet
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# gene-neuron-correspondence-mapping

## Summary

Map the 9078 marker genes and 434 cell types of DCNet onto input, hidden, and output neurons using the pickled gene-cell relation matrix (rix.pkl/relation_matrix.pkl) and gene-to-neuron index (index.pkl), so each hidden neuron corresponds to one cell type whose incoming weights are masked to its marker genes only.

## When to use

Use when reconstructing or retraining DCNet: you need to wire the 9078x434 zero-one gene-cell matrix into the network's input-to-hidden weights and align input/output neuron ordering with the gene index.

## When NOT to use

- Input expression rows are not human gene symbols or cannot be aligned to the index.pkl gene ordering (e.g. probe IDs, Ensembl IDs without symbol mapping)
- The analysis targets a different deconvolution method (CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, xCell) rather than the DCNet masked-autoencoder architecture
- Only pre-trained DCNet weights are needed for scoring with no architecture reconstruction or re-masking required

## Inputs

- rix.pkl / relation_matrix.pkl: zero-one matrix of correspondence between 9078 genes and 434 cells
- index.pkl: pickle storing the set of genes corresponding to input and output neurons
- demo_data.pkl or user expression matrix: samples as columns, human gene symbols as rows, log2(x+1)-transformed

## Outputs

- Masked mxnet gluon network (9078/434/21136 layers) with input-to-hidden connectivity matching the relation matrix
- Latent-layer activation values representing the abundance of 434 cell types per sample
- Saved model definition (e.g. dcnet_model.py) and .params checkpoint via net.save_parameters()

## How to apply

Load the one-zero gene-cell correspondence matrix (rix.pkl / relation_matrix.pkl) and the gene-to-neuron mappings (index.pkl) via Python pickle; index.pkl stores the set of genes corresponding to input and output neurons. Define an mxnet gluon network with input dimension 9078, hidden (latent) dimension 434, and output dimension 21136, then element-wise mask the input-to-hidden weight connections with the relation matrix P so each hidden neuron (cell type) connects only to its marker-gene input neurons. Apply ReLU activations on hidden and output layers, train with adam, L2loss, learning rate 1e-4, 600 iterations, 256 samples per batch on CPU. Input expression matrices must have samples as columns and human gene symbols as rows, log2(x+1)-transformed, and rows must be ordered to match the index.pkl gene-to-neuron mapping before the forward pass.

## Related tools

- **DCNet** (Masked autoencoder architecture whose 434 hidden neurons are mapped to cell types via the relation matrix) — https://github.com/xindd/DCNet
- **mxnet** (Gluon framework used to define the masked 9078/434/21136 network and apply ReLU/adam/L2loss settings)
- **Python** (Language for pickle-loading rix.pkl/index.pkl and building the network)

## Examples

```
import pickle; rix = pickle.load(open('relation_matrix.pkl','rb')); idx = pickle.load(open('index.pkl','rb')); net = build_masked_dcnet(rix, input_dim=9078, hidden_dim=434, output_dim=21136)
```

## Evaluation signals

- Forward pass on the demo expression matrix runs without error and exposes layers of 9078/434/21136 nodes
- Element-wise check: input-to-hidden weight mask matches the zero-one relation matrix P loaded from rix.pkl row-for-row
- Hidden-layer neuron count equals 434, matching the 434 cell types after 33 cell types with no detected marker genes were discarded
- Output dimension equals 21136 genes retained after removing genes zero in more than 1/3 of TCGA samples
- Latent-layer activations are non-negative (ReLU) and interpretable as cell-type abundances for log2(x+1)-transformed input

## Limitations

- Trained model weights or checkpoints are not stated as available in the paper; only source code is deposited (params exist in DCNet-Use but fine-tuning procedure for dropout-0.1 DCNet on user data is not specified)
- No software version numbers, random seed, or reproducibility controls are reported for the mxnet training
- Tissue-specific duplicate cell types were collapsed to a single representative and 33 cell types lacked detected marker genes, so the 434-neuron mapping is not a complete CellMarker catalog (467 cell types)

## Evidence

- [results] The encode and latent layers of the DCNet network contain 434 neurons, and the activation value of the latent layer represents the abundance of cells.: [evidence span withheld — anchor status: not_found]
