---
name: bilinear-neural-network-training
description: Use when training an OnClass model on annotated scRNA-seq data (h5ad)
  with Cell Ontology-mapped labels, to obtain a saved model file that supports seen-term
  classification and unseen Cell Ontology term prediction.
when_to_use_negative:
- You already have a saved/pretrained OnClass model and only need annotations — use
  ProcessTestFeature with use_pretrain rather than retraining (OnClass ships a pre-trained
  model that annotates millions of cells in minutes).
- The target cell types are topologically identical unseen siblings (same parents
  in the Cell Ontology graph) — OnClass explicitly cannot distinguish them.
- The train/test data have strong batch effects together with a large number of unseen
  cell types — batch-correction methods for this case are not yet developed.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: OnClass
  role: Provides EmbedCellTypes, ProcessTrainFeature, BuildModel, Train, ProcessTestFeature
    and Predict — the bilinear-network training pipeline itself (pip install OnClass==1.2)
  repo: https://github.com/wangshenguiuc/OnClass
- name: tensorflow
  role: Deep-learning backend used to build and train the OnClass bilinear neural
    network (pip install tensorflow==2.0, with tensorflow-gpu==2.0 optional)
- name: Scanorama
  role: Optional batch correction between train and test gene panels before model
    construction and training
  repo: https://github.com/brianhie/scanorama
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:bd234aa1a3f933320f6cde8430d04da54e5931584422be2eb1c6a2e75f074542
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - Scanorama
  - pip
  - tensorflow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6470588235294118
n_claims: 17
---

# bilinear-neural-network-training

## Summary

Train OnClass's bilinear neural network, which projects single-cell transcriptomes and Cell Ontology terms into the same low-dimensional space so that cells can be scored against all ontology terms, including unseen ones. This is the model-building and training stage of the OnClass scRNA-seq annotation workflow.

## When to use

Use when training an OnClass model on annotated scRNA-seq data (h5ad) with Cell Ontology-mapped labels, to obtain a saved model file that supports seen-term classification and unseen Cell Ontology term prediction.

## When NOT to use

- You already have a saved/pretrained OnClass model and only need annotations — use ProcessTestFeature with use_pretrain rather than retraining (OnClass ships a pre-trained model that annotates millions of cells in minutes).
- The target cell types are topologically identical unseen siblings (same parents in the Cell Ontology graph) — OnClass explicitly cannot distinguish them.
- The train/test data have strong batch effects together with a large number of unseen cell types — batch-correction methods for this case are not yet developed.

## Inputs

- Annotated training gene expression matrix (h5ad) with labels mapped to Cell Ontology IDs
- Cell Ontology files from OnClass_data_public_minimal.tar.gz: cl.obo, cell type network, cl.ontology.nlp.emb embeddings
- Test gene expression matrix (h5ad) and its gene list for panel alignment
- Optional Scanorama-corrected train/test features for cross-dataset gene panels

## Outputs

- Saved OnClass model file (model_path) loadable via ProcessTestFeature(use_pretrain=...)
- Processed features and corrected gene lists (cor_train_feature, cor_test_feature, cor_train_genes, cor_test_genes)
- Predictions pred_Y_seen, pred_Y_all, and pred_label over all Cell Ontology terms
- Seen-term AUPRC and unseen-term AUROC metrics (e.g., written to cross_dataset_prediction_results.csv)

## How to apply

First embed the Cell Ontology into its low-dimensional space with OnClass_train_obj.EmbedCellTypes(train_label), using the cl.obo / cell-type-network / cl.ontology.nlp.emb files from OnClass_data_public_minimal.tar.gz and the NLP Char-level LSTM mapping (nlp_mapping=True) to convert uncontrolled free-text labels to Cell Ontology IDs. Call ProcessTrainFeature(train_feature, train_label, train_genes, ...) to align the gene panels and return cor_train_feature, cor_test_feature, cor_train_genes, cor_test_genes; when train and test datasets differ (e.g., cross-dataset Lemur/Muris panels), optionally apply Scanorama batch correction before this step. Build the bilinear network with BuildModel(ngene=len(cor_train_genes), nhidden=NHIDDEN), sizing the input to the corrected gene set, then train with Train(cor_train_feature, train_label, save_model=model_path, max_iter=MAX_ITER), setting max_iter high enough for convergence and saving the model file so it can be reused via ProcessTestFeature(use_pretrain=model_path, log_transform=False). At prediction time, rebuild the model from the pretrained file and call Predict(test_genes=cor_test_genes, use_normalize=True) to obtain pred_Y_seen, pred_Y_all, and pred_label covering all Cell Ontology terms; the ontology graph extends scores to unseen cell types.

## Related tools

- **OnClass** (Provides EmbedCellTypes, ProcessTrainFeature, BuildModel, Train, ProcessTestFeature and Predict — the bilinear-network training pipeline itself (pip install OnClass==1.2)) — https://github.com/wangshenguiuc/OnClass
- **tensorflow** (Deep-learning backend used to build and train the OnClass bilinear neural network (pip install tensorflow==2.0, with tensorflow-gpu==2.0 optional))
- **Scanorama** (Optional batch correction between train and test gene panels before model construction and training) — https://github.com/brianhie/scanorama

## Examples

```
OnClass_train_obj.BuildModel(ngene=len(cor_train_genes), nhidden=NHIDDEN); OnClass_train_obj.Train(cor_train_feature, train_label, save_model=model_path, max_iter=MAX_ITER)
```

## Evaluation signals

- Training runs to completion for max_iter=MAX_ITER iterations and writes the model file to model_path without NaN/failed saves.
- ProcessTestFeature(use_pretrain=model_path) successfully rebuilds the model from the saved pretrained file.
- Predict returns pred_Y_seen, pred_Y_all, and pred_label covering all Cell Ontology terms, not just training labels.
- On the Lemur 4 → Lemur 1 cross-dataset setting, seen-term AUPRC reproduces ≈0.79.
- On the Muris FACS → Lemur 2 cross-species setting (67% unseen cell types), unseen-term AUROC reproduces ≈0.94.

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph.
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states.
- Methods to correct batch effects in the presence of many unseen cell types are not yet developed.
- Python 3.6, OnClass==1.2 and tensorflow 2.0 are the documented working versions.

## Evidence

- [methods] Bilinear model construction: "OnClass_train_obj.BuildModel(ngene = len(cor_train_genes), nhidden = NHIDDEN)"
- [methods] Training with model save: "OnClass_train_obj.Train(cor_train_feature, train_label, save_model = model_path, max_iter = MAX_ITER)"
- [methods] Ontology embedding before training: "OnClass_train_obj.EmbedCellTypes(train_label)"
- [methods] Test-feature processing from pretrained model: "cor_test_feature = OnClass_train_obj.ProcessTestFeature(cor_test_feature, cor_test_genes, use_pretrain = model_path, log_transform = False)"
- [intro] Rationale: shared low-dimensional space: "by projecting single-cell transcriptomes and the Cell Ontology into the same low-dimensional space, OnClass advances other important applications, such as robust computation of marker genes"
- [discussion] Ontology graph extends scores to unseen terms: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [discussion] Sibling indistinguishability limitation: "Another limitation of OnClass is that it cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph."
