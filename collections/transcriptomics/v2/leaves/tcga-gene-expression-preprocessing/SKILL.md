---
name: tcga-gene-expression-preprocessing
description: 'Use when a bulk RNA-seq expression matrix (e.g., TCGA LUAD/LUSC from
  GDC) must be prepared as DCNet input: rows as human gene symbols, columns as samples,
  log2(x+1) transformed, with zero-expression and marker-gene filters applied.'
when_to_use_negative:
- Input is already a cell-type abundance/feature table — deconvolution preprocessing
  is for raw bulk expression matrices
- Single-cell RNA-seq data that does not require bulk deconvolution input formatting
  or the paper's TCGA-oriented gene filters
- Analyses where retaining genes with zero expression in many samples is required
  (e.g., splice or mutation-focused studies), since the 1/3-zero filter removes them
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
tools:
- name: DCNet
  role: Downstream consumer of the preprocessed expression matrix; the preprocessing
    step formats data to its required orientation and gene-symbol conventions
  repo: https://github.com/xindd/DCNet
- name: Python
  role: Environment in which the expression matrix loading, log2(x+1) transformation,
    and gene filtering are performed (DCNet is coded with mxnet in Python)
- name: mxnet
  role: Framework dependency for the DCNet pipeline that consumes the preprocessed
    matrix; preprocessing must produce arrays compatible with mxnet loading
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:725abdb380f2ab47e649f800051dbd215d9b0395ac9a7f0b05cf2b3bf5093919
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  - mxnet
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tcga-gene-expression-preprocessing

## Summary

Preprocesses TCGA RNA-seq expression matrices for DCNet cell-landscape deconvolution: orienting the matrix as rows of human gene symbols and columns of samples, applying the log2(x+1) transformation, and filtering genes and CellMarker-derived marker genes so only detectable, informative features feed the neural network.

## When to use

Use when a bulk RNA-seq expression matrix (e.g., TCGA LUAD/LUSC from GDC) must be prepared as DCNet input: rows as human gene symbols, columns as samples, log2(x+1) transformed, with zero-expression and marker-gene filters applied.

## When NOT to use

- Input is already a cell-type abundance/feature table — deconvolution preprocessing is for raw bulk expression matrices
- Single-cell RNA-seq data that does not require bulk deconvolution input formatting or the paper's TCGA-oriented gene filters
- Analyses where retaining genes with zero expression in many samples is required (e.g., splice or mutation-focused studies), since the 1/3-zero filter removes them

## Inputs

- TCGA (or other bulk) RNA-seq gene expression matrix with rows as human gene symbols and columns as samples, downloaded from GDC
- CellMarker database marker gene list (13,605 marker genes across 467 cell types in 158 human tissues)
- index.pkl — neuron-to-gene mapping for DCNet input/output neurons
- relation_matrix.pkl / rix.pkl — zero-one gene-cell correspondence matrix

## Outputs

- log2(x+1)-transformed, gene-filtered expression matrix (21,136 retained genes in the paper) oriented rows=genes, columns=samples
- Cleaned cell/marker-gene correspondence (duplicates removed, 33 cell types with no detected marker genes discarded)
- DCNet-ready input matrix consumable by the pretrained net_M434 params model

## How to apply

Download TCGA RNA-seq profiles from the GDC portal, then build the expression matrix so each column is a sample and each row is a human gene symbol. Apply the recommended log2(x+1) transformation to expression values. Filter genes by deleting those whose expression level is 0 in more than 1/3 of samples (the paper retained 21,136 genes this way), and delete genes with zero expression in more than 1/3 of TCGA samples before feeding DCNet. In parallel, preprocess CellMarker marker gene data (13,605 marker genes across 467 cell types in 158 tissues): remove tissue-specific duplicate cells keeping one representative, delete marker genes not detected in the TCGA gene sets, and discard the 33 cell types left with no detected marker genes. The filtered expression matrix and the gene-cell relation matrix (rix.pkl / relation_matrix.pkl) plus neuron index (index.pkl) are then consumed by the pretrained DCNet model.

## Related tools

- **DCNet** (Downstream consumer of the preprocessed expression matrix; the preprocessing step formats data to its required orientation and gene-symbol conventions) — https://github.com/xindd/DCNet
- **Python** (Environment in which the expression matrix loading, log2(x+1) transformation, and gene filtering are performed (DCNet is coded with mxnet in Python))
- **mxnet** (Framework dependency for the DCNet pipeline that consumes the preprocessed matrix; preprocessing must produce arrays compatible with mxnet loading)

## Examples

```
import pandas as pd, numpy as np; expr = pd.read_pickle('demo_data.pkl'); expr = np.log2(expr + 1); keep = (expr > 0).mean(axis=1) >= 1/3; expr_filtered = expr.loc[keep]
```

## Evaluation signals

- After filtering, the retained gene set should match the paper's scale (~21,136 genes when the 1/3-zero rule is applied to TCGA data)
- Matrix orientation check: every row is a human gene symbol and every column is a sample, matching DCNet's demo_data.pkl layout
- Values are log2(x+1)-transformed (non-negative, no raw counts passed to DCNet)
- Marker-gene audit: cells whose marker genes are absent from the TCGA gene sets are removed — 33 cell types should be discarded with no detected marker genes remaining
- Preprocessed matrix runs through the DCNet demo.ipynb without gene-index mismatches against index.pkl and relation_matrix.pkl

## Limitations

- The paper does not report random seeds or reproducibility controls, and no trained model weights beyond saved parameters are guaranteed, so preprocessed-input runs may vary slightly
- Genes whose expression is 0 in more than 1/3 of samples are permanently deleted, which may discard low-abundance but biologically relevant genes
- Only marker genes detected in the TCGA gene set are kept; cell types relying on unmeasured markers (33 discarded) cannot be deconvolved

## Evidence
