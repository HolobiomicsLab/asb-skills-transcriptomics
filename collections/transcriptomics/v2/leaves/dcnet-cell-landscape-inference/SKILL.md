---
name: dcnet-cell-landscape-inference
description: Use when you need to deconvolve a bulk RNA-seq expression matrix into
  abundances of >400 cell types (including non-immune types like fetal germ cells)
  and have samples as columns and human gene symbols as rows.
when_to_use_negative:
- Input is single-cell expression data (GSE86146 single-cell profiles must first be
  aggregated into bulk matrices per cell type before DCNet can be applied)
- Expression matrix rows are not human gene symbols or the matrix has not been sample-column
  formatted — DCNet's neurons are fixed to specific genes via index.pkl
- You need cell-type-specific inference beyond the 434 embedded Cell Ontology types
  or need de novo marker gene selection rather than the fixed relation matrix
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DCNet
  role: Interpretable ANN that embeds 434-cell-type/marker-gene relationships and
    infers cell content from bulk RNA-seq; runs on the trained .params file
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Python deep learning framework in which the DCNet model is coded and executed
- name: Python
  role: Runtime environment (Python 3, with pandas/numpy dependencies) for loading
    pkl assets and running DCNet inference
- name: ontoProc
  role: R package for working with biomedical/cell ontologies, relevant to navigating
    the Cell Ontology hierarchy used for cell-type leaf nodes
  repo: https://github.com/vjcitn/ontoProc
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:09c9fad1d483d89f55a086d33d449c90a8ecfcf640a4deb50e65908f6876ae3f
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  - mxnet
  - ontoProc
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# dcnet-cell-landscape-inference

## Summary

DCNet is an interpretable deep learning framework (mxnet/Python) that embeds the marker-gene relationships of 434 cell types and 9078 marker genes into an ANN structure to infer cell-type abundances from bulk RNA-seq samples. This skill covers running the pre-trained DCNet on a log2(x+1)-transformed expression matrix (rows = human gene symbols, columns = samples) and validating predicted abundances against single-cell-derived ground truth.

## When to use

Use when you need to deconvolve a bulk RNA-seq expression matrix into abundances of >400 cell types (including non-immune types like fetal germ cells) and have samples as columns and human gene symbols as rows.

## When NOT to use

- Input is single-cell expression data (GSE86146 single-cell profiles must first be aggregated into bulk matrices per cell type before DCNet can be applied)
- Expression matrix rows are not human gene symbols or the matrix has not been sample-column formatted — DCNet's neurons are fixed to specific genes via index.pkl
- You need cell-type-specific inference beyond the 434 embedded Cell Ontology types or need de novo marker gene selection rather than the fixed relation matrix

## Inputs

- Bulk RNA-seq gene expression matrix (rows = human gene symbols, columns = samples), log2(x+1) transformed, e.g. demo_data.pkl format
- index.pkl — gene-to-input/output-neuron mapping from xindd/DCNet-Use
- relation_matrix.pkl — zero-one gene-cell correspondence matrix
- net_M434_adam_T600_DCNetBnoise-D1.params — trained DCNet parameters (weights, biases)

## Outputs

- Predicted abundances for 434 cell types embedded in the DCNet network per bulk sample
- Correlation table of predicted vs. true abundances (e.g. gse86146_correlations.csv) with Pearson r values and p-values

## How to apply

Prepare an expression matrix with columns as samples and rows as human gene symbols, apply the recommended log2(x+1) transformation, and load the DCNet assets from xindd/DCNet-Use: index.pkl (gene assignments to input/output neurons), relation_matrix.pkl (zero-one gene-cell correspondence), and net_M434_adam_T600_DCNetBnoise-D1.params (trained weights). Run the workflow in the demo.ipynb notebook using the mxnet-based DCNet, which was trained with the adam optimizer, relu activation, L2 loss, learning rate 1e-4, 600 iterations, and 256 samples per batch on CPU. Extract predicted abundances for the cell types of interest from the 434-type Cell Ontology-embedded output. Validate by correlating predictions against true single-cell-derived abundances: the paper reports r = 0.87 for fetal germ cells and r = 0.79 for somatic cells on GSE86146 (p < .01). For best results the authors recommend the dropout-0.1 DCNet variant with fine-tuning on the user's sample dataset.

## Related tools

- **DCNet** (Interpretable ANN that embeds 434-cell-type/marker-gene relationships and infers cell content from bulk RNA-seq; runs on the trained .params file) — https://github.com/xindd/DCNet
- **mxnet** (Python deep learning framework in which the DCNet model is coded and executed)
- **Python** (Runtime environment (Python 3, with pandas/numpy dependencies) for loading pkl assets and running DCNet inference)
- **ontoProc** (R package for working with biomedical/cell ontologies, relevant to navigating the Cell Ontology hierarchy used for cell-type leaf nodes) — https://github.com/vjcitn/ontoProc

## Examples

```
expr_log = np.log2(expr + 1)  # log2(x+1) recommended; then run DCNet-Use demo.ipynb with index.pkl, relation_matrix.pkl and net_M434_adam_T600_DCNetBnoise-D1.params
```

## Evaluation signals

- Predicted FGC abundance correlates with true aggregated abundance at approximately r = 0.87 and somatic cells at approximately r = 0.79, both p < 0.01 (matching Figure 3c,d on GSE86146)
- Expression matrix was log2(x+1) transformed before inference and each row maps to a valid human gene symbol present in index.pkl
- Output contains abundance estimates for the full 434-cell-type set embedded in the network, extractable for Cell Ontology leaf-node types
- Run uses the published trained parameters file (net_M434_adam_T600_DCNetBnoise-D1.params) rather than randomly initialized weights
- Results table (e.g. gse86146_correlations.csv) reproduces the expected correlation ordering FGC > somatic

## Limitations

- No random seed or reproducibility controls for model training are documented; comparison methods were run with default parameters only
- Fine-tuning procedure for the recommended dropout-0.1 DCNet on user sample data is not specified, and which dropout setting (0.1/0.3/0.5) was used for the final model is not stated
- Trained model weights are distributed but no software version numbers are reported for mxnet or dependency packages; training was CPU-only with no runtime reported
- 33 cell types were discarded during training-set construction for lacking detected marker genes, so those types are not inferrable

## Evidence

- [methods] Each column is a sample, and each row should be a human gene symbol. A log2(x+1) transformation of the expression value is recommended.: [evidence span withheld — anchor status: not_found]
- [results] DCNet's predicted abundance shows high correlation with true abundance: 0.87 for FGC cells and 0.79 for somatic cells (p < .01): [evidence span withheld — anchor status: not_found]
