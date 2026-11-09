---
name: reference-query-label-transfer
description: Use when a query scRNA-seq dataset needs cell type labels consistent
  with an annotated reference, and gene-order-matched AnnData objects (ref_adata,
  query_adata) are available for label transfer.
when_to_use_negative:
- Query and reference AnnData objects cannot be made to share identical var_names
  in the same order — the README warns predictions require this and the workflow fails
  otherwise
- The task is unsupervised discovery of novel cell types rather than label transfer
  from an existing annotated reference; TOSICA predicts only labels present in label_dictionary.csv
- Interpretability at the pathway-token level is not needed and a simpler classifier
  with lower overhead would suffice
edam_operation: http://edamontology.org/operation_0322
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: TOSICA
  role: 'Core model: TOSICA.train fits the masked multi-head self-attention Transformer
    on the reference; TOSICA.pre annotates the query and returns Prediction, Probability,
    and the attention matrix'
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: AnnData container and single-cell data handling for ref_adata/query_adata
    preprocessing and subsetting (scanpy 1.7.1 reported)
- name: torch
  role: Deep learning backend for the Transformer (torch 1.7.1, with cudatoolkit 10.1
    per the README conda recipe)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:917b5b8145249c6e791bcd7507d8809299c19a656dcc1b243e28084e86ff7618
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: http://edamontology.org/operation_0322
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# reference-query-label-transfer

## Summary

Transfer cell type labels from a labeled reference scRNA-seq dataset (AnnData) to a new query dataset using TOSICA, a Transformer-based multi-head self-attention annotator whose tokens are masked by pathway gene sets (e.g. GO_bp.gmt, reactome.gmt), yielding predictions, probabilities, and interpretable attention weights.

## When to use

Use when a query scRNA-seq dataset needs cell type labels consistent with an annotated reference, and gene-order-matched AnnData objects (ref_adata, query_adata) are available for label transfer.

## When NOT to use

- Query and reference AnnData objects cannot be made to share identical var_names in the same order — the README warns predictions require this and the workflow fails otherwise
- The task is unsupervised discovery of novel cell types rather than label transfer from an existing annotated reference; TOSICA predicts only labels present in label_dictionary.csv
- Interpretability at the pathway-token level is not needed and a simpler classifier with lower overhead would suffice

## Inputs

- ref_adata: AnnData object of the labeled reference dataset with a cell type label column in ref_adata.obs
- query_adata: AnnData object of the query dataset (var_names matching ref_adata.var_names exactly and in the same order)
- gmt_path: a pre-prepared mask name (human_gobp, human_immune, human_reactome, human_tf, mouse_gobp, mouse_reactome, mouse_tf) or path to a .gmt file
- label_name: the name of the label column in ref_adata.obs

## Outputs

- new_adata.obs['Prediction']: predicted cell type labels for query cells
- new_adata.obs['Probability']: probability of each prediction
- new_adata.X: attention matrix over pathway tokens
- ./<my_project>/mask.npy, pathway.csv, label_dictionary.csv, model-n.pth (training artifacts)
- ./<my_project>/gene2token_weights.csv: gene-to-token weight matrix for interpretability
- Annotation accuracy (fraction of correctly predicted cells), e.g. recorded in predictions.csv

## How to apply

Build the model with TOSICA.train(ref_adata, gmt_path, project, label_name), choosing a pre-prepared mask (e.g. human_gobp/GO_bp.gmt, human_reactome/reactome.gmt, mouse_tf/m_TF.gmt) that maps input genes to pathway tokens; training artifacts (mask.npy, pathway.csv, label_dictionary.csv, model-n.pth) are written to the project folder. Split training and test sets by study or biological state, reserving 30% of the training set as validation, and train with SGD using cosine learning rate decay and cross-entropy loss. Before prediction, subset and reorder query_adata so its var_names exactly match ref_adata.var_names (`query_adata = query_adata[:,ref_adata.var_names]`), since TOSICA requires identical gene order. Run TOSICA.pre(query_adata, model_weight_path, project) to obtain new_adata.obs['Prediction'], new_adata.obs['Probability'], the attention matrix in new_adata.X, and gene2token_weights.csv; compute accuracy as the fraction of correctly predicted cells against the query labels. The Transformer's attention layers operate directly on pathway tokens without dimensionality reduction, which is what keeps the model batch-insensitive and interpretable.

## Related tools

- **TOSICA** (Core model: TOSICA.train fits the masked multi-head self-attention Transformer on the reference; TOSICA.pre annotates the query and returns Prediction, Probability, and the attention matrix) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (AnnData container and single-cell data handling for ref_adata/query_adata preprocessing and subsetting (scanpy 1.7.1 reported))
- **torch** (Deep learning backend for the Transformer (torch 1.7.1, with cudatoolkit 10.1 per the README conda recipe))

## Examples

```
new_adata = TOSICA.pre(query_adata[:, ref_adata.var_names], model_weight_path='./my_project/model-n.pth', project='my_project'); acc = (new_adata.obs['Prediction'] == query_adata.obs[label_name]).mean()
```

## Evaluation signals

- Annotation accuracy (fraction of correctly predicted cells) should be on the order of reported benchmarks: ~93.75% on hArtery, ~95.76% on hPancreas, ~81.06% on the largest unbalanced dataset mAtlas (vs 79.57% for ACTINN)
- Every value in new_adata.obs['Prediction'] must be present in the label_dictionary.csv produced during training
- query_adata.var_names equals ref_adata.var_names element-wise and in the same order before calling TOSICA.pre
- Training artifacts mask.npy, pathway.csv, label_dictionary.csv, and model weights exist in the project folder before prediction
- Predicted labels should be stable across batch/study splits — TOSICA's batch-insensitivity means accuracy should hold on held-out studies or biological states, not just random splits

## Limitations

- var_names of ref_adata and query_adata must be consistent and in the same order; mismatched gene sets require subsetting, which drops genes absent from the reference
- Random seed, batch size, learning rate value, and number of attention heads (H) are not specified in the methods, complicating exact reproduction
- Predictions are limited to the label vocabulary captured in label_dictionary.csv; truly novel/rare cell types can only appear as misclassifications
- Accuracy depends on the chosen mask (GO_bp, Reactome, immune, TF); a biologically mismatched gmt can degrade token interpretability and transfer quality

## Evidence

- [abstract] Here, we present TOSICA, a multi-head self-attention deep learning model based on Transformer that enables interpretable cell type annotation: "Here, we present TOSICA, a multi-head self-attention deep learning model based on Transformer that enables interpretable cell type annotation"
- [readme] Warning: var_names must be consistent: "> **Warning:** the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [discussion] Split and train protocol: [evidence span withheld — anchor status: not_found]
- [discussion] SGD with cosine decay: "Stochastic gradient descent (SGD) is chosen as optimizer, and we use cosine learning rate decay to avoid too large steps in late stage of training"
- [results] Benchmark accuracy: [evidence span withheld — anchor status: not_found]
