---
name: bulk-rnaseq-deconvolution-interpretation
description: Use when you have a bulk RNA-seq gene-by-sample expression matrix (rows
  = human gene symbols, columns = samples) and need cell-type composition estimates
  covering >400 cell types, e.g., to compare TME cell abundance across tumor subtypes.
when_to_use_negative:
- Input is single-cell RNA-seq data — DCNet deconvolves bulk RNA-seq; for scRNA-seq
  cell landscape validation the paper uses datasets like GSE81861 or GSE75688 rather
  than applying DCNet inference.
- You need a limited curated immune panel only — CIBERSORTx, EPIC, MCP-counter, quanTIseq,
  TIMER, or xCell target fewer cell types and may suffice; DCNet is for broad (>400
  cell type) landscapes.
- Marker genes of interest are absent from the input expression platform — 33 cell
  types were discarded because none of their marker genes were detected, so such cell
  types cannot be inferred.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: DCNet
  role: Interpretable deep learning (ANN) framework that embeds 434 cell types and
    their marker genes in the network to infer cell abundances from bulk RNA-seq
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Python framework in which the DCNet model is implemented and run
  repo: https://github.com/xindd/DCNet
- name: Python
  role: Implementation and downstream analysis language for DCNet and differential
    abundance testing
- name: CIBERSORTx
  role: Comparison method for cell abundance estimation benchmarked against DCNet
    with default parameters
- name: TIMER
  role: Comparison deconvolution method for evaluating cell abundance profiles
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:ed3f6c5ee4ac460227718682bfff60816efec385edbc31e145e28fedb9afbce4
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - CIBERSORTx
  - DCNet
  - Python
  - TIMER
  - mxnet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# bulk-rnaseq-deconvolution-interpretation

## Summary

Infers cell-type abundance landscapes from bulk RNA-seq samples using DCNet, an explainable neural network that embeds 434 cell types and 9078 marker genes, and interprets the resulting cell landscapes (e.g., differential abundance between LUAD and LUSC in TCGA).

## When to use

Use when you have a bulk RNA-seq gene-by-sample expression matrix (rows = human gene symbols, columns = samples) and need cell-type composition estimates covering >400 cell types, e.g., to compare TME cell abundance across tumor subtypes.

## When NOT to use

- Input is single-cell RNA-seq data — DCNet deconvolves bulk RNA-seq; for scRNA-seq cell landscape validation the paper uses datasets like GSE81861 or GSE75688 rather than applying DCNet inference.
- You need a limited curated immune panel only — CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, or xCell target fewer cell types and may suffice; DCNet is for broad (>400 cell type) landscapes.
- Marker genes of interest are absent from the input expression platform — 33 cell types were discarded because none of their marker genes were detected, so such cell types cannot be inferred.

## Inputs

- Bulk RNA-seq expression matrix (rows = human gene symbols, columns = samples; log2(x+1)-transformed, e.g., demo_data.pkl)
- DCNet trained parameter file (net_M434_adam_T600_DCNetBnoise-D1.params)
- Gene-cell relation matrix (relation_matrix.pkl, zero-one matrix of gene-cell correspondence)
- Gene index file (index.pkl, gene set for input and output neurons)
- TCGA LUAD/LUSC tumor subtype labels from GDC for differential abundance testing

## Outputs

- Cell abundance matrix (>400 cell types) per bulk RNA-seq sample
- Filtered differential cell table (cell type, fold change, q-value) e.g., differential_cells.csv for LUAD vs LUSC

## How to apply

Load the DCNet parameters (net_M434_adam_T600_DCNetBnoise-D1.params), the zero-one relation matrix mapping genes to cells, and the index.pkl file that stores the gene set for each input/output neuron; the model embeds 434 cell types and 9078 marker genes in the network. Prepare the input expression matrix with rows as human gene symbols and columns as samples, apply a log2(x+1) transformation as recommended, then run DCNet (trained with adam optimizer, relu activation, L2 loss, learning rate 1e-4, 600 iterations, 256 samples per batch, CPU) to infer cell abundances; a dropout of 0.1 model is recommended with optional fine-tuning on your sample dataset. For subtype contrast (e.g., TCGA LUAD vs LUSC), compute per-cell-type fold change between groups, apply statistical tests with multiple-testing correction, and filter at fold change >2 or <0.5 with q<0.01 while retaining directionality. Preprocessing involves deleting tissue-specific duplicate cells, deleting marker genes not detected in TCGA gene sets, discarding 33 cell types with no detected marker genes, and removing genes with zero expression in more than 1/3 of samples (21,136 genes retained).

## Related tools

- **DCNet** (Interpretable deep learning (ANN) framework that embeds 434 cell types and their marker genes in the network to infer cell abundances from bulk RNA-seq) — https://github.com/xindd/DCNet
- **mxnet** (Python framework in which the DCNet model is implemented and run) — https://github.com/xindd/DCNet
- **Python** (Implementation and downstream analysis language for DCNet and differential abundance testing)
- **CIBERSORTx** (Comparison method for cell abundance estimation benchmarked against DCNet with default parameters)
- **TIMER** (Comparison deconvolution method for evaluating cell abundance profiles)

## Examples

```
git clone https://github.com/xindd/DCNet.git && cd DCNet && pip install -r requirements.txt && python setup.py install
```

## Evaluation signals

- Output abundance matrix covers the 434 cell types indexed by index.pkl (433 retained after discarding 33 cell types lacking detected marker genes during training preprocessing)
- Validation against single-cell datasets (GSE81861 CRC, GSE75688 BC, GSE86146 FGC, GSE72056 melanoma) shows DCNet accurately recapitulates known cell landscapes with better robustness and stability than CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, and xCell
- Robustness check: input data randomly deleted with probability 0.1/0.3/0.5 (zero-filled) should not destabilize inferred abundances; dropout-0.1 model recommended
- Differential cells table satisfies fold change >2 or <0.5 with q<0.01, e.g., higher in LUSC: Neuroepithelial, Myofibroblast, Basal; higher in LUAD: Granulocyte, Alveolar, Leukocyte, Secretory, MDSC, Ciliated
- Expression matrix rows are human gene symbols with log2(x+1) transformation applied before inference

## Limitations

- No software version numbers reported for mxnet, R packages, or comparison tools; no random seed or reproducibility controls for model training
- Trained model weights/checkpoints are provided (net_M434_adam_T600_DCNetBnoise-D1.params), but the fine-tuning procedure for the recommended dropout-0.1 DCNet on user sample data is not specified
- Random gene dropout probabilities (0.1/0.3/0.5) were used for augmentation, but which setting was used for the final model is not stated; training was CPU-only with no hardware specification or runtime reported
- Comparison methods were run with default parameters only, no parameter tuning reported

## Evidence

- [readme] DCNet, an "explainable" artificial neural network model was created to infer cell content from bulk RNA-Seq samples. It embeds the relationship between 434 cells and 9078 marker genes: "DCNet, an "explainable" artificial neural network model was created to infer cell content from bulk RNA-Seq samples. It embeds the relationship between 434 cells and 9078 marker genes"
- [readme] Each column is a sample, and each row should be a human gene symbol. Please see demo_data as an example. A log2(x+1) transformation of the expression value is recommended.: "Each column is a sample, and each row should be a human gene symbol."
- [readme] its parameters are set as adam optimizer, relu activation function, L2loss loss function, learning rate 1e-4, number of iterations 600, 256 samples per batch, cpu training respectively.: "its parameters are set as adam optimizer, relu activation function, L2loss loss function, learning rate 1e-4, number of iterations 600, 256 samples per batch, cpu training respectively."
- [discussion] then 33 cell types were discarded because none of marker-genes were detected: "then 33 cell types were discarded because none of marker-genes were detected"
- [discussion] the genes whose expression level is 0 in more than 1/3 samples were deleted, 21,136 genes were retained: "the genes whose expression level is 0 in more than 1/3 samples were deleted, 21,136 genes were retained"
- [discussion] we recommend choosing DCNet with a dropout of 0.1, and fine-tuning the DCNet model in the sample data set: "we recommend choosing DCNet with a dropout of 0.1, and fine-tuning the DCNet model in the sample data set"
- [discussion] DCNet model were coded using the mxnet framework in python: "DCNet model were coded using the mxnet framework in python"
- [abstract] can infer the cell landscape with more than 400 cell types based on bulk RNA-seq data: "can infer the cell landscape with more than 400 cell types based on bulk RNA-seq data"
