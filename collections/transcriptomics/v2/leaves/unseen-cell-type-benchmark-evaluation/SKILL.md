---
name: unseen-cell-type-benchmark-evaluation
description: Use when evaluating whether a cell-type annotation method can assign
  cells to Cell Ontology terms absent from the training set, especially on Tabula
  Muris Senis Muris droplet data with controlled unseen-term proportions.
when_to_use_negative:
- The task requires distinguishing topologically identical unseen cell types (siblings
  with the same parents in the Cell Ontology graph) — OnClass cannot separate them
- Annotations are uncontrolled free text needing the Char-level LSTM NLP mapping —
  this benchmark disables nlp_mapping and assumes controlled 'cell_ontology_id' labels
- Data require batch correction with many unseen cell types present — such correction
  methods are not yet developed per the article
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: OnClass
  role: Ontology-based classifier under test; embeds cell types, builds bilinear neural
    network, propagates scores to unseen Cell Ontology terms
  repo: https://github.com/wangshenguiuc/OnClass
- name: ACTINN
  role: Neural-network baseline for automated cell type identification trained on
    the same splits
  repo: https://github.com/mafeiyang/ACTINN
- name: tensorflow 2.0
  role: Backend for training the OnClass bilinear neural network model
- name: Scanorama
  role: Optional batch correction step documented in the OnClass workflow before feature
    processing
  repo: https://github.com/brianhie/scanorama
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:75ece8e09a274a8c94fcaaa511355b5c5a71548acd68de9dbbc298aab3cd0d8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - ACTINN
  - OnClass
  - Python
  - Scanorama
  - TensorFlow
  - pip
  - tensorflow 2.0
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# unseen-cell-type-benchmark-evaluation

## Summary

Benchmark OnClass against ACTINN, SVM, LR and reject-option variants (SVM(reject), LR(reject), Cell BLAST(reject), DOC(reject)) on Muris droplet splits with controlled proportions of unseen Cell Ontology terms, scored by AUROC over unseen terms and Accuracy@3/Accuracy@5.

## When to use

Use when evaluating whether a cell-type annotation method can assign cells to Cell Ontology terms absent from the training set, especially on Tabula Muris Senis Muris droplet data with controlled unseen-term proportions.

## When NOT to use

- The task requires distinguishing topologically identical unseen cell types (siblings with the same parents in the Cell Ontology graph) — OnClass cannot separate them
- Annotations are uncontrolled free text needing the Char-level LSTM NLP mapping — this benchmark disables nlp_mapping and assumes controlled 'cell_ontology_id' labels
- Data require batch correction with many unseen cell types present — such correction methods are not yet developed per the article

## Inputs

- Tabula Muris Senis Muris droplet scRNA-seq expression data as h5ad AnnData files (train/test splits with 'cell_ontology_id' labels)
- Cell Ontology resources: cl.obo, cell type network file, cl.ontology.nlp.emb NLP text embeddings
- figshare scRNA-seq archive (OnClass_data_public_scRNA_data.tar.gz, three parts) and OnClass_data_public_minimal.tar.gz

## Outputs

- pred_Y_seen and pred_Y_all score matrices from OnClassModel.Predict covering all Cell Ontology terms including unseen ones
- AUROC over unseen cell type terms per method per unseen proportion
- Accuracy@3 and Accuracy@5 per method per unseen proportion

## How to apply

Download the three-part figshare scRNA-seq archive and extract with 'cat OnClass_data_public_scRNA_data.tar.gz.* | tar -xz'; extract OnClass_data_public_minimal.tar.gz for the Cell Ontology files (cl.obo, cell type network file, cl.ontology.nlp.emb text embeddings). Load ontologies via read_ontology_file, then read train/test h5ad data with read_data using the 'cell_ontology_id' AnnData label key with nlp_mapping disabled for controlled-vocabulary labels. Split Muris droplet data so the test set holds controlled proportions of unseen terms (runs at 70% and 30% unseen). Train OnClass via EmbedCellTypes(train_label), ProcessTrainFeature, BuildModel(ngene=len(cor_train_genes), nhidden=NHIDDEN), Train(cor_train_feature, train_label, max_iter=MAX_ITER), then Predict with use_normalize=True to obtain pred_Y_seen and pred_Y_all that propagate scores to all ontology terms including unseen ones; train and predict with the comparison methods on identical splits.

## Related tools

- **OnClass** (Ontology-based classifier under test; embeds cell types, builds bilinear neural network, propagates scores to unseen Cell Ontology terms) — https://github.com/wangshenguiuc/OnClass
- **ACTINN** (Neural-network baseline for automated cell type identification trained on the same splits) — https://github.com/mafeiyang/ACTINN
- **tensorflow 2.0** (Backend for training the OnClass bilinear neural network model)
- **Scanorama** (Optional batch correction step documented in the OnClass workflow before feature processing) — https://github.com/brianhie/scanorama

## Examples

```
cat OnClass_data_public_scRNA_data.tar.gz.* | tar -xz && pip install OnClass==1.2 tensorflow==2.0
```

## Evaluation signals

- At 70% unseen cell types, OnClass reaches AUROC ~0.87 while ACTINN, SVM, LR and reject variants all fall below 0.67
- At 30% unseen cell types, OnClass attains Accuracy@3 = 0.56 and Accuracy@5 = 0.60
- Predict(use_normalize=True) returns pred_Y_seen and pred_Y_all whose columns cover all Cell Ontology terms, not just training labels
- Train/test h5ad files load with 'cell_ontology_id' as the label key and nlp_mapping disabled without mapping errors
- The three-part figshare archive concatenates and extracts cleanly into Muris droplet expression data plus ontology files from OnClass_data_public_minimal.tar.gz

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states
- Methods to correct batch effects in the presence of many unseen cell types are not yet developed

## Evidence

- [discussion] Propagates scores to unseen terms: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [intro] Benchmark conclusion: "Through careful benchmarking, we found that our method outperformed existing methods at annotating unseen Cell Ontology terms"
- [methods] Installation: "pip install OnClass==1.2"
- [methods] Training API: "OnClass_train_obj.BuildModel(ngene = len(cor_train_genes), nhidden = NHIDDEN)"
- [readme] Version note: "Update on July 4th, 2021. OnClass is now upgraded to 1.2 version!"
