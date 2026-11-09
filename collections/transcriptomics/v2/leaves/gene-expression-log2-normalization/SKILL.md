---
name: gene-expression-log2-normalization
description: Use when a gene expression matrix (rows = human gene symbols, columns
  = samples) is about to be fed into DCNet or a deconvolution baseline, and raw counts/TPM
  values are not yet log-scaled.
when_to_use_negative:
- The expression matrix is already log-transformed (e.g., pre-normalized log2 TPM),
  where a second log2(x+1) pass would distort the values DCNet expects
- The downstream tool expects non-log linear-scale expression (some deconvolution
  baselines have their own input normalization conventions, so check each before reusing
  the DCNet-transformed matrix)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: DCNet
  role: Deep-learning deconvolution model whose recommended input is the log2(x+1)-transformed
    gene-by-sample expression matrix
  repo: https://github.com/xindd/DCNet
- name: Python
  role: Environment (with pandas/numpy) in which the elementwise log2(x+1) transformation
    is applied to the expression matrix before DCNet inference
provenance:
  source_task_ids:
  - task_003
  - task_005
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:c66bb275a936cb0039d65c720b06137d94c5dbc64a7f51aba8b18411b0518dde
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - CIBERSORTx
  - ConsensusClusterPlus
  - DCNet
  - EPIC
  - MCP-counter
  - Python
  - TIMER
  - quanTIseq
  - xCell
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# gene-expression-log2-normalization

## Summary

Applies the log2(x+1) transformation to gene expression matrices so bulk RNA-seq values are on the scale expected by DCNet and downstream deconvolution/clustering analyses (e.g., pseudo-bulk CRC/melanoma matrices and TCGA landscapes).

## When to use

Use when a gene expression matrix (rows = human gene symbols, columns = samples) is about to be fed into DCNet or a deconvolution baseline, and raw counts/TPM values are not yet log-scaled.

## When NOT to use

- The expression matrix is already log-transformed (e.g., pre-normalized log2 TPM), where a second log2(x+1) pass would distort the values DCNet expects
- The downstream tool expects non-log linear-scale expression (some deconvolution baselines have their own input normalization conventions, so check each before reusing the DCNet-transformed matrix)

## Inputs

- Gene expression matrix with rows as human gene symbols and columns as samples (e.g., pseudo-bulk aggregated GSE81861/GSE72056 single-cell data or TCGA bulk RNA-seq)
- demo_data.pkl-style expression profile from the DCNet-Use repository as a format reference

## Outputs

- Log2(x+1)-transformed expression matrix in the same gene-by-sample layout, ready as DCNet input
- Transformed matrix used for downstream cell-abundance inference and consensus clustering

## How to apply

Verify the input matrix is oriented with each row as a human gene symbol and each column as a sample, as required by DCNet's input format. Apply the log2(x+1) transformation elementwise to the expression values before running DCNet inference (e.g., on GSE81861 CRC or GSE72056 melanoma pseudo-bulk aggregates, or the 10,176-sample TCGA bulk matrix). This variance-stabilizing transform compresses the dynamic range of highly expressed genes so the mxnet-implemented network — trained with adam optimizer, relu activation, L2 loss, learning rate 1e-4, 600 iterations, batch size 256 — sees inputs on the scale it was trained on. The same transformed matrix should be used for consistency when comparing DCNet abundances against baselines such as CIBERSORTx, EPIC, MCP-counter, quanTIseq, TIMER, and xCell, or when clustering TCGA landscapes with ConsensusClusterPlus (manhattan distance, PAM, 1000 iterations).

## Related tools

- **DCNet** (Deep-learning deconvolution model whose recommended input is the log2(x+1)-transformed gene-by-sample expression matrix) — https://github.com/xindd/DCNet
- **Python** (Environment (with pandas/numpy) in which the elementwise log2(x+1) transformation is applied to the expression matrix before DCNet inference)

## Examples

```
import numpy as pd_np, pandas as pd; expr_log = np.log2(expr_raw + 1)  # expr_raw: DataFrame rows=gene symbols, cols=samples
```

## Evaluation signals

- All transformed values are non-negative, since log2(x+1) maps zero expression to 0 and positive values to positive reals
- Matrix dimensions (genes x samples) are unchanged by the transformation, preserving row = gene symbol, column = sample orientation
- Zero-expression entries in the raw matrix map to exactly 0 after transformation, providing a simple spot-check invariant
- Downstream DCNet runs (and reproduced Figure 3a correlations, e.g., DCNet T/B cell 0.81/0.95 in melanoma) are only expected to match when the recommended transform was applied consistently

## Limitations

- The article reports no software version numbers or random seeds, so the exact scaling behavior across mxnet/pandas/numpy versions is not pinned down
- No fine-tuning procedure for the recommended dropout-0.1 DCNet on user sample data is specified, so the transform's interaction with fine-tuning is undocumented
- The recommendation applies to DCNet input specifically; baselines were each run with default parameters and may require different input normalization

## Evidence
