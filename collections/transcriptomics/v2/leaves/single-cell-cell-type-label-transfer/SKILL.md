---
name: single-cell-cell-type-label-transfer
description: Use when a query AnnData needs cell type labels transferred from an annotated
  reference (e.g. resolving monocyte subtypes in pan-cancer tumor-infiltrating myeloid
  data), and you want interpretable, pathway-token-based predictions with per-cell
  probabilities.
when_to_use_negative:
- The reference and query AnnData var_names differ in order or membership and you
  cannot subset the query — the README warns predictions require identical, ordered
  genes.
- You need an unsupervised discovery of entirely novel cell types with no labeled
  reference — this skill transfers labels from a reference dataset rather than de
  novo annotating clusters.
- Your goal is dimensionality-reduced latent embeddings for visualization rather than
  interpretable label transfer — TOSICA deliberately avoids dimensionality reduction
  to keep attention traceable.
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: TOSICA
  role: 'Core tool: TOSICA.train builds the pathway mask and trains the Transformer
    on the reference; TOSICA.pre transfers labels to the query and returns predictions,
    probabilities, and the attention matrix'
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Loads and manipulates the AnnData objects (subsetting query genes to ref_adata.var_names)
    for training and prediction
  repo: https://github.com/JackieHanLab/TOSICA
- name: torch
  role: Deep learning backend (torch 1.7.1) executing the multi-head self-attention
    Transformer model
  repo: https://github.com/JackieHanLab/TOSICA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:e5b58edbce9b16cf4c08117b1c8243319b9ad36384e34aba956933bef360c491
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# single-cell-cell-type-label-transfer

## Summary

Transfer cell type annotations from a labeled reference scRNA-seq dataset to a query dataset using TOSICA, a Transformer-based multi-head self-attention model whose attention layers are masked by biologically relevant pathway/regulon tokens (e.g. Reactome, GO_bp, immune, TF .gmt gene sets), yielding predictions, probabilities, and an interpretable attention matrix.

## When to use

Use when a query AnnData needs cell type labels transferred from an annotated reference (e.g. resolving monocyte subtypes in pan-cancer tumor-infiltrating myeloid data), and you want interpretable, pathway-token-based predictions with per-cell probabilities.

## When NOT to use

- The reference and query AnnData var_names differ in order or membership and you cannot subset the query — the README warns predictions require identical, ordered genes.
- You need an unsupervised discovery of entirely novel cell types with no labeled reference — this skill transfers labels from a reference dataset rather than de novo annotating clusters.
- Your goal is dimensionality-reduced latent embeddings for visualization rather than interpretable label transfer — TOSICA deliberately avoids dimensionality reduction to keep attention traceable.

## Inputs

- Reference AnnData (ref_adata) with cell type labels in ref_adata.obs[label_key]
- Query AnnData (query_adata) with genes ordered to match ref_adata.var_names
- Gene set mask (.gmt file or pre-prepared mask name: human_gobp, human_immune, human_reactome, human_tf, m_GO_bp, m_reactome, m_TF)
- Trained model weights (./<my_project>/model-*.pth) for prediction step

## Outputs

- ./<my_project>/mask.npy (mask matrix)
- ./<my_project>/pathway.csv (gene set list)
- ./<my_project>/label_dictionary.csv (label list)
- new_adata.obs['Prediction'] (predicted labels per query cell)
- new_adata.obs['Probability'] (softmax cell type probability per query cell)
- new_adata.X (attention matrix over pathway tokens)
- new_adata.var['pathway_index'] (gene set per column)
- ./<my_project>/gene2token_weights.csv (gene-to-token weights matrix)
- tosica_predictions.csv (assembled predictions, probabilities, attention)

## How to apply

First train with TOSICA.train(ref_adata, gmt_path, project=<my_project>, label_name=<label_key>), which writes ./<my_project>/mask.npy, pathway.csv, label_dictionary.csv, and model-n.pth; choose a pre-prepared mask such as human_immune, human_reactome, human_gobp, or human_tf, or supply your own .gmt file. Before prediction, align the query genes to the reference with query_adata = query_adata[:, ref_adata.var_names] since ref_adata and query_adata var_names must be consistent and in the same order. Run new_adata = TOSICA.pre(query_adata, model_weight_path=<path to model-*.pth>, project=<my_project>), then extract new_adata.obs['Prediction'], new_adata.obs['Probability'], the attention matrix new_adata.X, and new_adata.var['pathway_index']; read ./<my_project>/gene2token_weights.csv to trace attention tokens back to gene sets. The rationale is that TOSICA's Transformer does no dimensionality reduction, keeping attention traceable to original input features, and attention layers masked by pathway tokens give batch-insensitive integration while retaining biological variation. Assemble predictions, probabilities, and attention into tosica_predictions.csv and validate that every query cell has Prediction and Probability fields with probabilities in [0,1] and that the attention matrix column count matches the number of gene sets in pathway.csv.

## Related tools

- **TOSICA** (Core tool: TOSICA.train builds the pathway mask and trains the Transformer on the reference; TOSICA.pre transfers labels to the query and returns predictions, probabilities, and the attention matrix) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Loads and manipulates the AnnData objects (subsetting query genes to ref_adata.var_names) for training and prediction) — https://github.com/JackieHanLab/TOSICA
- **torch** (Deep learning backend (torch 1.7.1) executing the multi-head self-attention Transformer model) — https://github.com/JackieHanLab/TOSICA

## Examples

```
query_adata = query_adata[:, ref_adata.var_names]; new_adata = TOSICA.pre(query_adata, model_weight_path='./my_project/model-6.pth', project='my_project')
```

## Evaluation signals

- tosica_predictions.csv exists and contains Prediction and Probability fields for every query cell, with all probabilities within [0, 1]
- The attention matrix column count matches the number of gene sets listed in ./<my_project>/pathway.csv
- query_adata.var_names equals ref_adata.var_names in the same order after the subsetting step (no dropped or reordered genes)
- mask.npy dimensions are consistent with the number of input genes and pathway tokens derived from the .gmt mask
- Downstream label transfer is batch-insensitive: predicted labels are consistent across studies/biological states split into training and test sets (30% of training held out as validation)

## Limitations

- The var_names (genes) of ref_adata and query_adata must be consistent and in the same order — mismatched gene lists break prediction.
- Training uses SGD with cosine learning rate decay and cross-entropy loss, but the number of attention heads (H), learning rate value, batch size, and random seed are not reported in the methods.
- Tokens are masked by pre-selected gene sets (e.g. c2.cp.reactome.v7.5.1.symbols.gmt, GO_bp.gmt, immune.gmt, TF.gmt); annotations are limited by the coverage of the chosen mask.
- Training and test sets are split by study or biological state, so transfer to a study with an unseen biological state may degrade.

## Evidence

- [abstract] TOSICA achieves fast and accurate one-stop annotation and batch-insensitive integration while providing biologically interpretable insights: "TOSICA achieves fast and accurate one-stop annotation and batch-insensitive integration while providing biologically interpretable insights"
- [discussion] The high accuracy and batch-insensitivity of TOSICA can be mainly attributed to the attention layers and tokens masked by high-level biologically relevant pathways or regulons: "The high accuracy and batch-insensitivity of TOSICA can be mainly attributed to the attention layers and tokens masked by high-level biologically relevant pathways or regulons"
- [readme] Warning that ref/query genes must match: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [discussion] Softmax over fully connected layer on CLS token: "The CLS of O is used as input of a fully connected network and followed by a softmax function to obtain the probability of cell types"
- [discussion] Train/test split and validation: [evidence span withheld — anchor status: not_found]
