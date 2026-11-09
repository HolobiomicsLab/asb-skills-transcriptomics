---
name: scanorama-batch-correction
description: Use when training and test scRNA-seq datasets come from different studies,
  platforms, or gene panels (e.g., Lemur 4 vs. Lemur 1, Muris FACS vs. Lemur 2) and
  batch effects may obscure cross-dataset cell type annotation with OnClass.
when_to_use_negative:
- Train and test cells come from the same dataset or sequencing run — batch correction
  could remove real biological signal
- The test set contains a very large number of unseen cell types relative to the reference,
  a setting the OnClass authors flag as one where batch-effect correction methods
  are not yet developed
- Features are already integrated or normalized across datasets (e.g., prior Scanpy
  external.pp.scanorama_integrate was applied)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Scanorama
  role: Batch-corrects train and test gene panels so OnClass classifiers are not confounded
    by cross-dataset technical effects
  repo: https://github.com/brianhie/scanorama
- name: OnClass
  role: Consumes the batch-corrected features for ontology-informed cell type training,
    prediction, and unseen-term scoring
  repo: https://github.com/wangshenguiuc/OnClass
- name: Python
  role: Implementation environment for the OnClass + Scanorama pipeline
  repo: https://github.com/wangshenguiuc/OnClass
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:c28abfe7a30609a4bbbf531ecc3fd89e82a3ffb079626b8f1d933df0e44abed7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/brianhie/scanorama
  tools:
  - OnClass
  - Python
  - Scanorama
  - pip
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# scanorama-batch-correction

## Summary

Optionally apply Scanorama batch correction between training and test gene expression panels (e.g., Lemur/Muris single-cell datasets) before OnClass training and prediction, so that platform/dataset effects do not confound cross-dataset cell type annotation.

## When to use

Use when training and test scRNA-seq datasets come from different studies, platforms, or gene panels (e.g., Lemur 4 vs. Lemur 1, Muris FACS vs. Lemur 2) and batch effects may obscure cross-dataset cell type annotation with OnClass.

## When NOT to use

- Train and test cells come from the same dataset or sequencing run — batch correction could remove real biological signal
- The test set contains a very large number of unseen cell types relative to the reference, a setting the OnClass authors flag as one where batch-effect correction methods are not yet developed
- Features are already integrated or normalized across datasets (e.g., prior Scanpy external.pp.scanorama_integrate was applied)

## Inputs

- Training dataset: annotated gene expression matrix (h5ad, read via anndata.read_h5ad) with Cell Ontology-mapped labels
- Test dataset: gene expression matrix (h5ad) from a different dataset/platform
- Gene lists for train and test panels (cor_train_genes, cor_test_genes)
- Cell Ontology files (cl.obo, cell type network, cl.ontology.nlp.emb embeddings) from OnClass_data_public_minimal.tar.gz

## Outputs

- Batch-corrected train and test gene expression matrices aligned on the shared gene panel
- Corrected features feeding OnClass ProcessTrainFeature / ProcessTestFeature and downstream predictions (pred_Y_seen, pred_Y_all, pred_label) saved to cross_dataset_prediction_results.csv

## How to apply

After loading the annotated training and test h5ad datasets with anndata and embedding the Cell Ontology via OnClassModel.EmbedCellTypes, enable Run_scanorama_batch_correction to align the train and test gene panels: pass the two cell-by-gene matrices and their gene lists to Scanorama's correct() (or correct_scanpy() for AnnData objects), obtaining batch-corrected feature matrices. Feed the corrected features into OnClassModel.ProcessTrainFeature before BuildModel/Train, and use ProcessTestFeature with the saved model file before Predict(test_genes=cor_test_genes, use_normalize=True). Rationale: Scanorama corrects for dataset-level technical effects so the bilinear neural network learns biology rather than batch; in the OnClass paper this step sits between ontology embedding and feature processing, and the corrected pipeline is validated by reproducing cross-dataset metrics (AUPRC 0.79 Lemur 4→Lemur 1; unseen AUROC 0.94 Muris FACS→Lemur 2 with 67% unseen cell types).

## Related tools

- **Scanorama** (Batch-corrects train and test gene panels so OnClass classifiers are not confounded by cross-dataset technical effects) — https://github.com/brianhie/scanorama
- **OnClass** (Consumes the batch-corrected features for ontology-informed cell type training, prediction, and unseen-term scoring) — https://github.com/wangshenguiuc/OnClass
- **Python** (Implementation environment for the OnClass + Scanorama pipeline) — https://github.com/wangshenguiuc/OnClass

## Examples

```
import scanorama; corrected, genes = scanorama.correct([train_feature, test_feature], [train_genes, test_genes])
```

## Evaluation signals

- Reproduced cross-dataset metrics match the reported values: AUPRC 0.79 for Lemur 4→Lemur 1 and AUROC 0.94 for unseen cell types in Muris FACS→Lemur 2 (67% unseen cell types)
- Corrected train and test matrices share the same gene panel (cor_train_genes == cor_test_genes dimensionality)
- Seen-term AUPRC and unseen-term AUROC computed on the test set are written to cross_dataset_prediction_results.csv
- Predicted labels (pred_label) fall within valid Cell Ontology terms after nlp_mapping of uncontrolled vocabulary
- Scanorama correction runs without altering cell counts of train/test matrices (only feature values change)

## Limitations

- Methods to correct batch effects in the presence of many unseen cell types are not yet developed
- Scanorama correction is optional; the OnClass workflow enables it only when Run_scanorama_batch_correction is set
- The Cell Ontology itself may miss cell types and cell states, and OnClass cannot distinguish topologically identical (sibling) unseen cell types — batch correction does not fix these issues

## Evidence

- [methods] Batch correction using Scanorama: "Batch correction using Scanorama::"
- [methods] Read training and test data with nlp mapping: "Read training and test data. Set nlp_mapping = True to use the Char-level LSTM that maps uncontrolled vocabulary to controlled vocabulary."
- [readme] Scanorama batch correction purpose: [evidence span withheld — anchor status: external_doc]
- [readme] Scanorama correct usage: [evidence span withheld — anchor status: external_doc]
- [discussion] Unseen cell type batch correction limitation: "We would also like to develop methods to correct the batch effects with the existence of a large number of unseen cell types."
