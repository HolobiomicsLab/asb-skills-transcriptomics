---
name: gene-to-token-weight-mapping
description: Use when you have run TOSICA.pre on a query AnnData and need to trace
  attention weights back to gene sets (e.g., reactome.gmt, GO_bp.gmt tokens) to interpret
  why cells received particular cell type predictions.
when_to_use_negative:
- The reference and query AnnData var_names are not consistent and in the same order
  — gene-to-token weights assume the same gene ordering used at training
- You need latent-space or dimensionality-reduced embeddings rather than attention
  traceable to original input features
- No gmt pathway mask was used during training, so attention tokens cannot be mapped
  back to gene sets
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: TOSICA
  role: Trains the masked Transformer and returns the attention matrix (new_adata.X)
    plus gene2token_weights.csv linking genes to pathway tokens
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Loads the query AnnData object and subsets it to ref_adata.var_names before
    prediction
- name: torch
  role: Backend for TOSICA's multi-head self-attention model and reading model-*.pth
    weights (torch 1.7.1)
  repo: https://github.com/JackieHanLab/TOSICA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:52755c4c62c0626d3b35942cc5144f3efe8ab55e281a82ad8ef814fe622bbca0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# Gene-to-token weight mapping

## Summary

Maps TOSICA's multi-head self-attention tokens back to input genes and gene sets (pathway tokens) so that predicted cell type annotations from the Transformer model are biologically interpretable, using ./<my_project>/gene2token_weights.csv and the attention matrix returned in new_adata.X.

## When to use

Use when you have run TOSICA.pre on a query AnnData and need to trace attention weights back to gene sets (e.g., reactome.gmt, GO_bp.gmt tokens) to interpret why cells received particular cell type predictions.

## When NOT to use

- The reference and query AnnData var_names are not consistent and in the same order — gene-to-token weights assume the same gene ordering used at training
- You need latent-space or dimensionality-reduced embeddings rather than attention traceable to original input features
- No gmt pathway mask was used during training, so attention tokens cannot be mapped back to gene sets

## Inputs

- Query AnnData object (h5ad) with var_names aligned to ref_adata.var_names via query_adata = query_adata[:, ref_adata.var_names]
- Trained TOSICA model weights (./<my_project>/model-*.pth)
- ./<my_project>/pathway.csv listing the gene sets used as attention tokens
- ./<my_project>/gene2token_weights.csv (gene-to-token weight matrix)
- GMT mask file used at training time (e.g., c2.cp.reactome.v7.5.1.symbols.gmt, GO_bp.gmt, immune.gmt, TF.gmt)

## Outputs

- Attention matrix (new_adata.X) with one column per pathway token
- Per-cell predicted labels (new_adata.obs['Prediction']) and probabilities (new_adata.obs['Probability'])
- Assembled per-cell table (e.g., tosica_predictions.csv) linking predictions, probabilities, and attention over gene sets
- Pathway-level attribution of cell type predictions (which gene sets drive each label)

## How to apply

After training TOSICA with a gmt mask (e.g., human_reactome/reactome.gmt), run TOSICA.pre(query_adata, model_weight_path=<path to model-*.pth>, project=<my_project>) and collect new_adata.X (the attention matrix over pathway tokens), new_adata.obs['Prediction'], and new_adata.obs['Probability']. TOSICA's Transformer performs no dimensionality reduction, so each attention layer remains traceable to the original input genes; read ./<my_project>/gene2token_weights.csv to obtain the linear gene-to-token embedding weights linking individual genes to pathway tokens, and ./<my_project>/pathway.csv for the gene set list. Assemble predictions, probabilities, and the attention matrix into a table (e.g., tosica_predictions.csv), then attribute per-cell attention to specific pathways/gene sets — for example, to show that TOSICA-resolved monocyte subtypes (C1 apoptotic CD14 enriched in tumors, paired t-test p=0.021; C0 CD16 in matched non-tumor tissues, p=0.0015) are driven by distinct gene sets. Validation: probabilities in [0,1], attention matrix column count equals the number of gene sets in pathway.csv, and query_adata.var_names were aligned to ref_adata.var_names before prediction.

## Related tools

- **TOSICA** (Trains the masked Transformer and returns the attention matrix (new_adata.X) plus gene2token_weights.csv linking genes to pathway tokens) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Loads the query AnnData object and subsets it to ref_adata.var_names before prediction)
- **torch** (Backend for TOSICA's multi-head self-attention model and reading model-*.pth weights (torch 1.7.1)) — https://github.com/JackieHanLab/TOSICA

## Examples

```
query_adata = query_adata[:, ref_adata.var_names]; new_adata = TOSICA.pre(query_adata, model_weight_path='./my_project/model-6.pth', project='my_project')
```

## Evaluation signals

- tosica_predictions.csv exists and contains Prediction and Probability fields for every query cell
- All Probability values fall within [0, 1]
- Attention matrix (new_adata.X) column count equals the number of gene sets listed in pathway.csv
- query_adata.var_names match ref_adata.var_names exactly and in the same order before running TOSICA.pre
- gene2token_weights.csv rows (genes) are consistent with the input gene universe defined by the gmt mask

## Limitations

- Interpretability relies on tokens masked by pre-defined gene sets (gmt files); predictions cannot be traced beyond the chosen pathway vocabulary
- The number of attention heads (H) is not specified in the methods, complicating per-head attribution
- GitHub URL in the article contains a stray trailing '58' (https://github.com/JackieHanLab/TOSICA58), potentially a broken repository link

## Evidence

- [intro] The Transformer framework does not involve dimensionality reduction, keeping attention layers traceable to original input features and thus interpretable: "the Transformer framework does not involve dimensionality reduction10,12, thus keeping all attention layer traceable to the original input features13"
- [readme] gene2token_weights.csv output from TOSICA.pre: "+ `./my_project/gene2token_weights.csv` : The weights matrix of genes to tokens"
- [discussion] attention tokens masked by biologically relevant pathways: "The high accuracy and batch-insensitivity of TOSICA can be mainly attributed to the attention layers and tokens masked by high-level biologically relevant pathways or regulons"
- [readme] genes must be consistent and ordered between reference and query: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [discussion] expression embedded into pathway tokens via linear transformation: [evidence span withheld — anchor status: not_found]
