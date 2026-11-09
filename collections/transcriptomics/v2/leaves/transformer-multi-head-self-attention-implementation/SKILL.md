---
name: transformer-multi-head-self-attention-implementation
description: Use when implementing or reproducing a masked self-attention cell-type
  annotation model that maps n input genes to k pathway tokens and returns per-cell
  attention matrices and cell-type probability vectors.
when_to_use_negative:
- When the task requires latent-space embedding or dimensionality reduction of scRNA-seq
  data — the Transformer framework deliberately does not involve dimensionality reduction,
  so attention layers are kept traceable to original input features instead
- When reference and query gene spaces differ — the var_names (genes) of ref_adata
  and query_adata must be consistent and in the same order before prediction
- When no pathway/regulon gene-set knowledge (gmt file or pre-prepared mask) is available
  for the species or gene nomenclature
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: TOSICA
  role: Provides TOSICA.train to build the gmt-derived mask matrix and TOSICA.pre
    to run the trained attention model on query data
  repo: https://github.com/JackieHanLab/TOSICA
- name: torch
  role: Framework in which the masked learnable embedding, CLS token concatenation,
    and multi-head self-attention layer are implemented (torch 1.7.1)
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Handles AnnData reference/query inputs and outputs (new_adata.obs['Prediction'],
    new_adata.X attention matrix) (scanpy 1.7.1)
  repo: https://github.com/JackieHanLab/TOSICA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:7177a2c8648b075cf7723a3b4ad2687059878ae5829abc297e29f12829dcf12f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# transformer-multi-head-self-attention-implementation

## Summary

Implements TOSICA's masked Transformer pipeline in torch: gene expression is embedded into pathway tokens via a masked linear embedding (W' = W*M) from a gmt-derived mask, a learnable CLS token is prepended, and multi-head self-attention followed by a softmax fully connected layer yields interpretable cell-type probabilities.

## When to use

Use when implementing or reproducing a masked self-attention cell-type annotation model that maps n input genes to k pathway tokens and returns per-cell attention matrices and cell-type probability vectors.

## When NOT to use

- When the task requires latent-space embedding or dimensionality reduction of scRNA-seq data — the Transformer framework deliberately does not involve dimensionality reduction, so attention layers are kept traceable to original input features instead
- When reference and query gene spaces differ — the var_names (genes) of ref_adata and query_adata must be consistent and in the same order before prediction
- When no pathway/regulon gene-set knowledge (gmt file or pre-prepared mask) is available for the species or gene nomenclature

## Inputs

- Reference AnnData object (ref_adata) with cell-type labels in ref_adata.obs
- gmt gene-set file (e.g. GO_bp.gmt, immune.gmt, reactome.gmt, TF.gmt) or a pre-prepared mask name such as human_gobp
- Query AnnData object (query_adata) with var_names matching ref_adata in the same order
- Model weight file model-n.pth from the training step

## Outputs

- mask.npy, pathway.csv, and label_dictionary.csv written to the project folder
- Trained model weights saved as model-n.pth
- new_adata.obs['Prediction'] and new_adata.obs['Probability'] per query cell
- Attention matrix in new_adata.X (one per cell)
- gene2token_weights.csv mapping genes to tokens

## How to apply

Build the mask matrix from a gmt gene-set file (e.g. GO_bp.gmt / human_gobp) mapping input genes to pathways with TOSICA.train, saving mask.npy, pathway.csv, and label_dictionary.csv to the project folder. In torch, implement the masked learnable embedding by applying W' = W*M so the n input genes are linearly embedded into k pathway tokens, each restricted to its gene set. Concatenate a learnable CLS class token to the top of the pathway token matrix by row, then apply the multi-head self-attention layer with query (Q), key (K), and value (V) matrices each separately linearly projected from the input matrix. Train on the reference AnnData with SGD and cosine learning rate decay under cross-entropy loss, using a 30% validation split, and save weights as model-n.pth. At inference, the CLS token is fed through a fully connected network followed by softmax to obtain cell-type probabilities; the rationale is that attention tokens masked by biologically relevant pathways or regulons keep the model traceable to original input features without dimensionality reduction.

## Related tools

- **TOSICA** (Provides TOSICA.train to build the gmt-derived mask matrix and TOSICA.pre to run the trained attention model on query data) — https://github.com/JackieHanLab/TOSICA
- **torch** (Framework in which the masked learnable embedding, CLS token concatenation, and multi-head self-attention layer are implemented (torch 1.7.1)) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Handles AnnData reference/query inputs and outputs (new_adata.obs['Prediction'], new_adata.X attention matrix) (scanpy 1.7.1)) — https://github.com/JackieHanLab/TOSICA

## Examples

```
TOSICA.train(ref_adata, gmt_path='human_gobp', project='my_project', label_name='celltype'); new_adata = TOSICA.pre(query_adata[:, ref_adata.var_names], project='my_project')
```

## Evaluation signals

- Predictions are valid probability vectors over cell types (softmax outputs) with a matching new_adata.obs['Probability']
- The attention matrix is returned per cell in new_adata.X
- Training artifacts exist in the project folder: mask.npy, pathway.csv, label_dictionary.csv, and model-n.pth
- gene2token_weights.csv is produced with the gene-to-token weight matrix
- Query prediction succeeds only when query_adata var_names equal ref_adata.var_names in the same order (query_adata = query_adata[:,ref_adata.var_names])

## Limitations

- The number of attention heads (H) is not specified in the methods; queries, keys and values are linearly projected H times without a stated H value
- The learning rate value is not specified despite SGD with cosine learning rate decay being named; batch size and random seed are also not reported
- The pre-prepared masks are limited to specific human (GO_bp, immune, reactome, TF) and mouse (m_GO_bp, m_reactome, m_TF) gmt files; custom masks require your own gmt
- Reference and query AnnData must share identical gene order — mismatched var_names break prediction

## Evidence

- [discussion] In Multi-head self-attention layer, the query (Q), key (K), and value (V) matrix are separately linearly projected from input matrix: "In Multi-head self-attention layer, the query (Q), key (K), and value (V) matrix are separately linearly projected from input matrix"
- [discussion] a learnable parameter class token (CLS) is concatenated to T at the top by row: "a learnable parameter class token (CLS) is concatenated to T at the top by row"
- [discussion] Stochastic gradient descent (SGD) is chosen as optimizer, and we use cosine learning rate decay: "Stochastic gradient descent (SGD) is chosen as optimizer, and we use cosine learning rate decay to avoid too large steps in late stage of training"
- [readme] the var_names (genes) of the ref_adata and query_adata must be consistent and in the same order.: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [abstract] TOSICA achieves fast and accurate one-stop annotation and batch-insensitive integration while providing biologically interpretable insights: "TOSICA achieves fast and accurate one-stop annotation and batch-insensitive integration while providing biologically interpretable insights for understanding cellular behavior"
- [discussion] the matrix is normalized by library-size correction using default size factor 10,000: "the matrix is normalized by library-size correction using default size factor 10,000"
