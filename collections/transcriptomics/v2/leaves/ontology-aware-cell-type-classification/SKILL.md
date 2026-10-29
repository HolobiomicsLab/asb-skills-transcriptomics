---
name: ontology-aware-cell-type-classification
description: Use when annotating scRNA-seq cells against a controlled vocabulary (Cell
  Ontology) and the test set contains cell types absent from training data, or when
  benchmarking classifiers on unseen-term annotation.
when_to_use_negative:
- Labels are free text rather than controlled-vocabulary Cell Ontology IDs — the NLP
  mapping path (nlp_mapping=True) is required, but the article notes improving NLP
  models remains future work
- Distinguishing topologically identical unseen cell types (siblings sharing the same
  parents in the Cell Ontology graph) is required — OnClass explicitly cannot do this
- The task is batch correction with many unseen cell types present — the article states
  such methods are not yet developed
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0621
tools:
- name: OnClass
  role: 'Ontology-aware classifier: embeds cell types, builds bilinear neural network,
    and propagates prediction scores to unseen Cell Ontology terms'
  repo: https://github.com/wangshenguiuc/OnClass
- name: ACTINN
  role: Baseline neural-network cell type identifier trained on the same splits for
    comparison
  repo: https://github.com/mafeiyang/ACTINN
- name: tensorflow 2.0
  role: Deep-learning backend used to install and run OnClass
- name: pip
  role: Installation channel for OnClass 1.2 and tensorflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:5a17b554c1954b03d88863e3f3e916f09d5adacf257ae21b54b8ffa47a8f1446
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - ACTINN
  - OnClass
  - Python
  - TensorFlow
  - pip
  - tensorflow 2.0
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# ontology-aware-cell-type-classification

## Summary

Classify single cells into Cell Ontology terms — including unseen terms with no annotated training cells — by projecting scRNA-seq transcriptomes and the ontology graph into a shared low-dimensional space with OnClass, and benchmark against baselines like ACTINN, SVM, and LR using AUROC and Accuracy@k.

## When to use

Use when annotating scRNA-seq cells against a controlled vocabulary (Cell Ontology) and the test set contains cell types absent from training data, or when benchmarking classifiers on unseen-term annotation.

## When NOT to use

- Labels are free text rather than controlled-vocabulary Cell Ontology IDs — the NLP mapping path (nlp_mapping=True) is required, but the article notes improving NLP models remains future work
- Distinguishing topologically identical unseen cell types (siblings sharing the same parents in the Cell Ontology graph) is required — OnClass explicitly cannot do this
- The task is batch correction with many unseen cell types present — the article states such methods are not yet developed

## Inputs

- Tabula Muris Senis Muris droplet scRNA-seq expression data (h5ad with 'cell_ontology_id' AnnData label key)
- Cell Ontology files: cl.obo, cell type network file, and cl.ontology.nlp.emb text embeddings
- Train/test splits with controlled proportions of unseen Cell Ontology terms (e.g., 70% and 30%)

## Outputs

- pred_Y_seen and pred_Y_all score matrices over all Cell Ontology terms from OnClassModel.Predict(use_normalize=True)
- AUROC over unseen cell type terms for OnClass and each baseline method
- Accuracy@3 and Accuracy@5 metrics per split

## How to apply

Download the figshare scRNA-seq archive ('cat OnClass_data_public_scRNA_data.tar.gz.* | tar -xz') and OnClass_data_public_minimal.tar.gz for the Cell Ontology files (cl.obo, cell type network, cl.ontology.nlp.emb). Load ontology resources via read_ontology_file, then read train/test h5ad with read_data using the 'cell_ontology_id' AnnData key (nlp_mapping=False for controlled-vocabulary labels). Split data so the test set holds controlled proportions of unseen terms (e.g., 70% and 30%). For OnClass, call EmbedCellTypes(train_label), ProcessTrainFeature, BuildModel(ngene=len(cor_train_genes), nhidden=NHIDDEN), Train(cor_train_feature, train_label, max_iter=MAX_ITER), then Predict(use_normalize=True) to get pred_Y_seen and pred_Y_all whose scores propagate to all Cell Ontology terms including unseen ones. Train comparison methods (ACTINN, SVM, LR and reject-option variants SVM(reject), LR(reject), Cell BLAST(reject), DOC(reject)) on identical splits; the rationale is that OnClass's bilinear neural network uses the ontology graph to extend scores to unseen terms, which flat classifiers cannot do.

## Related tools

- **OnClass** (Ontology-aware classifier: embeds cell types, builds bilinear neural network, and propagates prediction scores to unseen Cell Ontology terms) — https://github.com/wangshenguiuc/OnClass
- **ACTINN** (Baseline neural-network cell type identifier trained on the same splits for comparison) — https://github.com/mafeiyang/ACTINN
- **tensorflow 2.0** (Deep-learning backend used to install and run OnClass)
- **pip** (Installation channel for OnClass 1.2 and tensorflow)

## Examples

```
pip install OnClass==1.2 tensorflow==2.0; OnClass_train_obj.EmbedCellTypes(train_label); OnClass_train_obj.BuildModel(ngene=len(cor_train_genes), nhidden=NHIDDEN); OnClass_train_obj.Train(cor_train_feature, train_label, max_iter=MAX_ITER)
```

## Evaluation signals

- At 70% unseen cell types, OnClass should reach AUROC ≈ 0.87 while all baselines (ACTINN, SVM, LR and reject variants) fall below 0.67
- At 30% unseen terms, Accuracy@3 = 0.56 and Accuracy@5 = 0.60 are the accepted reference values
- pred_Y_all must contain non-zero scores for Cell Ontology terms absent from the training labels (score propagation to unseen terms succeeded)
- AUROC computed strictly over unseen terms; Accuracy@3/Accuracy@5 computed on the same controlled splits for every method

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states
- Methods to correct batch effects in the presence of many unseen cell types are not yet developed

## Evidence

- [results] OnClass outperforms at 70% unseen: [evidence span withheld — anchor status: not_found]
- [abstract] Unseen-term capability via ontology graph: "A key advantage of OnClass is its capability to classify cells into cell types not present in the training data because it uses the Cell Ontology graph to infer cell type relationships."
- [methods] Training API calls: "OnClass_train_obj.BuildModel(ngene = len(cor_train_genes), nhidden = NHIDDEN)"
- [methods] Install versions: "pip install OnClass==1.2"
- [discussion] Sibling limitation: "Another limitation of OnClass is that it cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph."
