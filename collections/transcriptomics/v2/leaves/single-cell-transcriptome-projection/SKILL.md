---
name: single-cell-transcriptome-projection
description: Use when you need cell expression profiles and Cell Ontology cell-type
  terms to share one embedding space, e.g. to classify cells to unseen ontology terms
  or compute marker genes per ontology category.
when_to_use_negative:
- Your cell types of interest are topologically identical siblings in the Cell Ontology
  graph (same parents) — OnClass cannot distinguish them and projection will not separate
  them.
- Input matrices are already corrected embeddings from another method (e.g. Scanorama
  integration output) and no Cell Ontology term mapping is needed.
- The Cell Ontology does not capture your cell states — it was not developed specifically
  for scRNA-seq and likely misses cell types and cell states.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: OnClass 1.2
  role: Provides EmbedCellTypes (ontology graph random walk with restart + SVD) and
    ProcessTrainFeature/ProcessTestFeature (nonlinear bilinear projection of expression
    matrices into the shared embedding space)
  repo: https://github.com/wangshenguiuc/OnClass
- name: Python 3.6
  role: Runtime environment required to run OnClass 1.2
  repo: https://github.com/wangshenguiuc/OnClass
- name: tensorflow 2.0
  role: Backend OnClass 1.2 is installed and trained against
- name: Scanorama
  role: Optional upstream batch correction before projecting train/test features into
    the embedding space
  repo: https://github.com/brianhie/scanorama
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:6d9413b186df91f204b07def8c47097734259fda742056d6fbfd06f0d7d41945
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - OnClass 1.2
  - Python
  - Python 3.6
  - Scanorama
  - TensorFlow
  - tensorflow 2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# single-cell-transcriptome-projection

## Summary

Projects cell-by-gene scRNA-seq expression matrices into the same low-dimensional space in which Cell Ontology cell types have been embedded, so that cell profiles and ontology terms become directly comparable for classification and marker gene computation.

## When to use

Use when you need cell expression profiles and Cell Ontology cell-type terms to share one embedding space, e.g. to classify cells to unseen ontology terms or compute marker genes per ontology category.

## When NOT to use

- Your cell types of interest are topologically identical siblings in the Cell Ontology graph (same parents) — OnClass cannot distinguish them and projection will not separate them.
- Input matrices are already corrected embeddings from another method (e.g. Scanorama integration output) and no Cell Ontology term mapping is needed.
- The Cell Ontology does not capture your cell states — it was not developed specifically for scRNA-seq and likely misses cell types and cell states.

## Inputs

- Cell Ontology files: cl_obo_file (OBO), cell_type_network_file (is_a hierarchy), cell_type_nlp_emb_file (cl.ontology.nlp.emb text embeddings)
- train_label: Cell Ontology term IDs for training cells
- train_feature / test_feature: cell-by-gene expression matrices
- train_genes / test_genes: gene lists matching the expression matrices

## Outputs

- Low-dimensional cell-type embedding matrix (from EmbedCellTypes, SVD of random-walk-with-restart propagation)
- cor_train_feature and cor_test_feature: projected cell feature matrices in the shared embedding space
- cor_train_genes / cor_test_genes: processed gene lists
- Saved artifacts such as cell_type_embeddings.npz

## How to apply

First load the ontology files via read_ontology_file('cell ontology', ontology_data_dir) to obtain cl_obo_file, the cell_type_network_file, and the cl.ontology.nlp.emb NLP embeddings, then build an undirected cell-type similarity graph from the Cell Ontology 'is_a' hierarchy. Instantiate OnClassModel (OnClass 1.2, TensorFlow 2.0, Python 3.6) with these two files and call EmbedCellTypes(train_label), which runs random walk with restart on the ontology graph (restart probabilities in the 0.5-0.8 range, clusDCA-style) followed by SVD to produce low-dimensional cell-type embeddings. Then call ProcessTrainFeature(train_feature, train_label, train_genes, test_feature, test_genes), which applies the nonlinear bilinear transformation to push the cell-by-gene matrices into the same embedding space as the ontology terms. The rationale is that only a shared space lets OnClass extend classification scores to unseen Cell Ontology terms and supports robust marker gene computation. Save the resulting cell-type embedding matrix and projected feature matrices (e.g. to cell_type_embeddings.npz) for downstream BuildModel/Train steps.

## Related tools

- **OnClass 1.2** (Provides EmbedCellTypes (ontology graph random walk with restart + SVD) and ProcessTrainFeature/ProcessTestFeature (nonlinear bilinear projection of expression matrices into the shared embedding space)) — https://github.com/wangshenguiuc/OnClass
- **Python 3.6** (Runtime environment required to run OnClass 1.2) — https://github.com/wangshenguiuc/OnClass
- **tensorflow 2.0** (Backend OnClass 1.2 is installed and trained against)
- **Scanorama** (Optional upstream batch correction before projecting train/test features into the embedding space) — https://github.com/brianhie/scanorama

## Examples

```
cor_train_feature, cor_test_feature, cor_train_genes, cor_test_genes = OnClass_train_obj.ProcessTrainFeature(train_feature, train_label, train_genes, test_feature, test_genes)
```

## Evaluation signals

- Every Cell Ontology ID present in train_label receives a finite, nonzero embedding row after EmbedCellTypes.
- The projected cor_train_feature / cor_test_feature matrices share the same embedding dimensionality as the cell-type embedding matrix.
- Embedding matrix and projected features save cleanly to cell_type_embeddings.npz without NaNs or shape mismatches.
- Gene lists returned by ProcessTrainFeature (cor_train_genes, cor_test_genes) remain consistent with the input train_genes/test_genes.
- Downstream OnClass_train_obj.BuildModel/Train runs without dimensionality errors on the projected features.

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph.
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states.
- Methods to correct batch effects in the presence of many unseen cell types are not yet developed.

## Evidence

- [intro] Shared-space rationale: "by projecting single-cell transcriptomes and the Cell Ontology into the same low-dimensional space, OnClass advances other important applications, such as robust computation of marker genes"
- [methods] Ontology embedding call: "OnClass_train_obj.EmbedCellTypes(train_label)"
- [methods] Projection call: "cor_train_feature, cor_test_feature, cor_train_genes, cor_test_genes = OnClass_train_obj.ProcessTrainFeature(train_feature, train_label, train_genes"
- [discussion] Score extension to unseen terms: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [readme] Version pinning: "Update on July 4th, 2021. OnClass is now upgraded to 1.2 version!"
