---
name: scrna-seq-prediction-probability-extraction
description: Use when a trained TOSICA Transformer model has annotated query scRNA-seq
  cells (e.g., monocyte subtypes in pan-cancer tumor-infiltrating myeloid data) and
  per-cell Prediction, Probability, and gene-set attention values must be exported
  for validation or enrichment testing.
when_to_use_negative:
- You only need cluster labels from an unsupervised workflow (Leiden/Seurat clustering)
  rather than reference-based label transfer with interpretability — TOSICA.pre requires
  a trained model and aligned reference genes.
- The query AnnData gene names/order do not match ref_adata.var_names and cannot be
  subset to match — predictions would be invalid.
- The input is already a feature/attention table rather than raw cell-by-gene expression;
  this skill operates on AnnData objects consumed by TOSICA.pre.
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: TOSICA
  role: Provides TOSICA.pre() which returns the AnnData containing obs['Prediction'],
    obs['Probability'], attention matrix X, and writes gene2token_weights.csv
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Loads and aligns the query AnnData object (query_adata = query_adata[:, ref_adata.var_names])
    before prediction
- name: torch
  role: Backend (version 1.7.1) on which the trained Transformer model weights are
    applied during TOSICA.pre
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:c39eec395f76bba93448729c72edd003fec677b7bda46145c19ce9cb575da983
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# scrna-seq-prediction-probability-extraction

## Summary

Extract TOSICA's per-cell predicted labels, softmax probabilities, and the attention matrix from the AnnData returned by TOSICA.pre(), assembling them into a single tosica_predictions.csv table for downstream cell-type and tissue-enrichment analyses.

## When to use

Use when a trained TOSICA Transformer model has annotated query scRNA-seq cells (e.g., monocyte subtypes in pan-cancer tumor-infiltrating myeloid data) and per-cell Prediction, Probability, and gene-set attention values must be exported for validation or enrichment testing.

## When NOT to use

- You only need cluster labels from an unsupervised workflow (Leiden/Seurat clustering) rather than reference-based label transfer with interpretability — TOSICA.pre requires a trained model and aligned reference genes.
- The query AnnData gene names/order do not match ref_adata.var_names and cannot be subset to match — predictions would be invalid.
- The input is already a feature/attention table rather than raw cell-by-gene expression; this skill operates on AnnData objects consumed by TOSICA.pre.

## Inputs

- query AnnData object (scRNA-seq counts, genes aligned to ref_adata.var_names)
- trained TOSICA model weights (model-*.pth) from the project folder
- project folder contents: pathway.csv (gene set list), gene2token_weights.csv, mask.npy, label_dictionary.csv

## Outputs

- tosica_predictions.csv with per-cell Prediction, Probability, and attention-matrix columns
- new_adata.obs['Prediction'] — predicted cell type labels
- new_adata.obs['Probability'] — softmax probability in [0, 1]
- new_adata.X — attention matrix (one column per gene set)
- new_adata.var['pathway_index'] — gene set of each attention column

## How to apply

First align the query AnnData to the reference by subsetting query_adata = query_adata[:, ref_adata.var_names], since TOSICA requires var_names to be consistent and in the same order. Run new_adata = TOSICA.pre(query_adata, model_weight_path=<path to model-*.pth>, project=<my_project>) to obtain label transfer from the trained model. Extract new_adata.obs['Prediction'], new_adata.obs['Probability'], and the attention matrix new_adata.X, and read ./<my_project>/gene2token_weights.csv to link attention tokens back to gene sets via pathway.csv. Assemble predictions, probabilities, and attention into one table and write it as tosica_predictions.csv; the rationale is that TOSICA's Transformer avoids dimensionality reduction, so the attention matrix remains traceable to original pathway-token inputs and is interpretable.

## Related tools

- **TOSICA** (Provides TOSICA.pre() which returns the AnnData containing obs['Prediction'], obs['Probability'], attention matrix X, and writes gene2token_weights.csv) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Loads and aligns the query AnnData object (query_adata = query_adata[:, ref_adata.var_names]) before prediction)
- **torch** (Backend (version 1.7.1) on which the trained Transformer model weights are applied during TOSICA.pre)

## Examples

```
import scanpy as sc, TOSICA; query_adata = sc.read_h5ad('query.h5ad')[:, ref_adata.var_names]; new_adata = TOSICA.pre(query_adata, model_weight_path='./weights/model-6.pth', project='my_project')
```

## Evaluation signals

- tosica_predictions.csv exists and contains Prediction and Probability fields for every query cell
- All Probability values lie within [0, 1], consistent with the softmax over the fully connected CLS layer
- Attention matrix column count equals the number of gene sets listed in pathway.csv
- Every predicted label maps to an entry in the project's label_dictionary.csv
- Query AnnData var_names match ref_adata.var_names exactly in identity and order before running TOSICA.pre

## Limitations

- ref_adata and query_adata var_names (genes) must be consistent and in the same order, or prediction fails/invalid results; the README explicitly warns to run query_adata = query_adata[:, ref_adata.var_names].
- Requires a model already trained via TOSICA.train with a GMT-derived mask (e.g., human_immune immune.gmt); TOSICA.pre cannot run without the project folder's model weights and pathway.csv.
- The number of attention heads (H) is not specified in the methods, limiting full architectural reproducibility of the attention matrix.

## Evidence

- [readme] new_adata.X : Attention matrix
+ `new_adata.obs['Prediction']` : Predicted labels
+ `new_adata.obs['Probability']` : Probability of the prediction: "new_adata.X : Attention matrix
+ `new_adata.obs['Prediction']` : Predicted labels
+ `new_adata.obs['Probability']` : Probability of the prediction"
- [readme] the var_names (genes) of the ref_adata and query_adata must be consistent and in the same order: "> **Warning:** the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [discussion] The CLS of O is used as input of a fully connected network and followed by a softmax function to obtain the probability of cell types: "The CLS of O is used as input of a fully connected network and followed by a softmax function to obtain the probability of cell types"
- [intro] the Transformer framework does not involve dimensionality reduction, thus keeping all attention layer traceable to the original input features: "the Transformer framework does not involve dimensionality reduction10,12, thus keeping all attention layer traceable to the original input features13"
