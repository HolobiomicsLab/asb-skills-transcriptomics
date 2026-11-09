---
name: tumor-microenvironment-landscape-inference
description: Use when you have a bulk RNA-seq expression matrix (rows = human gene
  symbols, columns = samples, e.g. TCGA) and need cell-type abundance estimates across
  ~434 TME cell types to cluster patients into TME subtypes.
when_to_use_negative:
- Input is single-cell RNA-seq data — DCNet is designed to deconvolve bulk RNA-seq
  into cell landscapes, not to quantify cell types directly from scRNA-seq
- You need only a small panel of known cell types with a well-validated signature
  matrix — tools like CIBERSORTx, EPIC, quanTIseq, TIMER, or xCell (run with default
  parameters in the paper's comparison) may be more appropriate
- Marker genes for your cell types of interest are not detected in the expression
  matrix — the article discarded 33 cell types because none of their marker genes
  were detected
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: DCNet
  role: Interpretable ANN that embeds relationships between 434 cell types and their
    marker genes to infer cell-type abundances from bulk RNA-seq
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Python deep-learning framework in which the DCNet model is implemented and
    run
- name: ConsensusClusterPlus
  role: R package used to consensus-cluster the inferred cell abundance landscape
    (manhattan distance, PAM, 1000 iterations) into IPS/NIPS subtypes
- name: CIBERSORTx
  role: Baseline deconvolution method used to compare DCNet-inferred cell abundance
    profiles
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:1c5f2c8d6b3f11fd4d52b5213a20ad3d4bc93f3298aa5272f15d71ca2c327c7c
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - CIBERSORTx
  - ConsensusClusterPlus
  - DCNet
  - Python
  - mxnet
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# tumor-microenvironment-landscape-inference

## Summary

Infers a tumor microenvironment cell landscape (434 cell types) from bulk RNA-seq using DCNet, an interpretable mxnet ANN whose input/output neurons are wired to marker genes via a genes-cells zero-one relation matrix, and supports downstream TME subtyping (IPS/NIPS) with survival separation.

## When to use

Use when you have a bulk RNA-seq expression matrix (rows = human gene symbols, columns = samples, e.g. TCGA) and need cell-type abundance estimates across ~434 TME cell types to cluster patients into TME subtypes.

## When NOT to use

- Input is single-cell RNA-seq data — DCNet is designed to deconvolve bulk RNA-seq into cell landscapes, not to quantify cell types directly from scRNA-seq
- You need only a small panel of known cell types with a well-validated signature matrix — tools like CIBERSORTx, EPIC, quanTIseq, TIMER, or xCell (run with default parameters in the paper's comparison) may be more appropriate
- Marker genes for your cell types of interest are not detected in the expression matrix — the article discarded 33 cell types because none of their marker genes were detected

## Inputs

- Bulk RNA-seq gene expression matrix (rows = human gene symbols, columns = samples), log2(x+1) transformed
- DCNet model parameters file net_M434_adam_T600_DCNetBnoise-D1.params
- relation_matrix.pkl — zero-one correspondence matrix between genes and cells
- index.pkl — gene set corresponding to input and output neurons
- TCGA clinical survival data (for subtype/survival merging)
- CellMarker marker gene data (13,605 marker genes across 467 cell types in 158 human tissues)

## Outputs

- Cell-type abundance landscape matrix (samples x 434 cell types)
- IPS/NIPS subtype assignments from ConsensusClusterPlus clustering
- Figure 4a-style abundance landscape plot
- Kaplan-Meier overall-survival curves with log-rank test p-value

## How to apply

Prepare the expression matrix with each column a sample and each row a human gene symbol, applying the recommended log2(x+1) transformation. Load the DCNet parameters (net_M434_adam_T600_DCNetBnoise-D1.params), the zero-one relation_matrix.pkl mapping genes to cells, and index.pkl that stores the gene set corresponding to each input/output neuron from the xindd/DCNet-Use repository. Run DCNet (mxnet, Python 3; trained with adam optimizer, relu activation, L2loss, learning rate 1e-4, 600 iterations, 256 samples per batch, CPU) to infer cell abundances; the article recommends the dropout-0.1 model with optional fine-tuning on your own sample dataset. For TCGA-scale subtyping, cluster the 10,176-sample abundance landscape with the R package ConsensusClusterPlus using manhattan distance, PAM clustering, and 1000 iterations to recover IPS and NIPS subtypes, then merge labels with clinical survival data for Kaplan-Meier/log-rank testing. Rationale: embedding marker-gene-to-cell relationships directly in the network architecture makes the 'black box' prediction traceable and extends deconvolution beyond the limited cell types of CIBERSORTx/EPIC/MCP-counter/quanTIseq/TIMER/xCell.

## Related tools

- **DCNet** (Interpretable ANN that embeds relationships between 434 cell types and their marker genes to infer cell-type abundances from bulk RNA-seq) — https://github.com/xindd/DCNet
- **mxnet** (Python deep-learning framework in which the DCNet model is implemented and run)
- **ConsensusClusterPlus** (R package used to consensus-cluster the inferred cell abundance landscape (manhattan distance, PAM, 1000 iterations) into IPS/NIPS subtypes)
- **CIBERSORTx** (Baseline deconvolution method used to compare DCNet-inferred cell abundance profiles)

## Examples

```
git clone https://github.com/xindd/DCNet.git; cd DCNet; pip install -r requirements.txt; python setup.py install
```

## Evaluation signals

- DCNet-inferred abundances recapitulate the cell landscape of single-cell RNA-seq datasets (e.g. GSE81861 CRC, GSE75688 BC, GSE72056 melanoma) with better robustness and stability than CIBERSORTx/EPIC/MCP-counter/quanTIseq/TIMER/xCell
- Consensus clustering of the 10,176-sample TCGA landscape recovers exactly two robust subtypes (IPS and NIPS)
- Kaplan-Meier overall-survival curves of the two subtypes separate with log-rank p < .0001, IPS patients surviving better
- Input matrix passes format checks: each row a human gene symbol, each column a sample, values log2(x+1) transformed
- Robustness check: DCNet tolerates randomly deleting input data with probability 0.1, 0.3, 0.5 (0 fill) and still recovers the landscape

## Limitations

- No software version numbers reported for mxnet, R packages, or comparison tools; no random seed or reproducibility controls for model training
- Trained model weights are provided but a fine-tuning procedure for the recommended dropout-0.1 DCNet on user sample data is not specified
- Training was CPU only with no hardware specification or runtime reported; comparison methods were all run with default parameters, no tuning
- Marker genes not detected in the TCGA gene sets and genes with zero expression in more than 1/3 of samples are removed, so some cell types (33 discarded) cannot be inferred

## Evidence
