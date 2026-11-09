---
name: scrna-seq-cell-type-prediction
description: Use when scRNA-seq cells must be labeled against the Cell Ontology controlled
  vocabulary, especially when the query or test set may contain cell type terms absent
  from the training labels.
when_to_use_negative:
- Input is raw unprocessed count data that has not yet passed ProcessTrainFeature/ProcessTestFeature
  alignment to the pretrained model's gene space
- The goal is marker gene identification for ontology terms rather than per-cell annotation
  scores
- Free-text annotations must be mapped to Cell Ontology terms — that requires the
  NLP/Char-level LSTM mapping step, not the graph propagation step
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: OnClass
  role: Ontology-based classifier; its Predict step propagates seen-term scores over
    the Cell Ontology graph via random walk with restart to score unseen terms
  repo: https://github.com/wangshenguiuc/OnClass
- name: tensorflow
  role: Deep learning backend (tensorflow 2.0 / tensorflow-gpu 2.0) required to load
    and run the pretrained OnClass model
- name: numpy
  role: Numerical handling of the score matrices and saving pred_Y_all.npy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:aea5a28f7e0677b4a77f0337939b631bd4bb6e8181f7f3f3ceb43350dfeb0e3e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - numpy
  - tensorflow
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# scrna-seq-cell-type-prediction

## Summary

Predict cell type annotations for single-cell RNA-seq data with OnClass, propagating confidence scores over the Cell Ontology graph via random walk with restart so that all ontology terms—including unseen terms with no annotated training cells—receive per-cell scores.

## When to use

Use when scRNA-seq cells must be labeled against the Cell Ontology controlled vocabulary, especially when the query or test set may contain cell type terms absent from the training labels.

## When NOT to use

- Input is raw unprocessed count data that has not yet passed ProcessTrainFeature/ProcessTestFeature alignment to the pretrained model's gene space
- The goal is marker gene identification for ontology terms rather than per-cell annotation scores
- Free-text annotations must be mapped to Cell Ontology terms — that requires the NLP/Char-level LSTM mapping step, not the graph propagation step

## Inputs

- Pretrained OnClass model file (model_path) with checkpoint
- Processed test feature matrix (cor_test_feature) aligned to Cell Ontology training features
- Test gene list (cor_test_genes) matching the model's gene vocabulary
- cell_type_nlp_emb_file (Cell Ontology term NLP embeddings)
- cell_type_network_file (Cell Ontology graph structure)

## Outputs

- pred_Y_seen: cells x seen cell types confidence score matrix
- pred_Y_all: cells x all Cell Ontology terms propagated score matrix (pred_Y_all.npy)
- pred_label: integer predicted label per cell
- String label list obtained via OnClass_test_obj.i2co mapping to Cell Ontology term IDs

## How to apply

Load the pretrained OnClass model and the processed test feature matrix (cor_test_feature, cor_test_genes) produced by ProcessTestFeature. Initialize an OnClassModel with cell_type_nlp_emb_file and cell_type_network_file, then rebuild the network from the pretrained checkpoint via BuildModel(ngene=None, use_pretrain=model_path). Run OnClass_test_obj.Predict(cor_test_feature, test_genes=cor_test_genes, use_normalize=True), which returns pred_Y_seen (cell-by-seen-cell-type scores), pred_Y_all (cell-by-all-Cell-Ontology-term scores), and pred_label. In the third step, OnClass overlays the seen-term confidence scores on the Cell Ontology graph and propagates them with random walk with restart, so unseen terms inherit propagated scores. Map integer labels back to ontology IDs using OnClass_test_obj.i2co, and save the propagated matrix with np.save to pred_Y_all.npy.

## Related tools

- **OnClass** (Ontology-based classifier; its Predict step propagates seen-term scores over the Cell Ontology graph via random walk with restart to score unseen terms) — https://github.com/wangshenguiuc/OnClass
- **tensorflow** (Deep learning backend (tensorflow 2.0 / tensorflow-gpu 2.0) required to load and run the pretrained OnClass model)
- **numpy** (Numerical handling of the score matrices and saving pred_Y_all.npy)

## Examples

```
pred_Y_seen, pred_Y_all, pred_label = OnClass_test_obj.Predict(cor_test_feature, test_genes=cor_test_genes, use_normalize=True); np.save('pred_Y_all.npy', pred_Y_all)
```

## Evaluation signals

- pred_Y_all is a numeric matrix with exactly one row per test cell and one column per all Cell Ontology term
- Columns corresponding to unseen (non-training) terms contain non-zero propagated scores
- Integer pred_label values map to valid Cell Ontology term IDs via OnClass_test_obj.i2co
- pred_Y_seen column set matches only the cell types present in the training labels, while pred_Y_all strictly supersedes it
- Running with use_normalize=True yields per-cell score magnitudes comparable to a reference run on the same model checkpoint

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states, so terms absent from the ontology cannot be predicted
- Methods to correct batch effects in the presence of many unseen cell types are not yet developed

## Evidence

- [discussion] OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph.: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [abstract] A key advantage of OnClass is its capability to classify cells into cell types not present in the training data: "A key advantage of OnClass is its capability to classify cells into cell types not present in the training data because it uses the Cell Ontology graph to infer cell type relationships."
- [methods] cor_test_feature = OnClass_train_obj.ProcessTestFeature(cor_test_feature, cor_test_genes, use_pretrain = model_path, log_transform = False): "cor_test_feature = OnClass_train_obj.ProcessTestFeature(cor_test_feature, cor_test_genes, use_pretrain = model_path, log_transform = False)"
- [discussion] Another limitation of OnClass is that it cannot distinguish topologically identical unseen cell types: "Another limitation of OnClass is that it cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph."
- [readme] The algorithm is implemented in Python (https://github.com/wangshenguiuc/OnClass) and it has been included in PyPy: "OnClass is now upgraded to 1.2 version!"
