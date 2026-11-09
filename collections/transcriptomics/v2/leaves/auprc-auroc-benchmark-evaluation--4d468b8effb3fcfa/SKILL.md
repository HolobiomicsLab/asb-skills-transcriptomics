---
name: auprc-auroc-benchmark-evaluation
description: Use when evaluating a single-cell cell-type classifier in a cross-dataset
  or unseen-term setting, where test labels include Cell Ontology terms absent from
  the training set.
when_to_use_negative:
- The task is within-dataset cross-validation with no unseen Cell Ontology terms;
  AUPRC/AUROC over unseen terms is not meaningful when all test labels appear in training
  (use the run_one_dataset_cross_validation.py path instead)
- You need to distinguish topologically identical unseen cell types (siblings with
  the same parents in the Cell Ontology graph) — OnClass explicitly cannot do this
- Your annotation labels are free text and you are not using the provided NLP model
  to map them to Cell Ontology terms; scores will be invalid until labels are mapped
  to the controlled vocabulary
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: OnClass
  role: Trains the ontology-informed bilinear neural network and extends probability
    scores to unseen cell types via the Cell Ontology graph; produces the predictions
    scored by AUPRC/AUROC
  repo: https://github.com/wangshenguiuc/OnClass
- name: Scanorama
  role: Optional batch correction between train and test gene panels (Run_scanorama_batch_correction)
    before model training/prediction
  repo: https://github.com/brianhie/scanorama
- name: pip
  role: Installs OnClass==1.2 and tensorflow==2.0 dependencies
  repo: https://github.com/wangshenguiuc/OnClass
- name: tensorflow 2.0
  role: Backend for the OnClass bilinear neural network model
  repo: https://github.com/wangshenguiuc/OnClass
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:49e819aaaea4c4d2deeaad541bd6ecf5e7ef5839da530270af157a4566f14b6a
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# auprc-auroc-benchmark-evaluation

## Summary

Benchmarking protocol for ontology-based single-cell annotation: train an OnClass model on one dataset (e.g., Lemur 4), test on another (e.g., Lemur 1), and score seen-cell-type predictions with AUPRC and unseen-cell-type predictions with AUROC.

## When to use

Use when evaluating a single-cell cell-type classifier in a cross-dataset or unseen-term setting, where test labels include Cell Ontology terms absent from the training set.

## When NOT to use

- The task is within-dataset cross-validation with no unseen Cell Ontology terms; AUPRC/AUROC over unseen terms is not meaningful when all test labels appear in training (use the run_one_dataset_cross_validation.py path instead)
- You need to distinguish topologically identical unseen cell types (siblings with the same parents in the Cell Ontology graph) — OnClass explicitly cannot do this
- Your annotation labels are free text and you are not using the provided NLP model to map them to Cell Ontology terms; scores will be invalid until labels are mapped to the controlled vocabulary

## Inputs

- Annotated training single-cell gene expression matrix (h5ad) plus cell labels
- Annotated test single-cell gene expression matrix (h5ad) from a different dataset (e.g., Lemur 1 or Lemur 2)
- Cell Ontology files: cl.obo, cell type network, and cl.ontology.nlp.emb embeddings (OnClass_data_public_minimal.tar.gz)
- Optional NLP mapping from uncontrolled vocabulary to controlled Cell Ontology vocabulary

## Outputs

- pred_Y_seen, pred_Y_all, and pred_label covering all Cell Ontology terms for each test cell
- Seen-term AUPRC and unseen-term AUROC metrics (targets: 0.79 Lemur 4->Lemur 1 AUPRC; 0.94 Muris FACS->Lemur 2 unseen AUROC)
- cross_dataset_prediction_results.csv with all predictions and metrics

## How to apply

Load the annotated train/test h5ad files with anndata's read_h5ad together with the Cell Ontology files (cl.obo, cell type network, cl.ontology.nlp.emb) from OnClass_data_public_minimal.tar.gz, optionally setting nlp_mapping to map free-text labels to Cell Ontology IDs. Embed cell types with OnClassModel.EmbedCellTypes(train_label), optionally apply Scanorama batch correction between train and test gene panels, then call ProcessTrainFeature, BuildModel(ngene=len(cor_train_genes), nhidden=NHIDDEN), and Train(save_model=model_path, max_iter=MAX_ITER). For testing, call ProcessTestFeature(use_pretrain=model_path, log_transform=False), rebuild the model from the pretrained file, and Predict(test_genes=cor_test_genes, use_normalize=True) to obtain pred_Y_seen, pred_Y_all, and pred_label over all Cell Ontology terms. Compute seen-term AUPRC and unseen-term AUROC on the test set; the article's reproduction targets are AUPRC 0.79 for Lemur 4 to Lemur 1 and AUROC 0.94 on unseen terms in the cross-species Muris FACS to Lemur 2 setting (67% unseen cell types). Save predictions and metrics to cross_dataset_prediction_results.csv.

## Related tools

- **OnClass** (Trains the ontology-informed bilinear neural network and extends probability scores to unseen cell types via the Cell Ontology graph; produces the predictions scored by AUPRC/AUROC) — https://github.com/wangshenguiuc/OnClass
- **Scanorama** (Optional batch correction between train and test gene panels (Run_scanorama_batch_correction) before model training/prediction) — https://github.com/brianhie/scanorama
- **pip** (Installs OnClass==1.2 and tensorflow==2.0 dependencies) — https://github.com/wangshenguiuc/OnClass
- **tensorflow 2.0** (Backend for the OnClass bilinear neural network model) — https://github.com/wangshenguiuc/OnClass

## Examples

```
git clone https://github.com/wangshenguiuc/OnClass.git; python script/run_cross_dataset_prediction.py
```

## Evaluation signals

- Seen-term AUPRC on the held-out dataset matches ~0.79 for the Lemur 4 (train) to Lemur 1 (test) split
- Unseen-term AUROC matches ~0.94 in the cross-species Muris FACS (train) to Lemur 2 (test) setting with 67% unseen cell types
- pred_label output covers all Cell Ontology terms (not just training-set terms), confirming unseen-term scoring is possible
- cross_dataset_prediction_results.csv is written containing all predictions and both metrics
- Model file saved via Train(save_model=model_path) is successfully reloaded by ProcessTestFeature(use_pretrain=model_path)

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- Methods to correct batch effects in the presence of many unseen cell types are not yet developed, which may affect cross-dataset scores
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states, capping the meaningful label space

## Evidence

- [intro] intro: "Through careful benchmarking, we found that our method outperformed existing methods at annotating unseen Cell Ontology terms"
- [methods] methods: "cor_test_feature = OnClass_train_obj.ProcessTestFeature(cor_test_feature, cor_test_genes, use_pretrain = model_path, log_transform = False)"
- [discussion] discussion: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [readme] readme: [evidence span withheld — anchor status: external_doc]
