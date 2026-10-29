---
name: ontology-embedding-usage
description: Use when you must annotate scRNA-seq cells to Cell Ontology terms that
  lack annotated training cells (unseen terms), leveraging the ontology graph rather
  than per-term supervised training alone.
when_to_use_negative:
- When every Cell Ontology term of interest already has annotated cells in the training
  set, so a purely supervised classifier suffices
- When the target cell types are topologically identical siblings in the Cell Ontology
  graph — OnClass cannot distinguish them, and the ontology-based propagation will
  not resolve the ambiguity
- When the vocabulary is uncontrolled free text and no mapping to Cell Ontology terms
  is available
tools:
- name: OnClass
  role: Implements Cell Ontology embedding and random walk with restart score propagation
    to classify cells, including unseen terms
  repo: https://github.com/wangshenguiuc/OnClass
- name: tensorflow 2.0
  role: Deep learning backend for the OnClass neural model
- name: Python 3.6
  role: Runtime environment required by OnClass (installed via pip)
- name: numpy
  role: Numeric handling and np.save of the propagated score matrix
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:09bf4a99252b80f2d412feb13483c2f9e98d6215496aa12291b22559d04e1920
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - Python 3.6
  - numpy
  - tensorflow 2.0
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# ontology-embedding-usage

## Summary

Embed Cell Ontology terms into a low-dimensional space and use the ontology graph structure (e.g., random walk with restart score propagation) to extend per-cell confidence scores from seen cell types to unseen Cell Ontology terms.

## When to use

Use when you must annotate scRNA-seq cells to Cell Ontology terms that lack annotated training cells (unseen terms), leveraging the ontology graph rather than per-term supervised training alone.

## When NOT to use

- When every Cell Ontology term of interest already has annotated cells in the training set, so a purely supervised classifier suffices
- When the target cell types are topologically identical siblings in the Cell Ontology graph — OnClass cannot distinguish them, and the ontology-based propagation will not resolve the ambiguity
- When the vocabulary is uncontrolled free text and no mapping to Cell Ontology terms is available

## Inputs

- Pretrained OnClass model file (model_path)
- Processed test feature matrix cor_test_feature and gene list cor_test_genes from ProcessTestFeature
- Cell Ontology structure (cell_type_nlp_emb_file and cell_type_network_file)
- Training labels (train_label) for EmbedCellTypes when training from scratch

## Outputs

- pred_Y_seen: cell-by-seen-cell-type score matrix
- pred_Y_all: cell-by-all-Cell-Ontology-term propagated score matrix (saved as pred_Y_all.npy)
- pred_label: integer per-cell predicted labels mapped to ontology IDs via i2co

## How to apply

Initialize an OnClassModel with cell_type_nlp_emb_file and cell_type_network_file, then rebuild the network from the pretrained checkpoint via BuildModel(ngene=None, use_pretrain=model_path). After predicting cell-by-seen-cell-type scores with OnClass_test_obj.Predict(cor_test_feature, test_genes=cor_test_genes, use_normalize=True) — which returns pred_Y_seen, pred_Y_all, and pred_label — overlay the confidence scores on the Cell Ontology graph and propagate them using random walk with restart so every Cell Ontology term receives a per-cell score. The ontology embedding step (EmbedCellTypes) projects the controlled vocabulary into the same low-dimensional space as single-cell transcriptomes, which is what allows scores for sibling/ancestor terms to be inferred from graph structure. Map integer labels back to ontology IDs with OnClass_test_obj.i2co and save the propagated matrix with np.save to pred_Y_all.npy.

## Related tools

- **OnClass** (Implements Cell Ontology embedding and random walk with restart score propagation to classify cells, including unseen terms) — https://github.com/wangshenguiuc/OnClass
- **tensorflow 2.0** (Deep learning backend for the OnClass neural model)
- **Python 3.6** (Runtime environment required by OnClass (installed via pip))
- **numpy** (Numeric handling and np.save of the propagated score matrix)

## Examples

```
OnClass_test_obj.Predict(cor_test_feature, test_genes=cor_test_genes, use_normalize=True)
```

## Evaluation signals

- pred_Y_all is a numeric matrix with one row per test cell and one column per all Cell Ontology term
- Columns corresponding to unseen (non-training) terms contain non-zero propagated scores
- Integer labels round-trip correctly through OnClass_test_obj.i2co to produce string ontology IDs
- Predictions for seen terms agree with a purely supervised baseline (pred_Y_seen vs pred_Y_all restricted to training terms)
- Pipeline runs under the pinned environment: Python 3.6, OnClass==1.2, tensorflow==2.0

## Limitations

- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states
- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- Methods to correct batch effects in the presence of many unseen cell types are not yet developed

## Evidence

- [abstract] random walk with restart enables unseen-term scoring: "A key advantage of OnClass is its capability to classify cells into cell types not present in the training data because it uses the Cell Ontology graph to infer cell type relationships."
- [discussion] third-step propagation description: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [methods] embedding step call: "OnClass_train_obj.EmbedCellTypes(train_label)"
- [intro] supervised methods fail on unseen terms, motivating graph use: "they are unable to classify cells to unseen terms (i.e., terms which do not have any annotated cells in the training data)"
