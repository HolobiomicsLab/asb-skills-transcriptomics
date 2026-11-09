---
name: scrna-data-loading-and-preprocessing
description: Use when scRNA-seq training/test data must be loaded from h5ad files
  and preprocessed for ontology-based cell type annotation, especially when cell type
  labels are free text and need mapping to Cell Ontology IDs.
when_to_use_negative:
- Labels are already controlled Cell Ontology IDs — the NLP mapping step (Char-level
  LSTM) is unnecessary overhead
- The downstream task is unrelated to ontology-based annotation (e.g. clustering or
  trajectory analysis), where the Cell Ontology files and ProcessTrainFeature machinery
  are not applicable
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: OnClass
  role: Provides the data loading and feature-preprocessing API (read data with nlp_mapping,
    EmbedCellTypes, ProcessTrainFeature, ProcessTestFeature) used to prepare h5ad
    data and Cell Ontology files for annotation
  repo: https://github.com/wangshenguiuc/OnClass
- name: Scanorama
  role: Optional batch correction of the train and test gene panels before feature
    processing (Run_scanorama_batch_correction)
  repo: https://github.com/brianhie/scanorama
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:240e38aac8a45788aa21104d8fec654893eef4dc71008f5099b78489ad7a32ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - Scanorama
  - pip
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# scrna-data-loading-and-preprocessing

## Summary

Loading annotated single-cell RNA-seq datasets (h5ad) together with Cell Ontology resources (cl.obo, cell type network, cl.ontology.nlp.emb) and preparing expression features for OnClass training/testing, including optional Scanorama batch correction and NLP-based mapping of free-text labels to Cell Ontology terms.

## When to use

Use when scRNA-seq training/test data must be loaded from h5ad files and preprocessed for ontology-based cell type annotation, especially when cell type labels are free text and need mapping to Cell Ontology IDs.

## When NOT to use

- Labels are already controlled Cell Ontology IDs — the NLP mapping step (Char-level LSTM) is unnecessary overhead
- The downstream task is unrelated to ontology-based annotation (e.g. clustering or trajectory analysis), where the Cell Ontology files and ProcessTrainFeature machinery are not applicable

## Inputs

- Annotated training gene expression dataset (h5ad)
- Annotated test gene expression dataset (h5ad)
- Cell Ontology files: cl.obo, cell type network, cl.ontology.nlp.emb embeddings (from OnClass_data_public_minimal.tar.gz)
- Free-text cell type labels (optionally mapped via NLP to Cell Ontology IDs)

## Outputs

- cor_train_feature, cor_test_feature, cor_train_genes, cor_test_genes (processed feature matrices and gene lists from ProcessTrainFeature)
- Ontology-embedded cell type representations from EmbedCellTypes
- Optionally Scanorama batch-corrected train/test gene panel features

## How to apply

Read the annotated train/test gene expression matrices with anndata's read_h5ad, alongside the Cell Ontology files (cl.obo, cell type network, and cl.ontology.nlp.emb embeddings) from OnClass_data_public_minimal.tar.gz; set nlp_mapping=True to invoke the Char-level LSTM that maps uncontrolled vocabulary to controlled vocabulary. Embed the Cell Ontology via OnClassModel.EmbedCellTypes(train_label) to project cell types into the ontology-informed low-dimensional space. Optionally apply Scanorama batch correction between train and test gene panels when Run_scanorama_batch_correction is enabled. Then process features with ProcessTrainFeature to obtain cor_train_feature, cor_test_feature, cor_train_genes, and cor_test_genes, and process the test set with ProcessTestFeature(use_pretrain=model_path, log_transform=False) before prediction.

## Related tools

- **OnClass** (Provides the data loading and feature-preprocessing API (read data with nlp_mapping, EmbedCellTypes, ProcessTrainFeature, ProcessTestFeature) used to prepare h5ad data and Cell Ontology files for annotation) — https://github.com/wangshenguiuc/OnClass
- **Scanorama** (Optional batch correction of the train and test gene panels before feature processing (Run_scanorama_batch_correction)) — https://github.com/brianhie/scanorama

## Examples

```
OnClass_train_obj.ReadData(train_feature, train_label, test_feature, test_label, train_genes, test_genes, nlp_mapping=True); cor_train_feature, cor_test_feature, cor_train_genes, cor_test_genes = OnClass_train_obj.ProcessTrainFeature(train_feature, train_label, train_genes)
```

## Evaluation signals

- Processed features and gene lists (cor_train_feature, cor_test_feature, cor_train_genes, cor_test_genes) are returned from ProcessTrainFeature without errors and gene counts match the intersection of the train/test gene panels
- With nlp_mapping=True, free-text annotations are successfully mapped to Cell Ontology IDs via the Char-level LSTM before training
- If Scanorama batch correction is enabled, corrected train and test panels are aligned on the same corrected feature space
- Downstream validation: metrics from a model trained on these preprocessed features should match reported cross-dataset performance (AUPRC 0.79 Lemur 4→Lemur 1; unseen AUROC 0.94 Muris FACS→Lemur 2)

## Limitations

- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states, so labels may not map cleanly to ontology terms
- Methods to correct batch effects in the presence of many unseen cell types are not yet developed, so Scanorama correction may be unreliable when most test cell types are unseen
- OnClass requires Python 3.6 and specific package versions (OnClass 1.2, tensorflow 2.0), which constrains the environment for this preprocessing

## Evidence

- [methods] Read training and test data with nlp_mapping: "Read training and test data. Set nlp_mapping = True to use the Char-level LSTM that maps uncontrolled vocabulary to controlled vocabulary."
- [methods] ProcessTrainFeature feature processing: "cor_train_feature, cor_test_feature, cor_train_genes, cor_test_genes = OnClass_train_obj.ProcessTrainFeature(train_feature, train_label, train_genes"
- [methods] ProcessTestFeature with pretrained model: "cor_test_feature = OnClass_train_obj.ProcessTestFeature(cor_test_feature, cor_test_genes, use_pretrain = model_path, log_transform = False)"
- [readme] Scanorama batch correction role: [evidence span withheld — anchor status: external_doc]
- [methods] OnClass data availability and ontology resources: "we use the hierarchical ontology structure from the `Cell Ontology <http://www.obofoundry.org/ontology/cl.html>`__"
