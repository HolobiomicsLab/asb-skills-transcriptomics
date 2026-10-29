---
name: bone-disease-cell-state-trajectory-comparison
description: Use when scRNA-seq query cells from a disease-vs-control bone/cartilage
  context must be ordered along a developmental trajectory (e.g., HomC → preHTC →
  HTC chondrocyte maturation) and the key question is whether an attention-defined
  regulatory transition fails in the disease state.
when_to_use_negative:
- When the goal is de novo trajectory inference from raw gene expression rather than
  comparison of attention-defined regulon states against a trained reference — TOSICA
  requires a trained model and matched gene set
- When input genes are not traceable to the TF.gmt regulon mask (e.g., non-human data
  or genes not matching ref_adata.var_names), since token meaning depends on the human_tf
  gene-set mapping
- When an abstract feature space is sufficient and no biological interpretability
  of the trajectory driver is needed — other annotation-transfer tools without regulon-masked
  tokens may be faster
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: TOSICA
  role: Trains the regulon-masked Transformer (TOSICA.train with human_tf/TF.gmt)
    and predicts on query cells (TOSICA.pre) to produce the attention matrix used
    as regulon-state embeddings
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Normalizes the attention matrix by library-size correction (default size factor
    10,000) and performs diffusion-map pseudotime ordering of the attention embeddings
- name: torch
  role: Deep learning backend required to train and run the TOSICA multi-head self-attention
    model (torch version 1.7.1 per the article and README)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:0ffe828ff82dc0f009462b4da62828c8e4715673eeebed529e1e89ce0db0b431
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# bone-disease-cell-state-trajectory-comparison

## Summary

Compare chondrocyte developmental state trajectories between healthy and diseased (osteoarthritis hBone) conditions using TOSICA's TF-regulon attention matrix (from the human_tf/TF.gmt mask) as embeddings for pseudotime ordering, to detect whether a regulon transition (NF1-to-CEBP dominance) is disrupted along the disease-onset trajectory.

## When to use

Use when scRNA-seq query cells from a disease-vs-control bone/cartilage context must be ordered along a developmental trajectory (e.g., HomC → preHTC → HTC chondrocyte maturation) and the key question is whether an attention-defined regulatory transition fails in the disease state.

## When NOT to use

- When the goal is de novo trajectory inference from raw gene expression rather than comparison of attention-defined regulon states against a trained reference — TOSICA requires a trained model and matched gene set
- When input genes are not traceable to the TF.gmt regulon mask (e.g., non-human data or genes not matching ref_adata.var_names), since token meaning depends on the human_tf gene-set mapping
- When an abstract feature space is sufficient and no biological interpretability of the trajectory driver is needed — other annotation-transfer tools without regulon-masked tokens may be faster

## Inputs

- hBone reference AnnData (chondrocyte populations: HomC, preHTC, HTC) with cell type labels in obs
- Query hBone AnnData with var_names matched to ref_adata.var_names in the same order
- Pre-prepared human_tf mask (TF.gmt) or a path to a custom transcription-factor .gmt file
- Trained TOSICA model weights (model-n.pth from the <my_project> folder)
- scanpy 1.7.1 for library-size normalization and diffusion-map pseudotime ordering

## Outputs

- new_adata.X: per-cell attention matrix over regulon tokens (rows = query cells, columns = TF regulons)
- new_adata.obs['Prediction'] and new_adata.obs['Probability'] per query cell
- new_adata.var['pathway_index']: gene-set (regulon) identity of each attention column
- Diffusion-map pseudotime ordering of query cells on the attention embeddings
- Per-regulon attention curves along pseudotime, highlighting NF1 (HomC-to-HTC/preHTC) and CEBP (HTC) regulon dominance
- ./my_project/gene2token_weights.csv: genes-to-tokens weight matrix

## How to apply

Train TOSICA with TOSICA.train() on the hBone reference AnnData using the human_tf mask (TF.gmt) so pathway tokens correspond to transcription-factor regulons, then run TOSICA.pre() on the query hBone cells, which returns the attention matrix in new_adata.X alongside predicted labels and probabilities. Restrict the attention matrix to NF1 and CEBP regulon tokens using pathway_index from new_adata.var, aligning query genes to ref_adata.var_names so tokens are directly interpretable as regulon activations. Normalize the attention matrix with library-size correction (default size factor 10,000) and run diffusion-map pseudotime ordering on the attention embeddings with scanpy 1.7.1. Compare CEBP regulon attention along pseudotime between preHTC and HTC cells, testing whether the NF1-to-CEBP regulon transition is disrupted: the accepted qualitative criterion is that NF1 attention tracks the HomC-to-HTC/preHTC trajectory and CEBP attention is lower in preHTC than HTC, indicating a failed transition that characterizes OA onset. The rationale is that the Transformer keeps attention traceable to the original input features (regulon tokens) rather than abstract latent dimensions.

## Related tools

- **TOSICA** (Trains the regulon-masked Transformer (TOSICA.train with human_tf/TF.gmt) and predicts on query cells (TOSICA.pre) to produce the attention matrix used as regulon-state embeddings) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Normalizes the attention matrix by library-size correction (default size factor 10,000) and performs diffusion-map pseudotime ordering of the attention embeddings)
- **torch** (Deep learning backend required to train and run the TOSICA multi-head self-attention model (torch version 1.7.1 per the article and README))

## Examples

```
new_adata = TOSICA.pre(query_adata[:, ref_adata.var_names], model_weight_path='./human_tf_20220607/model-6.pth', project='human_tf')
```

## Evaluation signals

- Gene order check: query_adata[:, ref_adata.var_names] applied before TOSICA.pre — var_names of ref_adata and query_adata must be consistent and in the same order
- Regulon-token schema check: new_adata.var['pathway_index'] correctly maps attention columns to TF regulons (NF1, CEBP) from the TF.gmt mask
- Qualitative acceptance criterion: CEBP regulon attention is lower in preHTC than in HTC cells
- Trajectory criterion: NF1 attention tracks the HomC-to-HTC/preHTC trajectory while the NF1-to-CEBP transition fails along the OA-onset trajectory
- Training outputs exist: ./my_project/mask.npy, pathway.csv, label_dictionary.csv, model-n.pth generated by TOSICA.train

## Limitations

- The var_names (genes) of ref_adata and query_adata must be consistent and in the same order; mismatched gene ordering silently invalidates the attention-to-regulon mapping
- Key hyperparameters are under-specified in the paper: number of attention heads (H), learning rate value despite SGD with cosine decay, batch size, and random seed are not reported
- Attention-based trajectories are qualitative for the disease-onset comparison; the acceptance criterion (CEBP attention lower in preHTC than HTC) is a directional, not quantitative-statistical, test

## Evidence

- [readme] Attention matrix as embedding: "+ `new_adata.X` : Attention matrix"
- [readme] var_names consistency requirement: "> **Warning:** the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [discussion] Normalization and pseudotime basis: "the matrix is normalized by library-size correction using default size factor 10,000"
- [intro] Interpretability rationale for attention embeddings: "the Transformer framework does not involve dimensionality reduction10,12, thus keeping all attention layer traceable to the original input features"
- [discussion] Regulon-masked tokens drive accuracy: "The high accuracy and batch-insensitivity of TOSICA can be mainly attributed to the attention layers and tokens masked by high-level biologically relevant pathways or regulons"
