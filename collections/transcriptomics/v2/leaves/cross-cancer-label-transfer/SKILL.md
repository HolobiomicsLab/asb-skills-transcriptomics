---
name: cross-cancer-label-transfer
description: Use when a query single-cell RNA-seq dataset (e.g., six-cancer-type myeloid
  cells) needs cell-type labels transferred from a differently sourced reference (e.g.,
  KIDNEY/UCEC/ESCA pan-cancer myeloid), especially across batches or cancer types
  where annotation consistency and interpretability matter.
when_to_use_negative:
- The reference and query AnnData cannot be reconciled to identical, same-ordered
  var_names (genes), which TOSICA explicitly warns against.
- You need unsupervised discovery of novel cell types rather than transferring existing
  reference labels to a query dataset.
- The task requires a method applicable to a very large number of batches where Seurat-based
  integration approaches are being compared — note some competitors become inapplicable
  above ~85 batches, so benchmark scope must be checked per method.
edam_operation: http://edamontology.org/operation_0372
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: TOSICA
  role: 'Core Transformer model: TOSICA.train builds the pathway mask and trains on
    the reference; TOSICA.pre predicts query labels and returns the attention matrix'
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Loads reference and query single-cell data as AnnData objects and performs
    downstream PCA/UMAP on the normalized attention matrix
- name: torch
  role: Deep-learning backend (torch 1.7.1 required) on which the TOSICA multi-head
    self-attention model is implemented
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:1581a51bdb9a735bf3b5486eb153d3c1414a1026806c95e4668c449478c2ff3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_0372
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# cross-cancer-label-transfer

## Summary

Transfer cell-type labels across cancers by training a TOSICA Transformer model with a pathway-masked attention layer on a reference scRNA-seq dataset and predicting labels on a query dataset, using scanpy AnnData objects and pre-prepared gmt masks such as reactome.gmt. The pathway masking makes the model batch-insensitive while retaining interpretable, traceable attention over original input genes.

## When to use

Use when a query single-cell RNA-seq dataset (e.g., six-cancer-type myeloid cells) needs cell-type labels transferred from a differently sourced reference (e.g., KIDNEY/UCEC/ESCA pan-cancer myeloid), especially across batches or cancer types where annotation consistency and interpretability matter.

## When NOT to use

- The reference and query AnnData cannot be reconciled to identical, same-ordered var_names (genes), which TOSICA explicitly warns against.
- You need unsupervised discovery of novel cell types rather than transferring existing reference labels to a query dataset.
- The task requires a method applicable to a very large number of batches where Seurat-based integration approaches are being compared — note some competitors become inapplicable above ~85 batches, so benchmark scope must be checked per method.

## Inputs

- Reference AnnData (ref_adata) with cell-type labels in an obs column
- Query AnnData (query_adata) with genes reordered as query_adata[:, ref_adata.var_names]
- Pre-prepared mask name or path to a .gmt file (e.g., human_reactome / reactome.gmt)
- Label key in ref_adata.obs identifying the annotation column

## Outputs

- Project folder with mask.npy, pathway.csv, label_dictionary.csv, model-n.pth
- new_adata.X: attention matrix for query cells
- new_adata.obs['Prediction']: predicted cell-type labels
- new_adata.obs['Probability']: prediction probabilities
- new_adata.var['pathway_index'] and gene2token_weights.csv for interpretability
- Optionally, normalized attention embedding plus PCA/UMAP and scIB metric scores (e.g., scib_ranking.csv)

## How to apply

Load the reference and query datasets as scanpy AnnData objects and force gene consistency with query_adata[:, ref_adata.var_names], since TOSICA requires ref_adata and query_adata var_names to be identical and in the same order. Train with TOSICA.train(ref_adata, gmt_path='human_reactome', project=<my_project>, label_name=<label_key>), which maps input genes to REACTOME gene sets from reactome.gmt, builds the mask matrix, and writes mask.npy, pathway.csv, label_dictionary.csv, and model-n.pth into the project folder. Run TOSICA.pre(query_adata, project=<my_project>) to obtain the attention matrix in new_adata.X, predicted labels in new_adata.obs['Prediction'], probabilities in new_adata.obs['Probability'], and gene-to-token weights in gene2token_weights.csv. For integration-style use, normalize the attention matrix by library-size correction (default size factor 10,000) before PCA, nearest-neighbor graph construction, and UMAP; pathway-masked tokens keep the latent space traceable to original genes. Training uses SGD with cosine learning-rate decay and cross-entropy loss, splitting studies or biological states into train/test with 30% of the training set held out as validation. If benchmarking against other integration methods, compute scIB metrics and combine as overall score = 0.6 × biology conservation + 0.4 × batch removal (note Seurat-based methods are inapplicable above ~85 batches).

## Related tools

- **TOSICA** (Core Transformer model: TOSICA.train builds the pathway mask and trains on the reference; TOSICA.pre predicts query labels and returns the attention matrix) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Loads reference and query single-cell data as AnnData objects and performs downstream PCA/UMAP on the normalized attention matrix)
- **torch** (Deep-learning backend (torch 1.7.1 required) on which the TOSICA multi-head self-attention model is implemented)

## Examples

```
TOSICA.train(ref_adata, gmt_path='human_reactome', project='pancan_myeloid', label_name='celltype'); new_adata = TOSICA.pre(query_adata[:, ref_adata.var_names], project='pancan_myeloid')
```

## Evaluation signals

- Project folder from training contains exactly mask.npy, pathway.csv, label_dictionary.csv, and model-n.pth.
- Query prediction output has non-empty obs['Prediction'] and obs['Probability'] for all query cells, with labels drawn from label_dictionary.csv.
- ref_adata.var_names equals query_adata.var_names in identical order before TOSICA.pre is run.
- Attention matrix has one row per query cell and one column per pathway/gene set, matching pathway.csv / var['pathway_index'].
- If benchmarked, scIB overall score = 0.6 × biology conservation + 0.4 × batch removal, and the ranking table contains the expected number of applicable methods (11 in the pan-cancer myeloid benchmark, with TOSICA ranked second).

## Limitations

- The var_names (genes) of ref_adata and query_adata must be consistent and in the same order; mismatches break prediction.
- Number of attention heads (H), learning rate, batch size, and random seed are not specified in the methods, complicating exact reproduction.
- Alternative masks exist (GO_bp.gmt, immune.gmt, TF.gmt, mouse variants) but the article does not give guidance on mask choice beyond default pre-prepared options.
- Seurat-based comparison methods are inapplicable for >85 batches, limiting cross-method benchmark coverage.

## Evidence

- [readme] Reference-vs-query training with reactome mask: "TOSICA.train(ref_adata, gmt_path,project=<my_project>,label_name=<label_key>)"
- [readme] Warning on gene consistency: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [discussion] Attention normalization for integration: "the matrix is normalized by library-size correction using default size factor 10,000"
- [discussion] Pathway masking rationale: "The high accuracy and batch-insensitivity of TOSICA can be mainly attributed to the attention layers and tokens masked by high-level biologically relevant pathways or regulons"
- [discussion] Training procedure: "Stochastic gradient descent (SGD) is chosen as optimizer, and we use cosine learning rate decay"
