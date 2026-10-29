---
name: cell-ontology-graph-score-propagation
description: Use when the query dataset may contain cell types absent from the training
  reference and scores must be produced for all Cell Ontology terms, not just seen
  training labels.
when_to_use_negative:
- All query cell types have annotated cells in the training set, so extending scores
  to unseen ontology terms is unnecessary
- Topologically identical unseen cell types (siblings with the same parents in the
  Cell Ontology graph) must be distinguished — OnClass cannot separate them
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0621
tools:
- name: OnClass
  role: Performs random walk with restart score propagation over the Cell Ontology
    graph to extend seen-cell-type scores to unseen terms
  repo: https://github.com/wangshenguiuc/OnClass
- name: Python
  role: Implementation language for running OnClass and saving pred_Y_all.npy with
    numpy
  repo: https://github.com/wangshenguiuc/OnClass
- name: tensorflow
  role: Deep learning backend required to load and run the pretrained OnClass model
- name: numpy
  role: Saves the propagated pred_Y_all score matrix to pred_Y_all.npy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:3b32e2cc21c9c8fad3023f1842123649861cda2778ee92ef934df193ec0b51db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - numpy
  - tensorflow
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cell-ontology-graph-score-propagation

## Summary

Refines per-cell annotation scores by overlaying confidence scores on the Cell Ontology graph and propagating them via random walk with restart (OnClass step 3), so every Cell Ontology term — including unseen terms with no annotated training cells — receives a confidence score.

## When to use

Use when the query dataset may contain cell types absent from the training reference and scores must be produced for all Cell Ontology terms, not just seen training labels.

## When NOT to use

- All query cell types have annotated cells in the training set, so extending scores to unseen ontology terms is unnecessary
- Topologically identical unseen cell types (siblings with the same parents in the Cell Ontology graph) must be distinguished — OnClass cannot separate them

## Inputs

- Pretrained OnClass model file (model_path)
- Processed test feature matrix cor_test_feature from ProcessTestFeature
- Test gene list cor_test_genes
- cell_type_nlp_emb_file and cell_type_network_file (Cell Ontology graph/embedding files)

## Outputs

- pred_Y_seen: cell-by-seen-cell-type score matrix
- pred_Y_all: cell-by-all-Cell-Ontology-term score matrix with propagated scores for unseen terms
- pred_label: integer predicted labels per cell
- String label list obtained via OnClass_test_obj.i2co
- pred_Y_all.npy saved via np.save

## How to apply

Initialize an OnClassModel with cell_type_nlp_emb_file and cell_type_network_file, then rebuild the network from the pretrained checkpoint via BuildModel(ngene=None, use_pretrain=model_path). Run Predict(cor_test_feature, test_genes=cor_test_genes, use_normalize=True) to obtain pred_Y_seen (cell-by-seen-cell-type scores), pred_Y_all (cell-by-all-Cell-Ontology-term scores), and pred_label. In the third step, OnClass overlays the seen-cell-type confidence scores on the Cell Ontology graph and propagates them with random walk with restart, extending scores to unseen cell types. Map integer labels back to ontology IDs with OnClass_test_obj.i2co and save the propagated matrix with np.save to pred_Y_all.npy.

## Related tools

- **OnClass** (Performs random walk with restart score propagation over the Cell Ontology graph to extend seen-cell-type scores to unseen terms) — https://github.com/wangshenguiuc/OnClass
- **Python** (Implementation language for running OnClass and saving pred_Y_all.npy with numpy) — https://github.com/wangshenguiuc/OnClass
- **tensorflow** (Deep learning backend required to load and run the pretrained OnClass model)
- **numpy** (Saves the propagated pred_Y_all score matrix to pred_Y_all.npy)

## Examples

```
OnClass_test_obj.Predict(cor_test_feature, test_genes=cor_test_genes, use_normalize=True); np.save('pred_Y_all.npy', pred_Y_all)
```

## Evaluation signals

- pred_Y_all is a numeric matrix with one row per test cell and one column per all Cell Ontology term
- Columns corresponding to unseen (non-training) terms contain non-zero propagated scores
- pred_label integers map back to valid ontology IDs via OnClass_test_obj.i2co
- pred_Y_all width equals the full Cell Ontology term count, exceeding pred_Y_seen width (seen terms only)

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states
- Batch-effect correction methods in the presence of many unseen cell types are not yet developed

## Evidence

- [discussion] random walk with restart propagation: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [abstract] unseen-term classification capability: "A key advantage of OnClass is its capability to classify cells into cell types not present in the training data because it uses the Cell Ontology graph to infer cell type relationships."
- [methods] installation stack: "pip install OnClass==1.2"
- [discussion] pretrained model availability: "as part of the package we provided a pre-trained model that can output cell type annotations for millions of cells in a few minutes on a modern server."
- [discussion] sibling limitation: "Another limitation of OnClass is that it cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph."
