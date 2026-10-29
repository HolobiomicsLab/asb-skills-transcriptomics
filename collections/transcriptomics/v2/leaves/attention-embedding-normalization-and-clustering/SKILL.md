---
name: attention-embedding-normalization-and-clustering
description: Use when TOSICA.pre() has returned new_adata with the attention matrix
  in .X and you need to trajectory-order or cluster cells on regulon/pathway token
  attention rather than raw gene expression.
when_to_use_negative:
- When you only need predicted cell-type labels and probabilities — TOSICA.pre() already
  returns new_adata.obs['Prediction'] and 'Probability' without embedding analysis.
- When the query AnnData genes do not match the reference var_names in the same order
  — the attention matrix is not valid until query_adata = query_adata[:, ref_adata.var_names]
  is applied.
- When the model was trained with abstract (non-biological) tokens rather than pathway/regulon
  masks such as human_tf (TF.gmt), since token attention then lacks regulon-level
  interpretability.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: TOSICA
  role: Produces the attention matrix (new_adata.X) and token annotations (var['pathway_index'])
    that this skill normalizes and clusters
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: scanpy 1.7.1 performs library-size normalization, PCA, nearest-neighbor graph,
    UMAP, and diffusion-map pseudotime on the attention embeddings
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:64d5c82c029a482c41192a5f141e07bba873cd5e85dd339a729913c68d0c2310
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# attention-embedding-normalization-and-clustering

## Summary

Post-hoc analysis of TOSICA's attention matrix: library-size normalize the per-cell attention embeddings (default size factor 10,000), then run diffusion-map pseudotime and neighborhood clustering (PCA, nearest-neighbor graph, UMAP) to recover interpretable regulatory dynamics from the token-level attention space.

## When to use

Use when TOSICA.pre() has returned new_adata with the attention matrix in .X and you need to trajectory-order or cluster cells on regulon/pathway token attention rather than raw gene expression.

## When NOT to use

- When you only need predicted cell-type labels and probabilities — TOSICA.pre() already returns new_adata.obs['Prediction'] and 'Probability' without embedding analysis.
- When the query AnnData genes do not match the reference var_names in the same order — the attention matrix is not valid until query_adata = query_adata[:, ref_adata.var_names] is applied.
- When the model was trained with abstract (non-biological) tokens rather than pathway/regulon masks such as human_tf (TF.gmt), since token attention then lacks regulon-level interpretability.

## Inputs

- TOSICA.pre() output AnnData: attention matrix in new_adata.X, predicted labels and probabilities in new_adata.obs, token/gene-set names in new_adata.var['pathway_index']
- Cell-state labels (e.g. HomC, preHTC, HTC) in query_adata.obs for trajectory comparisons

## Outputs

- Library-size-normalized attention embedding (size factor 10,000)
- Diffusion-map pseudotime ordering of cells in attention space
- PCA, nearest-neighbor graph, and UMAP built on attention embeddings
- Per-token attention tracks along pseudotime (e.g. NF1 and CEBP regulon attention)

## How to apply

After TOSICA.pre() returns new_adata whose .X holds the attention matrix and whose var['pathway_index'] names the gene set (token) for each column, treat the attention matrix as a cell-by-feature embedding. Normalize it by library-size correction using the default size factor of 10,000, exactly as done in the paper's downstream analysis with scanpy 1.7.1. To extract a specific regulatory signal, restrict columns to tokens of interest via pathway_index — e.g. the NF1 and CEBP regulon tokens from the human_tf mask (TF.gmt) trained model. Then run diffusion-map pseudotime ordering on the attention embeddings and, in parallel, PCA followed by nearest-neighbor graph construction and UMAP for visualization/clustering. Comparing a regulon's attention along pseudotime between cell states (e.g. CEBP attention in preHTC vs HTC chondrocytes) turns the Transformer's traceable attention layers into a quantitative trajectory readout.

## Related tools

- **TOSICA** (Produces the attention matrix (new_adata.X) and token annotations (var['pathway_index']) that this skill normalizes and clusters) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (scanpy 1.7.1 performs library-size normalization, PCA, nearest-neighbor graph, UMAP, and diffusion-map pseudotime on the attention embeddings)

## Examples

```
new_adata = TOSICA.pre(query_adata, model_weight_path='./weights20220607/model-6.pth', project='my_project'); sc.pp.normalize_total(new_adata, target_sum=1e4)
```

## Evaluation signals

- The normalized attention matrix has per-cell attention values scaled by the default library-size factor of 10,000.
- Token columns are traceable to named gene sets via new_adata.var['pathway_index'] (e.g. NF1 and CEBP regulon tokens appear in the selection).
- Qualitative acceptance criterion from the hBone application: CEBP regulon attention is lower in preHTC than in HTC cells along the OA-onset trajectory.
- UMAP/neighborhood structure computed on attention embeddings separates the predicted cell states consistent with new_adata.obs['Prediction'] labels.
- Diffusion-map pseudotime ordering places HomC-to-HTC/preHTC cells along a continuous trajectory tracking NF1 attention.

## Limitations

- The number of attention heads (H) is not specified in the methods, limiting exact reproduction of the attention computation feeding the embeddings.
- Attention interpretability requires biologically relevant tokens (pathway/regulon masks); interpretability is lost when features are abstracted.
- Gene features of ref_adata and query_adata must be consistent and in the same order, or the attention matrix is misaligned with pathway_index tokens.

## Evidence

- [discussion] attention matrix normalization: "the matrix is normalized by library-size correction using default size factor 10,000"
- [results] diffusion-map pseudotime on attention embeddings: [evidence span withheld — anchor status: not_found]
- [discussion] token traceability: "The high accuracy and batch-insensitivity of TOSICA can be mainly attributed to the attention layers and tokens masked by high-level biologically relevant pathways or regulons"
- [readme] downstream embedding steps: "+ `new_adata.X` : Attention matrix
+ `new_adata.obs['Prediction']` : Predicted labels
+ `new_adata.var['pathway_index']` : Gene set of each colume"
- [intro] Transformer traceability rationale: "the Transformer framework does not involve dimensionality reduction10,12, thus keeping all attention layer traceable to the original input features13, thus making the models interpretable"
