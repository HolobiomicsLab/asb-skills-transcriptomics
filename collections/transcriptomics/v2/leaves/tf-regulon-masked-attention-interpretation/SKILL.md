---
name: tf-regulon-masked-attention-interpretation
description: Use when you need biologically interpretable cell-level attention over
  TF regulons from a trained TOSICA model — e.g., to order cells in pseudotime by
  regulon attention or compare regulon dominance between cell states or disease conditions.
when_to_use_negative:
- When only a generic cell-type label is needed without interpretability — other annotation
  tools suffice
- When reference and query gene sets (var_names) cannot be made consistent and identically
  ordered, since TOSICA requires this
- When the goal requires pathway-level (GO/Reactome) rather than TF-regulon-level
  token interpretability — use human_gobp or human_reactome masks instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: TOSICA
  role: Trains TF-masked Transformer model and returns per-cell attention over regulon
    tokens via TOSICA.train()/TOSICA.pre()
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Library-size normalization of the attention matrix and diffusion-map pseudotime
    ordering on attention embeddings (version 1.7.1)
- name: torch
  role: Deep-learning backend for TOSICA's multi-head self-attention model (version
    1.7.1)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:1c3f3b4e9ae8bb460dd191de23776159d8c44ac3f5aae1c708a2ca125ecb8ce4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# TF-regulon-masked attention interpretation

## Summary

Uses TOSICA's multi-head self-attention with tokens masked by transcription-factor gene sets (TF.gmt) so that attention weights are traceable to TF regulons, then treats the attention matrix as an interpretable embedding for trajectory and regulon-dominance analyses (e.g., NF1-to-CEBP transition in osteoarthritis chondrocytes).

## When to use

Use when you need biologically interpretable cell-level attention over TF regulons from a trained TOSICA model — e.g., to order cells in pseudotime by regulon attention or compare regulon dominance between cell states or disease conditions.

## When NOT to use

- When only a generic cell-type label is needed without interpretability — other annotation tools suffice
- When reference and query gene sets (var_names) cannot be made consistent and identically ordered, since TOSICA requires this
- When the goal requires pathway-level (GO/Reactome) rather than TF-regulon-level token interpretability — use human_gobp or human_reactome masks instead

## Inputs

- Reference AnnData object of single-cell RNA-seq with cell-type labels in obs
- Query AnnData object with var_names matching the reference (same genes, same order)
- TF.gmt transcription-factor gene-set file (human_tf pre-prepared mask)

## Outputs

- new_adata.X: per-cell attention matrix over regulon tokens
- new_adata.obs['Prediction'] and new_adata.obs['Probability']: predicted labels and probabilities
- new_adata.var['pathway_index']: gene set (regulon) identity of each attention column
- ./my_project/gene2token_weights.csv: gene-to-token weight matrix
- Pseudotime-ordered cells with regulon attention values for dominance comparisons

## How to apply

Train TOSICA with TOSICA.train() on the reference AnnData using the human_tf mask (TF.gmt) so pathway tokens correspond to transcription-factor regulons; the reference and query must share identical var_names in the same order (query_adata = query_adata[:,ref_adata.var_names]). Predict on query cells with TOSICA.pre(), which returns the attention matrix in new_adata.X, predicted labels/probabilities in obs, and the gene-set identity of each column in new_adata.var['pathway_index']. Restrict the attention matrix to the regulon tokens of interest (e.g., NF1 and CEBP). Normalize the attention matrix by library-size correction with default size factor 10,000, then build trajectories (diffusion-map pseudotime with scanpy 1.7.1) directly on the attention embeddings. Interpretability is guaranteed because the Transformer involves no dimensionality reduction, keeping attention layers traceable to original input features masked by biologically relevant regulons.

## Related tools

- **TOSICA** (Trains TF-masked Transformer model and returns per-cell attention over regulon tokens via TOSICA.train()/TOSICA.pre()) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Library-size normalization of the attention matrix and diffusion-map pseudotime ordering on attention embeddings (version 1.7.1))
- **torch** (Deep-learning backend for TOSICA's multi-head self-attention model (version 1.7.1))

## Examples

```
TOSICA.train(ref_adata, 'human_tf', project='tf_regulon', label_name='celltype'); new_adata = TOSICA.pre(query_adata[:, ref_adata.var_names], project='tf_regulon')
```

## Evaluation signals

- new_adata.X has one column per gene set in pathway_index; verify columns restricted to NF1/CEBP regulon tokens map correctly via new_adata.var['pathway_index']
- Attention matrix normalized with library-size correction using default size factor 10,000 reproduces comparable per-cell attention scales
- Predicted labels in obs['Prediction'] match expected reference cell types with reasonable obs['Probability'] values
- Qualitative acceptance: CEBP regulon attention is lower in preHTC than HTC cells, and the NF1-to-CEBP regulon transition fails along the OA-onset trajectory
- Confirm ref_adata.var_names == query_adata.var_names in the same order before TOSICA.pre()

## Limitations

- Attention-head count (H) is not specified in the methods, limiting exact architectural reproducibility
- The repository README warns that the var_names (genes) of ref_adata and query_adata must be consistent and in the same order
- Learning rate value, batch size, and random seed are not reported despite SGD with cosine learning rate decay being named

## Evidence

- [discussion] TOSICA's accuracy and batch-insensitivity are attributed to attention layers with tokens masked by biologically relevant pathways or regulons: "The high accuracy and batch-insensitivity of TOSICA can be mainly attributed to the attention layers and tokens masked by high-level biologically relevant pathways or regulons"
- [discussion] Attention matrix normalization and downstream analysis: "the matrix is normalized by library-size correction using default size factor 10,000"
- [intro] Transformer keeps attention traceable to input features: "the Transformer framework does not involve dimensionality reduction10,12, thus keeping all attention layer traceable to the original input features13"
- [readme] TOSICA.pre outputs attention matrix and pathway index: "+ `new_adata.X` : Attention matrix
+ `new_adata.obs['Prediction']` : Predicted labels"
- [readme] Gene order requirement: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [methods] Training with TF mask: "+ `human_tf` : TF.gmt"
