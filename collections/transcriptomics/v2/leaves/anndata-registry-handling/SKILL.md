---
name: anndata-registry-handling
description: 'Use when training or applying TOSICA on scanpy AnnData objects: a reference
  AnnData must be paired with a query AnnData whose var_names (genes) match in identity
  and order, and model artifacts must be written to and read from a project folder.'
when_to_use_negative:
- The query AnnData's genes cannot be subset to exactly match ref_adata.var_names
  in the same order — TOSICA explicitly warns var_names must be consistent and ordered
- The input is raw counts not yet organized as a scanpy AnnData with an .obs label
  column for the reference (TOSICA.train requires label_name to index ref_adata.obs)
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: TOSICA
  role: Provides TOSICA.train and TOSICA.pre, the functions that consume and produce
    AnnData objects and register mask/label/weight artifacts in the project folder
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: Supplies the AnnData container used for ref_adata and query_adata; var_names
    subsetting aligns query genes to the reference
- name: torch
  role: Backend implementing the masked embedding (W' = W*M) and multi-head self-attention;
    model weights are saved as model-n.pth in the project folder
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:59a995979818933ca7cca64190f17699382bf4958d27a00a016072eb186607bc
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# anndata-registry-handling

## Summary

Correct handling of AnnData objects when passing reference and query single-cell datasets through the TOSICA Transformer pipeline, including enforcing var_names consistency between reference and query and registering training artifacts (mask.npy, pathway.csv, label_dictionary.csv, model-n.pth) in a project folder.

## When to use

Use when training or applying TOSICA on scanpy AnnData objects: a reference AnnData must be paired with a query AnnData whose var_names (genes) match in identity and order, and model artifacts must be written to and read from a project folder.

## When NOT to use

- The query AnnData's genes cannot be subset to exactly match ref_adata.var_names in the same order — TOSICA explicitly warns var_names must be consistent and ordered
- The input is raw counts not yet organized as a scanpy AnnData with an .obs label column for the reference (TOSICA.train requires label_name to index ref_adata.obs)

## Inputs

- Reference AnnData (ref_adata) with a label column in .obs
- Query AnnData (query_adata) with var_names identical to and ordered as the reference
- Path to a .gmt gene-set file or a pre-prepared mask name (e.g. human_gobp / GO_bp.gmt)
- Project folder name and label key (label_name)

## Outputs

- ./my_project/mask.npy (mask matrix mapping genes to pathways)
- ./my_project/pathway.csv (gene set list)
- ./my_project/label_dictionary.csv (label list)
- ./my_project/model-n.pth (trained weights)
- new_adata.X (per-cell attention matrix)
- new_adata.obs['Prediction'] and new_adata.obs['Probability']
- new_adata.var['pathway_index'] and ./my_project/gene2token_weights.csv

## How to apply

Build the mask via TOSICA.train(ref_adata, gmt_path, project=<my_project>, label_name=<label_key>), which saves mask.npy, pathway.csv, and label_dictionary.csv into the project folder along with the trained model-n.pth weights. Before prediction, align the query AnnData to the reference with query_adata = query_adata[:, ref_adata.var_names] because TOSICA requires the var_names of ref_adata and query_adata to be consistent and in the same order (the masked embedding W' = W*M maps the same n genes to k pathway tokens). Run TOSICA.pre(query_adata, model_weight_path=<path>, project=<my_project>) to obtain a new AnnData whose .X holds the attention matrix and whose .obs carries 'Prediction' and 'Probability' columns. Verify that gene-to-token weights are exported as gene2token_weights.csv and that new_adata.var['pathway_index'] records the gene set for each column.

## Related tools

- **TOSICA** (Provides TOSICA.train and TOSICA.pre, the functions that consume and produce AnnData objects and register mask/label/weight artifacts in the project folder) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (Supplies the AnnData container used for ref_adata and query_adata; var_names subsetting aligns query genes to the reference)
- **torch** (Backend implementing the masked embedding (W' = W*M) and multi-head self-attention; model weights are saved as model-n.pth in the project folder)

## Examples

```
query_adata = query_adata[:, ref_adata.var_names]; new_adata = TOSICA.pre(query_adata, model_weight_path='./my_project/model-n.pth', project='my_project')
```

## Evaluation signals

- query_adata[:, ref_adata.var_names] executes without dropping genes — i.e. query and reference var_names sets are identical
- Project folder contains all four artifacts: mask.npy, pathway.csv, label_dictionary.csv, and model-n.pth
- new_adata.obs['Prediction'] contains valid cell-type labels and new_adata.obs['Probability'] holds softmax probability vectors over cell types
- new_adata.X is a per-cell attention matrix (one row per query cell, columns as pathway tokens)
- new_adata.var['pathway_index'] exists and gene2token_weights.csv is written to the project folder with gene-to-token weights

## Limitations

- var_names of ref_adata and query_adata must be consistent and in the same order; mismatched gene ordering silently breaks the masked gene-to-pathway embedding
- The number of attention heads (H), learning rate, batch size, and random seed are not reported in the methods, hindering exact reproduction
- The pretrained masks are species- and source-specific (human_gobp, human_immune, human_reactome, human_tf, mouse_gobp, mouse_reactome, mouse_tf); other gene sets require a custom .gmt

## Evidence

- [readme] Warning: the var_names (genes) of the ref_adata and query_adata must be consistent and in the same order.: "> **Warning:** the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [readme] Output listing of AnnData-adjacent artifacts: mask.npy, pathway.csv, label_dictionary.csv, model-n.pth: "+ `./my_project/mask.npy` : Mask matrix
+ `./my_project/pathway.csv` : Gene set list
+ `./my_project/label_dictionary.csv` : Label list
+ `./my_project/model-n.pth` : Weights"
- [readme] Query prediction outputs attached to the returned AnnData object: "+ `new_adata.X` : Attention matrix
+ `new_adata.obs['Prediction']` : Predicted labels
+ `new_adata.obs['Probability']` : Probability of the prediction"
- [discussion] Masked embedding maps n genes to k pathway tokens under the gmt-derived mask: [evidence span withheld — anchor status: not_found]
- [discussion] Training split specification used with the reference AnnData: [evidence span withheld — anchor status: not_found]
