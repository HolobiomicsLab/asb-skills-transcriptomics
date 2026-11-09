---
name: diffusion-map-pseudotime-trajectory-analysis
description: Use when you have TOSICA attention embeddings from a reference-annotated
  query dataset and need to order cells along a trajectory (e.g., HomC to HTC/preHTC
  chondrocyte maturation) to test how specific regulon attentions change with disease
  onset.
when_to_use_negative:
- The input is raw gene expression without a trained TOSICA model / reference annotation
  — the attention matrix must first be produced by TOSICA.pre().
- The analysis goal is de novo trajectory discovery from expression data rather than
  interpreting an annotated transition — standard expression-based pseudotime on the
  raw counts is more appropriate.
- ref_adata and query_adata var_names differ or are unordered — TOSICA prediction
  requires identical, consistently ordered genes.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0769
tools:
- name: TOSICA
  role: Produces the TF-regulon attention matrix (new_adata.X) and token-to-pathway
    mapping that serve as the pseudotime input; mask trained with human_tf (TF.gmt)
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Normalizes the attention matrix (library-size correction, size factor 10,000)
    and runs diffusion-map pseudotime ordering, version 1.7.1
- name: torch
  role: Deep-learning backend TOSICA depends on (torch 1.7.1) for training and prediction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:520bfa2ad5550440920f9f460769705512700a7f89db5957240e25dba391508d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# diffusion-map-pseudotime-trajectory-analysis

## Summary

Orders cells along a developmental or disease-progression trajectory by running diffusion-map pseudotime on TOSICA's TF-regulon attention matrix (new_adata.X), enabling regulon-level interpretation of cell-state transitions such as the NF1-to-CEBP regulon shift in chondrocyte differentiation.

## When to use

Use when you have TOSICA attention embeddings from a reference-annotated query dataset and need to order cells along a trajectory (e.g., HomC to HTC/preHTC chondrocyte maturation) to test how specific regulon attentions change with disease onset.

## When NOT to use

- The input is raw gene expression without a trained TOSICA model / reference annotation — the attention matrix must first be produced by TOSICA.pre().
- The analysis goal is de novo trajectory discovery from expression data rather than interpreting an annotated transition — standard expression-based pseudotime on the raw counts is more appropriate.
- ref_adata and query_adata var_names differ or are unordered — TOSICA prediction requires identical, consistently ordered genes.

## Inputs

- Reference AnnData with cell-type labels in obs (used for TOSICA.train with TF.gmt / human_tf mask)
- Query AnnData with var_names matching the reference in the same order
- Attention matrix (new_adata.X) from TOSICA.pre()
- pathway_index column from new_adata.var identifying each attention token's gene set

## Outputs

- Pseudotime-ordered cells over the normalized regulon-attention embedding
- Per-cell regulon attention trajectories (e.g., NF1 and CEBP attention vs pseudotime)
- Comparative statistics of regulon attention between trajectory stages (preHTC vs HTC)
- Per-cell predicted labels and probabilities (new_adata.obs['Prediction'], 'Probability')

## How to apply

Train TOSICA with TOSICA.train() on the reference AnnData using the human_tf mask (TF.gmt) so tokens correspond to transcription-factor regulons, then run TOSICA.pre() on the query cells — this returns the attention matrix in new_adata.X, predicted labels in new_adata.obs['Prediction'], and the token-to-geneset mapping in new_adata.var['pathway_index']. Restrict the attention matrix to the regulon tokens of interest (e.g., NF1 and CEBP) via pathway_index. Normalize the attention matrix by library-size correction using the default size factor of 10,000, then run diffusion-map pseudotime ordering on the attention embeddings with scanpy 1.7.1. Ensure ref_adata and query_adata share identical var_names in the same order (query_adata = query_adata[:, ref_adata.var_names]) before prediction. Finally, compare regulon attention (e.g., CEBP) along pseudotime between trajectory endpoints (preHTC vs HTC) to test whether the expected regulon transition is intact or disrupted.

## Related tools

- **TOSICA** (Produces the TF-regulon attention matrix (new_adata.X) and token-to-pathway mapping that serve as the pseudotime input; mask trained with human_tf (TF.gmt)) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Normalizes the attention matrix (library-size correction, size factor 10,000) and runs diffusion-map pseudotime ordering, version 1.7.1)
- **torch** (Deep-learning backend TOSICA depends on (torch 1.7.1) for training and prediction)

## Examples

```
new_adata = TOSICA.pre(query_adata[:, ref_adata.var_names], model_weight_path='./weights20220607/model-6.pth', project='human_tf_20220607'); TOSICA attention (new_adata.X) is then normalized (size factor 10,000) and diffusion-map pseudotime ordered with scanpy 1.7.1.
```

## Evaluation signals

- Validate the qualitative acceptance criterion: CEBP regulon attention is lower in preHTC than in HTC cells.
- Confirm NF1 attention tracks the HomC to HTC/preHTC trajectory, and that the NF1-to-CEBP regulon transition fails along the OA-onset trajectory, matching the reported finding.
- Check the attention matrix shape equals (n_query_cells, n_pathway_tokens) and that new_adata.var['pathway_index'] correctly maps NF1 and CEBP tokens.
- Verify ref_adata and query_adata var_names are identical and in the same order after subsetting query_adata = query_adata[:, ref_adata.var_names].
- Confirm predicted labels/probabilities in new_adata.obs are populated and consistent with the chondrocyte stages being ordered.

## Limitations

- TOSICA's accuracy and batch-insensitivity depend on tokens masked by biologically relevant gene sets; regulon-token interpretation is only as good as the TF.gmt mask.
- Attention is used as an embedding for pseudotime rather than a causal measure; ordering is run on normalized attention, not raw expression.
- The number of attention heads (H) is not specified in the methods, so per-head attention behavior cannot be reproduced exactly.

## Evidence

- [discussion] attention matrix normalized before pseudotime: "the matrix is normalized by library-size correction using default size factor 10,000"
- [readme] attention output fields from TOSICA.pre: "+ `new_adata.X` : Attention matrix
+ `new_adata.obs['Prediction']` : Predicted labels
+ `new_adata.var['pathway_index']` : Gene set of each colume"
- [readme] gene order warning: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [discussion] regulon-masked tokens explain interpretability: "The high accuracy and batch-insensitivity of TOSICA can be mainly attributed to the attention layers and tokens masked by high-level biologically relevant pathways or regulons"
