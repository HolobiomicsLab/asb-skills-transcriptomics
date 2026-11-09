---
name: donor-holdout-generalization-testing
description: Use when a cell type annotation model trained on CELLxGENE census data
  must be tested for generalization to held-out donors, or when comparing performance
  across label granularity or sequencing protocols.
when_to_use_negative:
- You only need to rank candidate labels on the training distribution — donor holdout
  tests cross-donor generalization, not in-distribution ranking quality
- Cell types appear in fewer than 30 donors in the corpus, so cross-donor generalization
  cannot be reliably quantified (paper filter step 5)
- Your data is not single-cell RNA-seq with author cell type labels mappable to the
  Cell Ontology; the subtype-aware correctness rule is undefined otherwise
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scTab
  role: Cell type prediction model whose trained checkpoint is evaluated on the donor-held-out
    test split
  repo: https://github.com/theislab/scTab
- name: TabNet
  role: Tabular architecture behind the scTab classifier; inference produces class
    probabilities and predicted labels
  repo: https://github.com/theislab/scTab
- name: NumPy
  role: Provides the sf_normalize routine used to size-factor normalize cells to 10000
    counts before inference
- name: cellxgene-census
  role: Standardized API used to query and download the CELLxGENE census (v2023-05-15)
    from which the donor split is derived
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-51059-5
    title: 'scTab: Scaling cross-tissue single-cell annotation models'
  source_content_hash: sha256:5ae195b600af79027923a3ebae98d6cb1aa4aaafd3fecbaee03ae976d87b45e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scTab
  tools:
  - NumPy
  - TabNet
  - cellxgene-census
  - scTab
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# donor-holdout-generalization-testing

## Summary

Evaluates a cross-tissue cell type classifier (scTab/TabNet) on a donor-held-out test split so that reported macro F1-scores reflect generalization to unseen donors rather than to re-measured cells. Uses Cell Ontology-aware correctness rules and imbalance-aware macro F1 at multiple granularities.

## When to use

Use when a cell type annotation model trained on CELLxGENE census data must be tested for generalization to held-out donors, or when comparing performance across label granularity or sequencing protocols.

## When NOT to use

- You only need to rank candidate labels on the training distribution — donor holdout tests cross-donor generalization, not in-distribution ranking quality
- Cell types appear in fewer than 30 donors in the corpus, so cross-donor generalization cannot be reliably quantified (paper filter step 5)
- Your data is not single-cell RNA-seq with author cell type labels mappable to the Cell Ontology; the subtype-aware correctness rule is undefined otherwise

## Inputs

- Trained scTab/TabNet classifier checkpoint (cellnet/models.py TabnetClassifier)
- Donor-holdout test split of the CELLxGENE census v2023-05-15 (~3.45M cells, 15% of 22.19M)
- Child node lookup matrix from 04_create_hierarchy_matrices.ipynb built with the Cell Ontology (CL)
- Assay/protocol annotation per cell (10x 3'/5', STRT-seq, Smart-seq2, BD Rhapsody Targeted mRNA)
- Information content scores for the 164 fine-grained cell type labels

## Outputs

- Macro F1-score on the donor-held-out test set (e.g., 0.7864±0.0010 for 1.7M-param model, 0.8323±0.0010 for 16.2M-param model)
- Coarse (31 labels) vs. fine (164 labels) macro F1 comparison, e.g., 0.897 coarse vs. 0.830 fine (Supp. Fig. 7a)
- Per-protocol macro F1 scores, e.g., ~0.8 on holdout 10X vs. ~0.4 on non-10X protocols (protocol_macro_f1_scores.csv)
- Comparison table written to coarse_vs_fine_f1

## How to apply

Split the 22.19M-cell CELLxGENE census (v2023-05-15, 19331 protein coding genes, 164 cell types) by donor into train/val/test so no donor appears in more than one split; the test fraction is 0.15 (~3.45 million cells). Preprocess test cells identically to training: sf_normalize each cell to 10000 counts, apply log1p, restrict to the 19331 protein coding gene space. Run inference with the trained TabNet classifier, then apply the ontology-aware correctness rule from cellnet/utils/cell_ontology.py: a prediction counts as correct if it equals the true label or is a subtype of it (via the child node lookup matrix from 04_create_hierarchy_matrices.ipynb); predicting a parent type or anything else counts as wrong. Compute per-cell-type F1 and average equally across cell types (macro F1) to counter strong class imbalance; for granularity analyses, map the 164 fine labels to 31 coarse labels by selecting labels with information content score ≤ 60 and compute macro F1 at each level separately. For protocol OOD evaluation, group test cells by assay (10x 3'/5' vs. STRT-seq, Smart-seq2, BD Rhapsody Targeted mRNA) and compute macro F1 per group.

## Related tools

- **scTab** (Cell type prediction model whose trained checkpoint is evaluated on the donor-held-out test split) — https://github.com/theislab/scTab
- **TabNet** (Tabular architecture behind the scTab classifier; inference produces class probabilities and predicted labels) — https://github.com/theislab/scTab
- **NumPy** (Provides the sf_normalize routine used to size-factor normalize cells to 10000 counts before inference)
- **cellxgene-census** (Standardized API used to query and download the CELLxGENE census (v2023-05-15) from which the donor split is derived)

## Examples

```
preds = model.predict(X_test_sf_log1p); correct = preds == y_true | child_lookup[np.arange(len(y_true)), preds]; f1 = (f1_score(y_true, preds, average='macro'))
```

## Evaluation signals

- Computed fine-grained macro F1 on the donor-held-out test set falls within reported uncertainty: 0.7864±0.0010 (1.7M params) and 0.8323±0.0010 (16.2M params)
- Coarse-label macro F1 (0.897) exceeds fine-label macro F1 (0.830), matching Supp. Fig. 7a within reasonable tolerance
- Holdout 10X protocol macro F1 is around 0.8 while non-10X protocol groups are around 0.4, consistent with Supp. Fig. 7b
- No donor overlaps between train and test cells (guaranteed by the donor-based split using is_primary_data == True to prevent label leakage)
- Macro F1 is computed by averaging per-cell-type F1 with equal weight, not by pooling all cells (to handle class imbalance)

## Limitations

- Cell Ontology is incomplete: not all cell types (especially rare ones) can be matched to an ontology term, and cell type relationships are still under active discussion, which affects classification metrics
- Plain macro F1 may not reflect dataset complexity across imbalanced cell types, tissues, and sequencing protocols; more detailed evaluation metrics are needed
- Generalizes poorly to non-10X protocols (STRT-seq, Smart-seq2, BD Rhapsody Targeted mRNA, macro F1 ~0.4) because the model was trained exclusively on 10X-based scRNA-seq data
- Pre-trained scTab models still contain fairly coarsely annotated cell type labels despite filtering of too-coarsely annotated labels

## Evidence

- [methods] Split data into train, val and test sets (donor-based split): "2. `02_create_train_val_test_split.ipynb`: Split data into train, val and test sets"
- [methods] Each cell type has to be observed in at least 30 donors to reliably quantify whether the classifier can generalize to new donors.: "5. Each cell type has to be observed in at least `30` donors to reliably quantify whether the classifier can 
  generalize to new donors."
- [other] Counting a prediction as correct when the classifier predicts a subtype of the true cell type label.: [evidence span withheld — anchor status: not_found]
- [methods] Data has to be primary data to prevent label leakage between the train and test set: "1. Data has to be primary data to prevent label leakage between the train and test set: `is_primary_data == True`"
- [methods] Normalization each cell to have 10000 counts then Log1p transformed: "1. Normalize each cell to have `10000` counts (size factor normalization)"
