---
name: unseen-cell-type-annotation
description: Use when the query scRNA-seq data may contain cell types absent from
  the reference training set and every Cell Ontology term — including unseen ones
  — needs a per-cell confidence score.
when_to_use_negative:
- When all cell types of interest are already annotated in the training set and propagation
  to unseen terms is unnecessary (the ontology step adds overhead).
- When you must distinguish topologically identical unseen cell types — OnClass cannot
  distinguish siblings that share the same parents in the Cell Ontology graph.
- When the query cell states are not represented in the Cell Ontology at all, since
  the ontology was not developed specifically for scRNA-seq and likely misses cell
  types and cell states.
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0621
tools:
- name: OnClass
  role: 'Core algorithm: predicts seen-term scores and propagates them over the Cell
    Ontology graph via random walk with restart to score unseen terms; installed via
    pip install OnClass==1.2.'
  repo: https://github.com/wangshenguiuc/OnClass
- name: tensorflow
  role: Deep-learning backend for the OnClass model (pip install tensorflow==2.0 /
    tensorflow-gpu==2.0).
- name: Python
  role: Implementation and scripting environment for OnClassModel initialization,
    Predict, label-to-ontology-ID mapping, and np.save of pred_Y_all.npy.
  repo: https://github.com/wangshenguiuc/OnClass
- name: numpy
  role: Saves the propagated cell-by-all-terms matrix via np.save(pred_Y_all.npy).
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:6e8516e9456a6e1f1550aa23c33384b9365ac1bae1081ec4aed50bbfce9bc8a7
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
yes_rate: 0.5555555555555556
n_claims: 9
---

# unseen-cell-type-annotation

## Summary

Annotate single cells to Cell Ontology terms that lack annotated cells in the training data by propagating per-cell confidence scores over the Cell Ontology graph with random walk with restart, as implemented in OnClass's third pipeline step.

## When to use

Use when the query scRNA-seq data may contain cell types absent from the reference training set and every Cell Ontology term — including unseen ones — needs a per-cell confidence score.

## When NOT to use

- When all cell types of interest are already annotated in the training set and propagation to unseen terms is unnecessary (the ontology step adds overhead).
- When you must distinguish topologically identical unseen cell types — OnClass cannot distinguish siblings that share the same parents in the Cell Ontology graph.
- When the query cell states are not represented in the Cell Ontology at all, since the ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states.

## Inputs

- Pretrained OnClass model file (model_path)
- Processed test feature matrix cor_test_feature and gene list cor_test_genes from ProcessTestFeature
- cell_type_nlp_emb_file (NLP embeddings of Cell Ontology terms)
- cell_type_network_file (Cell Ontology graph structure)

## Outputs

- pred_Y_seen: numeric cell-by-seen-cell-type score matrix
- pred_Y_all: numeric cell-by-all-Cell-Ontology-term score matrix (saved as pred_Y_all.npy), with non-zero propagated scores for unseen terms
- pred_label: integer predicted labels per cell
- String label list obtained by mapping integer labels through OnClass_test_obj.i2co to Cell Ontology IDs

## How to apply

First, load the pretrained OnClass model file and the processed test feature matrix (cor_test_feature, cor_test_genes) produced by upstream ProcessTestFeature. Initialize an OnClassModel with cell_type_nlp_emb_file and cell_type_network_file, then rebuild the network from the pretrained checkpoint via BuildModel(ngene=None, use_pretrain=model_path). Run OnClass_test_obj.Predict(cor_test_feature, test_genes=cor_test_genes, use_normalize=True), which returns pred_Y_seen (cell-by-seen-cell-type scores), pred_Y_all (cell-by-all-Cell-Ontology-term scores), and pred_label. The third step overlays the seen-cell-type confidence scores on the Cell Ontology graph and propagates them by random walk with restart, using the ontology's hierarchical structure to infer relationships and assign non-zero scores to unseen terms. Finally, map integer labels back to ontology IDs with OnClass_test_obj.i2co and save the propagated matrix with np.save to pred_Y_all.npy. The rationale is that the Cell Ontology graph encodes cell type relationships, so scores flow from seen terms to topologically related unseen terms.

## Related tools

- **OnClass** (Core algorithm: predicts seen-term scores and propagates them over the Cell Ontology graph via random walk with restart to score unseen terms; installed via pip install OnClass==1.2.) — https://github.com/wangshenguiuc/OnClass
- **tensorflow** (Deep-learning backend for the OnClass model (pip install tensorflow==2.0 / tensorflow-gpu==2.0).)
- **Python** (Implementation and scripting environment for OnClassModel initialization, Predict, label-to-ontology-ID mapping, and np.save of pred_Y_all.npy.) — https://github.com/wangshenguiuc/OnClass
- **numpy** (Saves the propagated cell-by-all-terms matrix via np.save(pred_Y_all.npy).)

## Examples

```
OnClass_test_obj.BuildModel(ngene=None, use_pretrain=model_path); pred_Y_seen, pred_Y_all, pred_label = OnClass_test_obj.Predict(cor_test_feature, test_genes=cor_test_genes, use_normalize=True); np.save('pred_Y_all.npy', pred_Y_all)
```

## Evaluation signals

- pred_Y_all is a numeric matrix with one row per test cell and one column per all Cell Ontology terms.
- Columns corresponding to unseen (non-training) terms contain non-zero propagated scores.
- pred_label integers map back to valid Cell Ontology IDs via OnClass_test_obj.i2co.
- pred_Y_seen has exactly as many columns as seen (training) cell types.
- Predict is called with use_normalize=True and the model rebuilt from the checkpoint via BuildModel(ngene=None, use_pretrain=model_path), matching the pretrained-model workflow.

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph.
- The Cell Ontology was not developed specifically for scRNA-seq and therefore likely misses cell types and cell states.
- Batch-effect correction methods in the presence of many unseen cell types are not yet developed.

## Evidence

- [discussion] Third-step random walk propagation: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [abstract] Capability to classify unseen terms: "A key advantage of OnClass is its capability to classify cells into cell types not present in the training data because it uses the Cell Ontology graph to infer cell type relationships."
- [methods] Installation and Python versions: "Please use Python3.6. To install OnClass using pip, run:"
- [discussion] Pretrained model availability: "as part of the package we provided a pre-trained model that can output cell type annotations for millions of cells in a few minutes on a modern server."
- [discussion] Sibling indistinguishability limitation: "Another limitation of OnClass is that it cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph."
