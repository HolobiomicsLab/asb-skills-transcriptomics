---
name: pretrained-neural-model-inference
description: Use when a test scRNA-seq expression matrix must be labeled against the
  Cell Ontology without retraining, especially when the query may contain cell types
  absent from any training annotations.
when_to_use_negative:
- You need to train a model on a new labeled dataset with custom annotations — use
  ProcessTrainFeature/Train instead of the pretrained checkpoint.
- Your analysis requires distinguishing topologically identical unseen cell types
  (siblings with the same parents in the Cell Ontology graph), which OnClass cannot
  separate.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: OnClass
  role: Provides the pre-trained neural model and Predict API that scores cells against
    Cell Ontology terms and propagates scores to unseen terms via random walk with
    restart
  repo: https://github.com/wangshenguiuc/OnClass
- name: tensorflow 2.0
  role: Runtime dependency for building and running the OnClass neural network from
    the pretrained checkpoint
- name: numpy
  role: Used to save the propagated all-term score matrix as pred_Y_all.npy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:22bb4dd31130e6175cd7394a7f64db98be81120cace4c0e1e3e677da30457319
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - numpy
  - tensorflow 2.0
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pretrained-neural-model-inference

## Summary

Apply OnClass's provided pre-trained neural model to score test single-cell transcriptomes against the Cell Ontology, propagating confidence scores via random walk with restart so all Cell Ontology terms—including unseen ones—receive predictions.

## When to use

Use when a test scRNA-seq expression matrix must be labeled against the Cell Ontology without retraining, especially when the query may contain cell types absent from any training annotations.

## When NOT to use

- You need to train a model on a new labeled dataset with custom annotations — use ProcessTrainFeature/Train instead of the pretrained checkpoint.
- Your analysis requires distinguishing topologically identical unseen cell types (siblings with the same parents in the Cell Ontology graph), which OnClass cannot separate.

## Inputs

- pretrained OnClass model file (model_path)
- processed test feature matrix cor_test_feature from ProcessTestFeature
- test gene list cor_test_genes
- Cell Ontology NLP embedding file (cell_type_nlp_emb_file)
- Cell Ontology network file (cell_type_network_file)

## Outputs

- pred_Y_seen: numeric matrix (cells x seen cell types) of confidence scores
- pred_Y_all: numeric matrix (cells x all Cell Ontology terms) with propagated scores for unseen terms, saved as pred_Y_all.npy
- pred_label: per-cell integer labels, mapped to Cell Ontology term IDs via OnClass_test_obj.i2co

## How to apply

Load the pretrained OnClass model file together with the processed test feature matrix (cor_test_feature, cor_test_genes) produced by upstream ProcessTestFeature. Initialize an OnClassModel with cell_type_nlp_emb_file and cell_type_network_file, then rebuild the network from the pretrained checkpoint via BuildModel(ngene=None, use_pretrain=model_path) rather than calling Train. Run OnClass_test_obj.Predict(cor_test_feature, test_genes=cor_test_genes, use_normalize=True) to obtain pred_Y_seen (cell-by-seen-cell-type scores), pred_Y_all (cell-by-all-Cell-Ontology-term scores), and pred_label. The third step overlays confidence scores on the Cell Ontology graph and propagates them with random walk with restart, so unseen terms receive non-zero scores; map integer labels back to ontology IDs with OnClass_test_obj.i2co and save the propagated matrix with np.save to pred_Y_all.npy.

## Related tools

- **OnClass** (Provides the pre-trained neural model and Predict API that scores cells against Cell Ontology terms and propagates scores to unseen terms via random walk with restart) — https://github.com/wangshenguiuc/OnClass
- **tensorflow 2.0** (Runtime dependency for building and running the OnClass neural network from the pretrained checkpoint)
- **numpy** (Used to save the propagated all-term score matrix as pred_Y_all.npy)

## Examples

```
OnClass_test_obj = OnClassModel(cell_type_nlp_emb_file, cell_type_network_file); OnClass_test_obj.BuildModel(ngene=None, use_pretrain=model_path); pred_Y_seen, pred_Y_all, pred_label = OnClass_test_obj.Predict(cor_test_feature, test_genes=cor_test_genes, use_normalize=True); np.save('pred_Y_all.npy', pred_Y_all)
```

## Evaluation signals

- pred_Y_all is numeric with one row per test cell and one column per Cell Ontology term
- Columns corresponding to unseen (non-training) Cell Ontology terms contain non-zero propagated scores
- pred_label integers map successfully back to Cell Ontology term IDs through OnClass_test_obj.i2co
- pred_Y_seen has exactly one column per seen (training) cell type
- pred_Y_all.npy is written and loadable via np.load with consistent dimensions

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph.
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states, limiting the terms that can be predicted.
- Environment constraints apply: OnClass 1.2 requires Python 3.6 and tensorflow 2.0.

## Evidence

- [discussion] Random walk propagation over Cell Ontology: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [discussion] Pretrained model availability: "as part of the package we provided a pre-trained model that can output cell type annotations for millions of cells in a few minutes on a modern server."
- [abstract] Unseen term classification: "A key advantage of OnClass is its capability to classify cells into cell types not present in the training data because it uses the Cell Ontology graph to infer cell type relationships."
- [methods] Installation requirements: "pip install OnClass==1.2"
- [discussion] Topological sibling limitation: "Another limitation of OnClass is that it cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph."
