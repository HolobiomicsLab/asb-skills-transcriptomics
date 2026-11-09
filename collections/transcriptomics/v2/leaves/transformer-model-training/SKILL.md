---
name: transformer-model-training
description: Use when a reference scRNA-seq dataset with known cell type labels (AnnData)
  must be used to train an interpretable annotation model that transfers labels to
  query datasets across batches.
when_to_use_negative:
- The query and reference AnnData objects cannot be made to share identical, identically
  ordered var_names — the README warns predictions require this consistency
- The task is unsupervised clustering of a single dataset rather than label transfer
  from an annotated reference
- The goal is de novo annotation without a labeled reference or pathway/regulon gene
  sets (gmt files) to mask tokens
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: TOSICA
  role: Core Python package providing TOSICA.train (reference model training with
    SGD and cosine learning rate decay) and TOSICA.pre (query annotation)
  repo: https://github.com/JackieHanLab/TOSICA
- name: torch
  role: Deep learning backend (version 1.7.1) implementing the multi-head self-attention
    Transformer layers
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: AnnData handling and downstream analysis of the attention matrix (library-size
    normalization, PCA, neighbor graph, UMAP)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:c4761178d9322481efd176a53cbccca6f6a7db81074b53f9dbc990cb08f248ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# transformer-model-training

## Summary

Train a TOSICA Transformer model (multi-head self-attention with pathway-masked tokens) on a reference scRNA-seq AnnData object to enable interpretable, batch-insensitive cell type label transfer to query datasets.

## When to use

Use when a reference scRNA-seq dataset with known cell type labels (AnnData) must be used to train an interpretable annotation model that transfers labels to query datasets across batches.

## When NOT to use

- The query and reference AnnData objects cannot be made to share identical, identically ordered var_names — the README warns predictions require this consistency
- The task is unsupervised clustering of a single dataset rather than label transfer from an annotated reference
- The goal is de novo annotation without a labeled reference or pathway/regulon gene sets (gmt files) to mask tokens

## Inputs

- Reference AnnData object (ref_adata) with cell type labels in obs
- GMT gene-set file or pre-prepared mask name (e.g. GO_bp.gmt, reactome.gmt, immune.gmt, TF.gmt, m_GO_bp.gmt, m_reactome.gmt, m_TF.gmt)
- Label key column name in ref_adata.obs
- Query AnnData object with var_names matching ref_adata.var_names exactly and in the same order

## Outputs

- ./my_project/mask.npy (mask matrix)
- ./my_project/pathway.csv (gene set list)
- ./my_project/label_dictionary.csv (label list)
- ./my_project/model-n.pth (model weights)
- new_adata.obs['Prediction'] (predicted labels), new_adata.obs['Probability'], new_adata.X (attention matrix), gene2token_weights.csv

## How to apply

Load the reference AnnData object and choose a pre-prepared mask (e.g. human_gobp from GO_bp.gmt or human_reactome from reactome.gmt) that maps input genes to pathway tokens via a masked linear transformation. Split training and test sets by study or biological state, reserving 30% of the training set as the validation set. Train with TOSICA.train using SGD with cosine learning rate decay and cross-entropy loss, which saves mask.npy, pathway.csv, label_dictionary.csv and model-n.pth weights into the project folder. The attention mechanism over pathway-masked tokens (plus a learnable CLS token followed by softmax) is what gives both accuracy and interpretability, since the Transformer avoids dimensionality reduction and keeps attention layers traceable to input genes. Before annotation, subset and reorder query_adata so its var_names exactly match ref_adata.var_names in the same order.

## Related tools

- **TOSICA** (Core Python package providing TOSICA.train (reference model training with SGD and cosine learning rate decay) and TOSICA.pre (query annotation)) — https://github.com/JackieHanLab/TOSICA
- **torch** (Deep learning backend (version 1.7.1) implementing the multi-head self-attention Transformer layers) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (AnnData handling and downstream analysis of the attention matrix (library-size normalization, PCA, neighbor graph, UMAP))

## Examples

```
pip install git+https://github.com/JackieHanLab/TOSICA.git; TOSICA.train(ref_adata, 'human_gobp', project='my_project', label_name='celltype')
```

## Evaluation signals

- Training outputs the four expected artifacts in the project folder: mask.npy, pathway.csv, label_dictionary.csv, model-n.pth
- Validation loss/accuracy computed on the 30% validation split held out from the training set
- Query annotation accuracy reaches the order of reported benchmark values (e.g. ~93.75% on hArtery, ~95.76% on hPancreas; mean 86.69% across six datasets)
- Predicted labels in new_adata.obs['Prediction'] are all present in label_dictionary.csv
- gene2token_weights.csv is produced, confirming interpretable gene-to-token weights are traceable to input genes

## Limitations

- var_names of ref_adata and query_adata must be consistent and in the same order, requiring explicit subsetting (query_adata = query_adata[:,ref_adata.var_names])
- Number of attention heads, learning rate value, batch size and random seed are not specified in the methods
- A more interpretable model may trade off against the untraceable, information-losing latent spaces of autoencoder-based approaches — accuracy depends on choosing biologically relevant pathway/regulon masks
- The GitHub URL in the discussion contains a stray trailing '58' (https://github.com/JackieHanLab/TOSICA58), a potentially broken repository link; the correct repo is https://github.com/JackieHanLab/TOSICA

## Evidence

- [readme] Training step with TOSICA.train: "TOSICA.train(ref_adata, gmt_path,project=<my_project>,label_name=<label_key>)"
- [readme] Pre-prepared masks in README: "+ `human_gobp` : GO_bp.gmt
+ `human_immune` : immune.gmt
+ `human_reactome` : reactome.gmt"
- [readme] Query gene order requirement: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [methods] Training split and validation: [evidence span withheld — anchor status: not_found]
- [methods] Optimizer and loss: "Stochastic gradient descent (SGD) is chosen as optimizer, and we use cosine learning rate decay"
- [intro] Interpretability rationale: "the Transformer framework does not involve dimensionality reduction10,12, thus keeping all attention layer traceable to the original input features13"
- [discussion] Attribution of accuracy: "The high accuracy and batch-insensitivity of TOSICA can be mainly attributed to the attention layers and tokens masked by high-level biologically relevant pathways or regulons"
