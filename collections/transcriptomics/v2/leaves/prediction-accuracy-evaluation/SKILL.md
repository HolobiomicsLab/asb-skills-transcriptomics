---
name: prediction-accuracy-evaluation
description: Use when a trained cell type annotation model (e.g., TOSICA) has predicted
  labels on a query dataset and you need to quantify annotation quality as a fraction
  of correctly predicted cells against known query/test labels.
when_to_use_negative:
- The query dataset has no ground-truth cell type labels (accuracy cannot be computed;
  use interpretability outputs like attention weights or gene2token_weights.csv instead)
- The query AnnData var_names are not consistent with, and in the same order as, the
  reference — predictions would be invalid and accuracy meaningless until subsetting
  is done
- The goal is unsupervised clustering or de novo annotation rather than label transfer
  from a reference dataset
tools:
- name: TOSICA
  role: Annotation model whose predicted labels (new_adata.obs['Prediction']) are
    evaluated; train with TOSICA.train and predict with TOSICA.pre
  repo: https://github.com/JackieHanLab/TOSICA
- name: scanpy
  role: AnnData handling for reference/query datasets and subsetting query_adata to
    match ref_adata.var_names before prediction
- name: torch
  role: Backend (version 1.7.1) for training with SGD, cosine learning rate decay,
    and cross-entropy loss
- name: ACTINN
  role: Second-best benchmarked method in the 19-method comparison (79.57% on mAtlas)
    used as an accuracy reference point
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-35923-4
    title: Transformer for one stop interpretable cell type annotation
  source_content_hash: sha256:a782386192146c6f500652bd04334e4ef4b0e38d4c02cdc6e54c0bfb1a2734ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JackieHanLab/TOSICA
  tools:
  - ACTINN
  - Python
  - TOSICA
  - scanpy
  - torch
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# prediction-accuracy-evaluation

## Summary

Computes cell type annotation accuracy as the fraction of correctly predicted cells for TOSICA's reference-query label transfer, benchmarked against 18 other annotators across six datasets (hArtery, hBone, hPancreas, mBrain, mPancreas, mAtlas).

## When to use

Use when a trained cell type annotation model (e.g., TOSICA) has predicted labels on a query dataset and you need to quantify annotation quality as a fraction of correctly predicted cells against known query/test labels.

## When NOT to use

- The query dataset has no ground-truth cell type labels (accuracy cannot be computed; use interpretability outputs like attention weights or gene2token_weights.csv instead)
- The query AnnData var_names are not consistent with, and in the same order as, the reference — predictions would be invalid and accuracy meaningless until subsetting is done
- The goal is unsupervised clustering or de novo annotation rather than label transfer from a reference dataset

## Inputs

- Reference AnnData object (ref_adata) used for training
- Query AnnData object (query_adata) with held-out ground-truth cell type labels
- Trained TOSICA model weights (model-n.pth) and project folder (mask.npy, pathway.csv, label_dictionary.csv)
- Gene set mask definition (e.g., human_gobp from GO_bp.gmt or human_reactome from reactome.gmt)

## Outputs

- Fraction of correctly predicted cells (annotation accuracy) per dataset
- predictions.csv recording annotation accuracy on the query/test labels
- new_adata.obs['Prediction'] (predicted labels) and new_adata.obs['Probability']
- gene2token_weights.csv and attention matrix (new_adata.X)

## How to apply

Train TOSICA on a reference AnnData object with TOSICA.train (SGD optimizer with cosine learning rate decay, cross-entropy loss, 30% of the training set held out as validation), then subset the query with query_adata[:,ref_adata.var_names] so var_names match the reference exactly and in order. Run TOSICA.pre to obtain new_adata.obs['Prediction'] and new_adata.obs['Probability']. Compare the predicted labels against the held-out query/test ground-truth labels and compute the fraction of correctly predicted cells, recording it in predictions.csv. Because training and test sets are split by study or biological state (not randomly), the accuracy estimate reflects genuine cross-dataset generalization; correct application yields values on the order of the reported benchmarks (e.g., ~93.75% on hArtery, ~95.76% on hPancreas, 86.69% mean across 19 methods).

## Related tools

- **TOSICA** (Annotation model whose predicted labels (new_adata.obs['Prediction']) are evaluated; train with TOSICA.train and predict with TOSICA.pre) — https://github.com/JackieHanLab/TOSICA
- **scanpy** (AnnData handling for reference/query datasets and subsetting query_adata to match ref_adata.var_names before prediction)
- **torch** (Backend (version 1.7.1) for training with SGD, cosine learning rate decay, and cross-entropy loss)
- **ACTINN** (Second-best benchmarked method in the 19-method comparison (79.57% on mAtlas) used as an accuracy reference point)

## Examples

```
new_adata = TOSICA.pre(query_adata[:, ref_adata.var_names], model_weight_path='./my_project/model-n.pth', project='my_project'); acc = (new_adata.obs['Prediction'] == query_adata.obs['truth']).mean()
```

## Evaluation signals

- Accuracy computed as the fraction of predicted labels in new_adata.obs['Prediction'] that match the query/test ground-truth labels should be on the order of reported values (e.g., ~93.75% on hArtery, ~95.76% on hPancreas)
- Every predicted label in new_adata.obs['Prediction'] must be present in the label_dictionary.csv generated during TOSICA.train
- new_adata.obs['Probability'] values should be valid probabilities and correlate with per-cell correctness (higher probability where predictions are correct)
- Accuracy on difficult unbalanced datasets (hBone, mPancreas, mAtlas) should fall within the reported top-2 rank; on the largest dataset mAtlas expect ~81.06% (vs. 79.57% for ACTINN)
- Training/validation split must be by study or biological state with 30% of the training set as validation — accuracy measured on randomly split cells would overestimate performance

## Limitations

- Training/test splits use different studies or biological states, so accuracy depends on split choice; batch size, random seed, and learning rate values are not reported in the paper
- TOSICA accuracy depends on gene set masks; the number of attention heads (H) is not specified in the methods, complicating exact reproduction
- The reference and query AnnData var_names must be consistent and in the same order; mismatched genes silently invalidate predictions
- Benchmark metrics are limited to accuracy (fraction of correctly predicted cells); other quality dimensions (rare cell types, unbalanced classes) are only qualitatively addressed

## Evidence

- [results] TOSICA ranks in the top 6 on every dataset and achieves the highest mean accuracy of 86.69% among all 19 benchmarked methods: [evidence span withheld — anchor status: not_found]
- [results] accuracy metric definition and benchmark on mAtlas: [evidence span withheld — anchor status: not_found]
- [discussion] train/test split by study or biological state: [evidence span withheld — anchor status: not_found]
- [readme] predicted labels output field: "new_adata.obs['Prediction'] : Predicted labels"
- [readme] gene ordering requirement for valid predictions: "the `var_names` (genes) of the `ref_adata` and `query_adata` must be consistent and in the same order."
- [other] accuracy computation and recording step: [evidence span withheld — anchor status: not_found]
