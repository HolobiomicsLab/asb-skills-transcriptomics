---
name: cell-ontology-embedding
description: Use when annotating single-cell data against the Cell Ontology controlled
  vocabulary, especially when test labels may include unseen cell type terms that
  have no annotated cells in the training set.
when_to_use_negative:
- When labels are not Cell Ontology terms and no controlled-vocabulary mapping is
  possible (the embedding projects Cell Ontology terms specifically)
- When the goal is to distinguish topologically identical unseen cell types (siblings
  with the same parents in the Cell Ontology graph), which OnClass cannot separate
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: OnClass
  role: Provides the EmbedCellTypes method that projects the Cell Ontology (cl.obo
    + cl.ontology.nlp.emb) into the ontology-informed low-dimensional space consumed
    by the bilinear classifier
  repo: https://github.com/wangshenguiuc/OnClass
- name: OnClass 1.2
  role: Pinned pip version of the OnClass package whose ontology embedding is used
  repo: https://github.com/wangshenguiuc/OnClass
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:344f667b29840ff93cb770d059ec89faede04c08dc9d97febd7352697434e0e2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - OnClass 1.2
  - Python
  - TensorFlow
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cell-ontology-embedding

## Summary

Project cell type terms from the Cell Ontology (cl.obo plus cl.ontology.nlp.emb NLP embeddings) into a low-dimensional, ontology-informed vector space using OnClass's EmbedCellTypes, so that cell types—including terms with no annotated cells in the training data—become classifiable alongside single-cell transcriptomes.

## When to use

Use when annotating single-cell data against the Cell Ontology controlled vocabulary, especially when test labels may include unseen cell type terms that have no annotated cells in the training set.

## When NOT to use

- When labels are not Cell Ontology terms and no controlled-vocabulary mapping is possible (the embedding projects Cell Ontology terms specifically)
- When the goal is to distinguish topologically identical unseen cell types (siblings with the same parents in the Cell Ontology graph), which OnClass cannot separate

## Inputs

- Cell Ontology files: cl.obo, cell type network file, and cl.ontology.nlp.emb NLP embeddings (from OnClass_data_public_minimal.tar.gz)
- Training cell type labels as Cell Ontology IDs (e.g. the 'cell_ontology_id' AnnData label key) or free text with nlp_mapping enabled
- Annotated training scRNA-seq expression data in h5ad format (e.g. Tabula Muris Senis, Lemur datasets)

## Outputs

- Ontology-informed low-dimensional embeddings of cell type terms used by the OnClass bilinear model
- Embedded label space supporting pred_Y_seen and pred_Y_all score vectors covering all Cell Ontology terms, including unseen ones

## How to apply

Load the Cell Ontology resources from OnClass_data_public_minimal.tar.gz (cl.obo, the cell type network file, and the cl.ontology.nlp.emb text embeddings), optionally enabling nlp_mapping to map free-text labels to Cell Ontology IDs. Call OnClassModel.EmbedCellTypes(train_label) to embed the training cell type labels into the ontology-informed low-dimensional space; this projection is what lets the subsequent bilinear neural network propagate probability scores to all Cell Ontology terms, including unseen ones. The embedded ontology space also supports downstream marker gene computation across all Cell Ontology categories. Embedding is done once per training label set, before ProcessTrainFeature and BuildModel(ngene=len(cor_train_genes), nhidden=NHIDDEN).

## Related tools

- **OnClass** (Provides the EmbedCellTypes method that projects the Cell Ontology (cl.obo + cl.ontology.nlp.emb) into the ontology-informed low-dimensional space consumed by the bilinear classifier) — https://github.com/wangshenguiuc/OnClass
- **OnClass 1.2** (Pinned pip version of the OnClass package whose ontology embedding is used) — https://github.com/wangshenguiuc/OnClass

## Examples

```
OnClass_train_obj.EmbedCellTypes(train_label)
```

## Evaluation signals

- Prediction outputs include pred_Y_all with scores for all Cell Ontology terms, confirming unseen-term scores were propagated through the embedded ontology
- On the Muris FACS to Lemur 2 cross-dataset setting with 67% unseen cell types, unseen-term AUROC should reach ~0.94
- On Muris droplet with 70% unseen cell types, AUROC should reach ~0.87 (vs. baselines below 0.67), and Accuracy@3 = 0.56 / Accuracy@5 = 0.60 at 30% unseen
- OnClass achieves AUPRC 0.79 when trained on Lemur 4 and tested on Lemur 1, indicating the embedded ontology space transfers across datasets

## Limitations

- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states
- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- Siblings indistinguishable in the ontology graph can only be separated as more annotated datasets become available

## Evidence

- [methods] Ontology embedding step grounds the label space: "OnClass_train_obj.EmbedCellTypes(train_label)"
- [intro] Rationale: joint low-dimensional projection enables marker genes and downstream tasks: "by projecting single-cell transcriptomes and the Cell Ontology into the same low-dimensional space, OnClass advances other important applications, such as robust computation of marker genes"
- [abstract] Ontology graph lets scores extend to unseen terms: "A key advantage of OnClass is its capability to classify cells into cell types not present in the training data because it uses the Cell Ontology graph to infer cell type relationships."
- [methods] Pinned package version: "pip install OnClass==1.2"
