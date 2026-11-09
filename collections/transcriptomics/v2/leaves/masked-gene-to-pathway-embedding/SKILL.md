---
name: masked-gene-to-pathway-embedding
description: Use when building an interpretable Transformer-based cell-type annotation
  model where input genes must be grouped into biologically relevant pathway tokens
  (from .gmt files such as GO_bp.gmt or reactome.gmt) before self-attention.
when_to_use_negative:
- Query and reference AnnData var_names differ or are not in the same order — the
  embedding assumes identical gene ordering (use query_adata = query_adata[:, ref_adata.var_names]
  first).
- You need a dimensionality-reducing autoencoder latent space; TOSICA deliberately
  avoids dimensionality reduction to keep attention traceable.
- No biologically relevant gene-set resource (.gmt mask) is available for the species
  or feature space of the data.
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: TOSICA
  role: Builds the gene-to-pathway mask matrix from gmt files during TOSICA.train
    and provides the masked embedding layer of the Transformer model
  repo: https://github.com/JackieHanLab/TOSICA
- name: torch
  role: Implements the masked learnable linear embedding (W' = W*M) producing pathway
    tokens and the subsequent multi-head self-attention layer
- name: scanpy
  role: Handles the reference and query AnnData single-cell objects fed into the embedding
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:5fe54ac38059c576521139ed5a8c24d83d1c1146d9c7ac9c75491356584d1d61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# masked-gene-to-pathway-embedding

## Summary

Embed the expression of n input genes into k pathway tokens via a masked linear transformation (W' = W*M), where the mask matrix is derived from a GSEA .gmt gene-set file, as the first layer of TOSICA's Transformer cell-type annotation model. This keeps every attention layer traceable to original gene features, enabling interpretable annotation.

## When to use

Use when building an interpretable Transformer-based cell-type annotation model where input genes must be grouped into biologically relevant pathway tokens (from .gmt files such as GO_bp.gmt or reactome.gmt) before self-attention.

## When NOT to use

- Query and reference AnnData var_names differ or are not in the same order — the embedding assumes identical gene ordering (use query_adata = query_adata[:, ref_adata.var_names] first).
- You need a dimensionality-reducing autoencoder latent space; TOSICA deliberately avoids dimensionality reduction to keep attention traceable.
- No biologically relevant gene-set resource (.gmt mask) is available for the species or feature space of the data.

## Inputs

- Reference AnnData object (ref_adata) with cell-type labels in ref_adata.obs
- gmt gene-set file (e.g. GO_bp.gmt, reactome.gmt, immune.gmt, TF.gmt) or a pre-prepared mask name such as human_gobp
- Label column name in ref_adata.obs (label_name)

## Outputs

- mask.npy (mask matrix mapping genes to pathways)
- pathway.csv (gene set list)
- label_dictionary.csv (label list)
- Pathway token matrix T (n genes embedded into k tokens), with learnable CLS token prepended

## How to apply

First build the mask matrix with TOSICA.train from a gmt gene-set file (e.g. the pre-prepared human_gobp/GO_bp.gmt mask), which maps input genes to pathways and saves mask.npy, pathway.csv, and label_dictionary.csv to the project folder. In torch, implement the masked learnable embedding as W' = W*M so the n input genes are linearly embedded into k tokens, each restricted to its gene set. Concatenate a learnable CLS class token to the top of the pathway token matrix by row before feeding the multi-head self-attention layer, in which query, key, and value matrices are separately linearly projected. The rationale is that the Transformer framework does not involve dimensionality reduction, so attention layers remain traceable to original input features; masking tokens by pathways/regulons gives TOSICA its accuracy and batch-insensitivity.

## Related tools

- **TOSICA** (Builds the gene-to-pathway mask matrix from gmt files during TOSICA.train and provides the masked embedding layer of the Transformer model) — https://github.com/JackieHanLab/TOSICA
- **torch** (Implements the masked learnable linear embedding (W' = W*M) producing pathway tokens and the subsequent multi-head self-attention layer)
- **scanpy** (Handles the reference and query AnnData single-cell objects fed into the embedding)

## Examples

```
TOSICA.train(ref_adata, gmt_path='human_gobp', project='my_project', label_name='celltype')
```

## Evaluation signals

- mask.npy, pathway.csv, and label_dictionary.csv exist in the project folder after TOSICA.train
- Every k pathway token is restricted to its gene set: the mask enforces each token's embedding weights are zero outside its mapped genes
- The CLS token is concatenated to the top of the pathway token matrix by row before attention
- Token count k equals the number of gene sets in the gmt file; input dimension equals the n genes shared (in order) by reference and query var_names

## Limitations

- var_names of ref_adata and query_adata must be consistent and in the same order, otherwise predictions are invalid
- Number of attention heads (H) is not specified in the methods, complicating exact reimplementation
- Interpretability depends on the quality/relevance of the chosen gene sets (pathways or regulons); alternative masks like c2.cp.reactome.v7.5.1.symbols.gmt or c3.all.v7.5.1.symbols.gmt change token semantics

## Evidence

- [discussion] expression levels of n genes are first embedded into k tokens using linear transformation weight (W): [evidence span withheld — anchor status: not_found]
- [discussion] a learnable parameter class token (CLS) is concatenated to T at the top by row: "a learnable parameter class token (CLS) is concatenated to T at the top by row"
- [intro] the Transformer framework does not involve dimensionality reduction10,12, thus keeping all attention layer traceable to the original input features13: "the Transformer framework does not involve dimensionality reduction10,12, thus keeping all attention layer traceable to the original input features13"
- [discussion] the matrix is normalized by library-size correction using default size factor 10,000: "we map the input genes to selected gene sets (gmt files)"
- [readme] the var_names (genes) of the ref_adata and query_adata must be consistent and in the same order.: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [readme] ./my_project/mask.npy : Mask matrix: "+ `./my_project/mask.npy` : Mask matrix
+ `./my_project/pathway.csv` : Gene set list
+ `./my_project/label_dictionary.csv` : Label list"
