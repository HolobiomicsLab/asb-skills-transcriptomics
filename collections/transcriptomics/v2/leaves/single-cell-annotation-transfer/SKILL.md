---
name: single-cell-annotation-transfer
description: Use when a newly generated scRNA-seq query dataset must be labeled by
  transferring annotations from a curated reference AnnData, and interpretable per-cell
  attention (e.g., TF-regulon tokens for trajectory analysis) is desired.
when_to_use_negative:
- The reference and query AnnData have inconsistent or differently ordered var_names
  and cannot be aligned to a common gene space (a stated hard requirement of TOSICA.pre).
- The goal is de novo unsupervised clustering of a dataset without a curated reference
  annotation to transfer from.
- Interpretability at the level of original input features is unnecessary and an untraceable
  autoencoder latent space is acceptable — although TOSICA remains usable, this skill's
  value (traceable regulon attention) would be unused.
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: TOSICA
  role: Trains the Transformer model on the reference dataset and predicts query cell
    labels, returning the attention matrix, predicted labels, and probabilities
  repo: https://github.com/JackieHanLab/TOSICA
- name: torch
  role: Deep learning backend (version 1.7.1) on which the TOSICA multi-head self-attention
    model runs
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Downstream analysis of the attention matrix (library-size normalization with
    default size factor 10,000, PCA, nearest-neighbor graph, UMAP, pseudotime)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:588bc2170c60cb9b0181d096aca420e327e76c789791568969e0dc439a14912f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-annotation-transfer

## Summary

Transfer cell type labels from a reference scRNA-seq dataset to a query dataset using TOSICA, a Transformer-based multi-head self-attention model whose pathway/regulon tokens are masked by biologically relevant gene sets (e.g., TF.gmt), yielding interpretable, batch-insensitive annotation with traceable attention.

## When to use

Use when a newly generated scRNA-seq query dataset must be labeled by transferring annotations from a curated reference AnnData, and interpretable per-cell attention (e.g., TF-regulon tokens for trajectory analysis) is desired.

## When NOT to use

- The reference and query AnnData have inconsistent or differently ordered var_names and cannot be aligned to a common gene space (a stated hard requirement of TOSICA.pre).
- The goal is de novo unsupervised clustering of a dataset without a curated reference annotation to transfer from.
- Interpretability at the level of original input features is unnecessary and an untraceable autoencoder latent space is acceptable — although TOSICA remains usable, this skill's value (traceable regulon attention) would be unused.

## Inputs

- Reference AnnData object (ref_adata) with cell type labels in a ref_adata.obs column
- Query AnnData object (query_adata) with var_names matching and ordered as ref_adata.var_names
- Gene-set mask file (.gmt), e.g., TF.gmt (human_tf), GO_bp.gmt, immune.gmt, reactome.gmt, or mouse equivalents

## Outputs

- new_adata.X: attention matrix over pathway/regulon tokens
- new_adata.obs['Prediction']: predicted cell type labels per query cell
- new_adata.obs['Probability']: probability of the predicted label
- new_adata.var['pathway_index']: gene set (gene set name) for each token column
- ./<my_project>/gene2token_weights.csv: gene-to-token weight matrix
- Model artifacts: mask.npy, pathway.csv, label_dictionary.csv, model-n.pth

## How to apply

Train TOSICA with TOSICA.train() on the reference AnnData using a chosen gene-set mask (e.g., human_tf: TF.gmt so tokens correspond to transcription-factor regulons), with the label column in ref_adata.obs and a project folder holding mask.npy, pathway.csv, label_dictionary.csv, and model-n.pth. Ensure query_adata var_names exactly match and are ordered as ref_adata.var_names (query_adata = query_adata[:, ref_adata.var_names]) before calling TOSICA.pre(), which returns new_adata.X as the attention matrix plus obs['Prediction'] and obs['Probability']. The environment is Python >= 3.8 with torch 1.7.1 and scanpy 1.7.1; downstream attention analysis normalizes by library-size correction (default size factor 10,000) before PCA/nearest-neighbor/UMAP or diffusion-map pseudotime. Rationale: the Transformer avoids dimensionality reduction, so attention layers remain traceable to original input features (genes mapped to gene sets via the mask), enabling regulon-level interpretation of annotations and trajectories.

## Related tools

- **TOSICA** (Trains the Transformer model on the reference dataset and predicts query cell labels, returning the attention matrix, predicted labels, and probabilities) — https://github.com/JackieHanLab/TOSICA
- **torch** (Deep learning backend (version 1.7.1) on which the TOSICA multi-head self-attention model runs) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Downstream analysis of the attention matrix (library-size normalization with default size factor 10,000, PCA, nearest-neighbor graph, UMAP, pseudotime))

## Examples

```
TOSICA.train(ref_adata, 'human_tf', project='my_project', label_name='celltype'); new_adata = TOSICA.pre(query_adata, project='my_project')
```

## Evaluation signals

- new_adata.X has one row per query cell and one column per gene-set token, with new_adata.var['pathway_index'] naming the gene set for each column (e.g., NF1 and CEBP regulons when using human_tf/TF.gmt).
- obs['Prediction'] labels fall within the label_dictionary.csv vocabulary from the reference training project folder.
- Qualitative acceptance (as in the hBone/OA task): CEBP regulon attention is lower in preHTC than HTC cells and the NF1-to-CEBP regulon transition tracks the HomC to HTC/preHTC trajectory.
- Annotation accuracy and batch-insensitivity match the reported behavior attributed to pathway/regulon-masked attention layers (e.g., consistent labels across studies when training/test are split by study or biological state).

## Limitations

- var_names (genes) of ref_adata and query_adata must be consistent and in the same order; misalignment breaks prediction.
- Number of attention heads (H), learning rate, batch size, and random seed are not specified in the methods, complicating exact reproduction.
- The workflow schematic (Supplementary Fig. 17) referenced in the paper is not included in the available document, and one README citation of the repo URL contained a stray trailing '58' (broken link).

## Evidence

- [abstract] TOSICA, a multi-head self-attention deep learning model based on Transformer that enables interpretable cell type annotation: "Here, we present TOSICA, a multi-head self-attention deep learning model based on Transformer that enables interpretable cell type annotation"
- [readme] Attention matrix returned in new_adata.X with predictions and pathway_index: "+ `new_adata.X` : Attention matrix
+ `new_adata.obs['Prediction']` : Predicted labels
+ `new_adata.var['pathway_index']` : Gene set of each colume"
- [readme] Gene ordering requirement between reference and query: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [discussion] Attention matrix normalized by library-size correction with default size factor 10,000: "the matrix is normalized by library-size correction using default size factor 10,000"
- [intro] Traceability of attention to original input features is the interpretability rationale: "the Transformer framework does not involve dimensionality reduction10,12, thus keeping all attention layer traceable to the original input features13"
