---
name: query-reference-gene-alignment
description: Use when preparing a query scRNA-seq AnnData for TOSICA.pre() prediction
  after training on a ref_adata, and the query's var_names (genes) may differ from
  or be ordered differently than the reference genes.
when_to_use_negative:
- Query and reference AnnData already have identical var_names in the same order —
  no alignment is needed
- Input is raw count matrices or a feature table not stored as an AnnData object with
  var_names
- You intend to train a new model rather than predict with an existing trained project
  folder
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: TOSICA
  role: Annotation model whose prediction step (TOSICA.pre) requires the query gene
    list to be aligned to the reference gene order
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Provides the AnnData object used for subsetting query_adata[:, ref_adata.var_names]
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:6b4db9374af3bf78e23604ffbde07867d95ad3f9b5b34f9bbf44a2faf1e21279
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# query-reference-gene-alignment

## Summary

Subset and reorder the query AnnData so its var_names exactly match the reference training AnnData's gene list before running TOSICA prediction, since the trained Transformer model expects inputs in the reference gene order.

## When to use

Use when preparing a query scRNA-seq AnnData for TOSICA.pre() prediction after training on a ref_adata, and the query's var_names (genes) may differ from or be ordered differently than the reference genes.

## When NOT to use

- Query and reference AnnData already have identical var_names in the same order — no alignment is needed
- Input is raw count matrices or a feature table not stored as an AnnData object with var_names
- You intend to train a new model rather than predict with an existing trained project folder

## Inputs

- Query AnnData object (query_adata) of single-cell RNA-seq expression
- Trained reference AnnData (ref_adata) whose var_names define the required gene order
- Trained TOSICA project folder (mask.npy, pathway.csv, label_dictionary.csv, model-n.pth)

## Outputs

- Aligned query_adata whose var_names match ref_adata.var_names exactly and in the same order
- new_adata.obs['Prediction'], new_adata.obs['Probability'] and attention matrix from downstream TOSICA.pre()

## How to apply

After training TOSICA on a reference AnnData (e.g. hPancreas reference with alpha cells excluded via ref_adata subsetting), check that the query AnnData's var_names are consistent with ref_adata.var_names. Reorder or subset the query with query_adata = query_adata[:, ref_adata.var_names] so genes are identical and in the same order, because the mask matrix built from a .gmt file (e.g. human_reactome) maps input genes to pathway tokens positionally. Only then call TOSICA.pre(query_adata, project) to obtain new_adata.obs['Prediction'], new_adata.obs['Probability'] and the attention matrix. Skipping this alignment silently shifts gene-to-token assignments and invalidates predictions and gene2token interpretability weights.

## Related tools

- **TOSICA** (Annotation model whose prediction step (TOSICA.pre) requires the query gene list to be aligned to the reference gene order) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Provides the AnnData object used for subsetting query_adata[:, ref_adata.var_names])

## Examples

```
query_adata = query_adata[:, ref_adata.var_names]; new_adata = TOSICA.pre(query_adata, project='my_project')
```

## Evaluation signals

- Query var_names equals ref_adata.var_names element-wise after alignment (same length, same order, no NaNs)
- Downstream TOSICA.pre() returns new_adata.obs['Prediction'] and new_adata.obs['Probability'] without shape or mask-mismatch errors
- In the deleted-alpha-cell hPancreas validation, the fraction of held-out alpha cells labeled 'Unknown' matches the reported ~76%
- gene2token_weights.csv token mapping remains biologically sensible because gene positions were not shifted

## Limitations

- Genes absent from the query dataset are dropped by the subsetting, so the query must contain the reference genes for the mask mapping to be valid
- The README warning is easy to miss; unaligned inputs may still run but produce silently invalid attention and gene-to-token weights
- The GitHub URL in the discussion section contains a stray trailing '58' (https://github.com/JackieHanLab/TOSICA58), a potentially broken link

## Evidence

- [readme] the var_names (genes) of the ref_adata and query_adata must be consistent and in the same order.: "> **Warning:** the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [other] Align the held-out alpha-cell query AnnData to the reference gene order with query_adata = query_adata[:,ref_adata.var_names].: [evidence span withheld — anchor status: not_found]
- [discussion] expression levels of n genes are first embedded into k tokens using linear transformation weight (W): [evidence span withheld — anchor status: not_found]
