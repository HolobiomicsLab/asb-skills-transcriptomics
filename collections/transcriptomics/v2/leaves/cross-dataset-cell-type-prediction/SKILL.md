---
name: cross-dataset-cell-type-prediction
description: Use when a labeled training scRNA-seq dataset (h5ad) must annotate cells
  from a different dataset — same species or cross-species — where the test set contains
  cell types with no annotated cells in the training data.
when_to_use_negative:
- Training and test cells come from the same dataset and all labels are seen — a standard
  single-dataset classifier such as ACTINN or singleCellNet cross-validation suffices
- The goal is marker gene identification or ontology refinement rather than annotating
  a new dataset
- Free-text cell labels cannot be mapped to Cell Ontology terms and no controlled-vocabulary
  annotations exist
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0089
tools:
- name: OnClass
  role: 'Core algorithm: ontology-embedded bilinear neural network that classifies
    cells into Cell Ontology terms including unseen ones; installed via pip install
    OnClass==1.2'
  repo: https://github.com/wangshenguiuc/OnClass
- name: Scanorama
  role: Optional batch correction of train and test gene panels before prediction
    when Run_scanorama_batch_correction is enabled
  repo: https://github.com/brianhie/scanorama
- name: tensorflow 2.0
  role: Backend for building and training the OnClass bilinear neural network (BuildModel/Train)
- name: Python
  role: Implementation language for the cross-dataset prediction workflow (run_cross_dataset_prediction.py)
  repo: https://github.com/wangshenguiuc/OnClass
- name: pip
  role: Package installer used to install OnClass 1.2 and tensorflow 2.0
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:772ff632b50040a78b3ecc2442054b2fff82d59b80d23cc4d25b86a466fc651d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - Scanorama
  - pip
  - tensorflow 2.0
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-dataset-cell-type-prediction

## Summary

Train an OnClass bilinear neural network on one annotated scRNA-seq dataset (e.g., Lemur 4) and predict Cell Ontology cell type labels on a different dataset (e.g., Lemur 1 or Muris FACS to Lemur 2), including unseen cell types, using the Cell Ontology graph to extend scores to terms absent from the training labels.

## When to use

Use when a labeled training scRNA-seq dataset (h5ad) must annotate cells from a different dataset — same species or cross-species — where the test set contains cell types with no annotated cells in the training data.

## When NOT to use

- Training and test cells come from the same dataset and all labels are seen — a standard single-dataset classifier such as ACTINN or singleCellNet cross-validation suffices
- The goal is marker gene identification or ontology refinement rather than annotating a new dataset
- Free-text cell labels cannot be mapped to Cell Ontology terms and no controlled-vocabulary annotations exist

## Inputs

- Annotated training gene expression matrix (h5ad) from the source dataset (e.g., Lemur 4 or Muris FACS)
- Test gene expression matrix (h5ad) from the target dataset (e.g., Lemur 1 or Lemur 2)
- Cell Ontology files: cl.obo, cell type network, and cl.ontology.nlp.emb embeddings from OnClass_data_public_minimal.tar.gz
- Optional Scanorama-corrected train/test gene panel features

## Outputs

- pred_Y_seen, pred_Y_all, and pred_label covering all Cell Ontology terms
- Seen-term AUPRC and unseen-term AUROC metrics saved to cross_dataset_prediction_results.csv
- Trained OnClass model file saved at model_path

## How to apply

Load the annotated train/test h5ad datasets with anndata and the Cell Ontology resources (cl.obo, cell type network, cl.ontology.nlp.emb) from OnClass_data_public_minimal.tar.gz, setting nlp_mapping as needed to map free-text labels to Cell Ontology IDs. Embed the ontology with OnClassModel.EmbedCellTypes(train_label) to project cell types into the ontology-informed low-dimensional space; optionally apply Scanorama batch correction between train and test gene panels when Run_scanorama_batch_correction is enabled. Process features with ProcessTrainFeature, build the bilinear network with BuildModel(ngene=len(cor_train_genes), nhidden=NHIDDEN), and train with Train(save_model=model_path, max_iter=MAX_ITER) on Python 3.6 with tensorflow 2.0 and OnClass 1.2. Process the test data via ProcessTestFeature(use_pretrain=model_path, log_transform=False), rebuild the model from the pretrained file, and call Predict(test_genes=cor_test_genes, use_normalize=True) to obtain pred_Y_seen, pred_Y_all, and pred_label covering all Cell Ontology terms; finally evaluate seen-term AUPRC and unseen-term AUROC and save predictions to cross_dataset_prediction_results.csv.

## Related tools

- **OnClass** (Core algorithm: ontology-embedded bilinear neural network that classifies cells into Cell Ontology terms including unseen ones; installed via pip install OnClass==1.2) — https://github.com/wangshenguiuc/OnClass
- **Scanorama** (Optional batch correction of train and test gene panels before prediction when Run_scanorama_batch_correction is enabled) — https://github.com/brianhie/scanorama
- **tensorflow 2.0** (Backend for building and training the OnClass bilinear neural network (BuildModel/Train))
- **Python** (Implementation language for the cross-dataset prediction workflow (run_cross_dataset_prediction.py)) — https://github.com/wangshenguiuc/OnClass
- **pip** (Package installer used to install OnClass 1.2 and tensorflow 2.0)

## Examples

```
python script/run_cross_dataset_prediction.py
```

## Evaluation signals

- Seen-term AUPRC on held-out cross-dataset test data should reach approximately 0.79 when training on Lemur 4 and testing on Lemur 1
- Unseen cell type AUROC should reach approximately 0.94 in the cross-species Muris FACS to Lemur 2 setting, which has 67% unseen cell types
- pred_label must cover all Cell Ontology terms, not only training-set terms, confirming ontology-based score extension to unseen types
- Metrics and predictions are written to cross_dataset_prediction_results.csv and should match reported values
- The trained model file saved by Train(save_model=model_path) must load successfully via ProcessTestFeature(use_pretrain=model_path)

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states
- Methods to correct batch effects in the presence of many unseen cell types are not yet developed; Scanorama correction is only optional and preliminary

## Evidence

- [methods] Cross-dataset benchmark performance: "This script trains an OnClass model on all cells from one Lemur dataset, saves that model to a model file"
- [discussion] Score extension to unseen types: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [methods] Training pipeline steps: "OnClass_train_obj.BuildModel(ngene = len(cor_train_genes), nhidden = NHIDDEN)"
- [methods] Test feature processing: "cor_test_feature = OnClass_train_obj.ProcessTestFeature(cor_test_feature, cor_test_genes, use_pretrain = model_path, log_transform = False)"
- [methods] Optional Scanorama correction: "Batch correction using Scanorama::"
- [discussion] Unseen-type classification capability: "our method is able to classify cells into any cell type within the Cell Ontology, even if such cell type does not have annotated cells in the training set."
