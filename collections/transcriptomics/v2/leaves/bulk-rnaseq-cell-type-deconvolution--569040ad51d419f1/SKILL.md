---
name: bulk-rnaseq-cell-type-deconvolution
description: Use when you have a bulk RNA-seq or pseudo-bulk expression matrix (rows
  = human gene symbols, columns = samples) and need to estimate abundance of hundreds
  of cell types for downstream TME clustering or survival analysis.
when_to_use_negative:
- Input is already single-cell data where cell identities are known (DCNet is designed
  to infer cell content from bulk expression, not to re-classify single cells)
- Expression matrix uses non-human gene symbols or cannot be matched to the index.pkl
  gene sets (33 cell types were discarded precisely because no marker genes were detected
  after ID matching)
- A reproducible, seed-controlled pipeline with versioned software is required — the
  paper reports no random seeds, no mxnet/R package versions, and no hardware/runtime
  specifications
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: DCNet
  role: Deconvolution network that embeds cell-marker-gene relationships in a visible
    ANN and uses the hidden layer to output cell-type abundance from bulk RNA-seq
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Python deep-learning framework in which the DCNet model is coded, trained
    (adam, relu, L2 loss, lr 1e-4, 600 iterations) and applied
- name: DCNet-Use
  role: Companion documentation repository hosting index.pkl, relation_matrix.pkl,
    pretrained .params file, demo_data.pkl, and the demo.ipynb walkthrough
  repo: https://github.com/xindd/DCNet-Use/blob/main/demo.ipynb
- name: CIBERSORTx
  role: Baseline deconvolution method run with default parameters for per-cell-type
    correlation comparison against DCNet
- name: EPIC
  role: Baseline deconvolution method run with default parameters for per-cell-type
    correlation comparison against DCNet
- name: MCP-counter
  role: Baseline deconvolution method run with default parameters for per-cell-type
    correlation comparison against DCNet
- name: quanTIseq
  role: Baseline deconvolution method run with default parameters for per-cell-type
    correlation comparison against DCNet
- name: TIMER
  role: Baseline deconvolution method run with default parameters for per-cell-type
    correlation comparison against DCNet
- name: xCell
  role: Baseline deconvolution method run with default parameters for per-cell-type
    correlation comparison against DCNet
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:e1d6a41823c6dff46c07a48a2a706c6ad9d0d10474519bfdd6bc7983d9e3ceb2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - CIBERSORTx
  - ConsensusClusterPlus
  - DCNet
  - DCNet-Use
  - EPIC
  - MCP-counter
  - Python
  - TIMER
  - mxnet
  - quanTIseq
  - xCell
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# bulk-rnaseq-cell-type-deconvolution

## Summary

Infers per-cell-type abundance landscapes from bulk RNA-seq samples using DCNet, an interpretable mxnet-implemented ANN that wires 9078 marker genes to 434 cell types, validated against CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, and xCell.

## When to use

Use when you have a bulk RNA-seq or pseudo-bulk expression matrix (rows = human gene symbols, columns = samples) and need to estimate abundance of hundreds of cell types for downstream TME clustering or survival analysis.

## When NOT to use

- Input is already single-cell data where cell identities are known (DCNet is designed to infer cell content from bulk expression, not to re-classify single cells)
- Expression matrix uses non-human gene symbols or cannot be matched to the index.pkl gene sets (33 cell types were discarded precisely because no marker genes were detected after ID matching)
- A reproducible, seed-controlled pipeline with versioned software is required — the paper reports no random seeds, no mxnet/R package versions, and no hardware/runtime specifications

## Inputs

- Bulk RNA-seq expression matrix with rows as human gene symbols and columns as samples (see demo_data.pkl format)
- index.pkl — gene set mapping input and output neurons to specific genes
- relation_matrix.pkl — zero-one correspondence matrix between genes and cells
- net_M434_adam_T600_DCNetBnoise-D1.params — pretrained DCNet weights (optional, for inference without retraining)
- Single-cell GEO datasets (e.g., GSE81861 CRC, GSE72056 melanoma) aggregated into pseudo-bulk samples with known cell abundances for benchmarking

## Outputs

- Per-sample cell-type abundance matrix (hidden-layer activations for 434 cell types)
- Per-cell-type Pearson correlations of predicted vs. true abundance for method comparison (e.g., Figure 3a heatmap)
- Cell landscape suitable for downstream consensus clustering (IPS/NIPS subtypes via ConsensusClusterPlus)

## How to apply

Prepare the input so each column is a sample and each row a human gene symbol, then apply a log2(x+1) transformation as recommended for DCNet input. Run DCNet using index.pkl (the gene sets wiring input/output neurons) and the relation_matrix.pkl zero-one gene-cell matrix, loading pretrained parameters net_M434_adam_T600_DCNetBnoise-D1.params or training with adam optimizer, relu activation, L2 loss, learning rate 1e-4, 600 iterations, batch size 256 on CPU. The hidden layer activations represent inferred cell-type content (434 cell types after preprocessing removed tissue-duplicated cells, 33 cell types lacking detected markers, and genes zero in >1/3 of TCGA samples). For benchmarking, restrict predictions to cell types present in the dataset (e.g., 7 CRC types in GSE81861, 6 melanoma types in GSE72056) and compute per-cell-type Pearson correlations with known abundance against Marker_sum and the six baseline deconvolution tools run at default parameters. The paper recommends fine-tuning the dropout-0.1 DCNet variant on your sample dataset before use.

## Related tools

- **DCNet** (Deconvolution network that embeds cell-marker-gene relationships in a visible ANN and uses the hidden layer to output cell-type abundance from bulk RNA-seq) — https://github.com/xindd/DCNet
- **mxnet** (Python deep-learning framework in which the DCNet model is coded, trained (adam, relu, L2 loss, lr 1e-4, 600 iterations) and applied)
- **DCNet-Use** (Companion documentation repository hosting index.pkl, relation_matrix.pkl, pretrained .params file, demo_data.pkl, and the demo.ipynb walkthrough) — https://github.com/xindd/DCNet-Use/blob/main/demo.ipynb
- **CIBERSORTx** (Baseline deconvolution method run with default parameters for per-cell-type correlation comparison against DCNet)
- **EPIC** (Baseline deconvolution method run with default parameters for per-cell-type correlation comparison against DCNet)
- **MCP-counter** (Baseline deconvolution method run with default parameters for per-cell-type correlation comparison against DCNet)
- **quanTIseq** (Baseline deconvolution method run with default parameters for per-cell-type correlation comparison against DCNet)
- **TIMER** (Baseline deconvolution method run with default parameters for per-cell-type correlation comparison against DCNet)
- **xCell** (Baseline deconvolution method run with default parameters for per-cell-type correlation comparison against DCNet)

## Examples

```
git clone https://github.com/xindd/DCNet.git && cd DCNet && pip install -r requirements.txt && python setup.py install  # then follow DCNet-Use demo.ipynb: load index.pkl, relation_matrix.pkl, net_M434_adam_T600_DCNetBnoise-D1.params
```

## Evaluation signals

- All per-cell-type Pearson correlations between DCNet predictions and true abundance exceed 0.7 on GSE81861 (CRC) and GSE72056 (melanoma) pseudo-bulk samples
- Reproduced benchmark matches Figure 3a anchors: DCNet T/B cell correlations of 0.81/0.95 in melanoma, xCell macrophage -0.09 in CRC, Marker_sum B cell -0.13 in CRC
- Input matrix verified as columns = samples, rows = human gene symbols, with log2(x+1) transformation applied before inference
- Inferred landscape covers ~434 cell types after the paper's filtering (tissue-duplicate removal, 33 marker-less cell types dropped, genes zero in >1/3 TCGA samples removed, 21,136 genes retained)
- DCNet correlations show stable behavior across cell types while baseline methods (e.g., xCell, Marker_sum) exhibit unstable/negative correlations on specific cell types

## Limitations

- No random seed or reproducibility controls reported for model training; all comparison methods run with default parameters and no tuning
- Random gene dropout probabilities (0.1/0.3/0.5) were used for augmentation but the setting used for the final model is not stated; the recommended dropout-0.1 fine-tuning procedure on user data is not specified
- Training was performed on CPU only with no hardware specification or runtime reported; training parameters (adam, lr 1e-4, 600 iterations, batch 256) are given but version numbers for mxnet and comparison tools are not
- As a black-box neural network, per-sample cell abundance depends on marker-gene selection from CellMarker (13,605 marker genes across 467 cell types); dependent marker-gene selection is an acknowledged source of deviation in such methods

## Evidence

- [readme] Deconvolution network (DCNet), an "explainable" artificial neural network model was created to infer cell content from bulk RNA-Seq samples. It embeds the relationship between 434 cells and 9078 marker genes in a neural network: "Deconvolution network (DCNet), an "explainable" artificial neural network model was created to infer cell content from bulk RNA-Seq samples. It embeds the relationship between 434 cells and 9078"
- [methods] Each column is a sample, and each row should be a human gene symbol.: "Each column is a sample, and each row should be a human gene symbol."
- [methods] A log2(x+1) transformation of the expression value is recommended.: "A log2(x+1) transformation of the expression value is recommended."
- [discussion] its parameters are set as adam optimizer, relu activation function, L2loss loss function, learning rate 1e-4, number of iterations 600, 256 samples per batch, cpu training: "its parameters are set as adam optimizer, relu activation function, L2loss loss function, learning rate 1e-4, number of iterations 600, 256 samples per batch, cpu training"
- [abstract] can infer the cell landscape with more than 400 cell types based on bulk RNA-seq data: "can infer the cell landscape with more than 400 cell types based on bulk RNA-seq data"
