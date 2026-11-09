---
name: random-walk-with-restart-embedding
description: Use when classifying scRNA-seq cells to Cell Ontology terms — including
  unseen cell types with no annotated training cells — or when cell type embeddings
  and cell-by-gene expression matrices must share one low-dimensional space.
when_to_use_negative:
- The task is plain cell type classification with a fixed label set and no need to
  annotate unseen ontology terms — a standard classifier without ontology embedding
  suffices
- You need to distinguish topologically identical unseen cell types (siblings with
  the same parents in the Cell Ontology graph), which OnClass cannot resolve
- The ontology is missing the cell types or cell states of interest, since the Cell
  Ontology was not developed specifically for scRNA-seq
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0089
tools:
- name: OnClass
  role: Implements EmbedCellTypes (random walk with restart + SVD on the Cell Ontology
    graph) and ProcessTrainFeature (nonlinear bilinear projection of transcriptomes
    into the same space)
  repo: https://github.com/wangshenguiuc/OnClass
- name: OnClass 1.2
  role: Pinned package version for the embedding pipeline (pip install OnClass==1.2)
  repo: https://github.com/wangshenguiuc/OnClass
- name: tensorflow 2.0
  role: Deep learning backend used by OnClass 1.2 for the embedding/model training
- name: Scanorama
  role: Optional batch correction of expression matrices before projecting them into
    the ontology embedding space
  repo: https://github.com/brianhie/scanorama
- name: Python 3.6
  role: Required interpreter version for running OnClass 1.2
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:4df341c5bcad1fe428c5ee2a967741a893194ebd997d5d1ca197f6320ff2c812
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# random-walk-with-restart-embedding

## Summary

Embeds Cell Ontology cell types into a shared low-dimensional space by running random walk with restart (clusDCA-style, restart probabilities 0.5-0.8) on the ontology 'is_a' graph followed by SVD, so that single-cell transcriptomes can be projected into the same space for ontology-aware classification.

## When to use

Use when classifying scRNA-seq cells to Cell Ontology terms — including unseen cell types with no annotated training cells — or when cell type embeddings and cell-by-gene expression matrices must share one low-dimensional space.

## When NOT to use

- The task is plain cell type classification with a fixed label set and no need to annotate unseen ontology terms — a standard classifier without ontology embedding suffices
- You need to distinguish topologically identical unseen cell types (siblings with the same parents in the Cell Ontology graph), which OnClass cannot resolve
- The ontology is missing the cell types or cell states of interest, since the Cell Ontology was not developed specifically for scRNA-seq

## Inputs

- Cell Ontology OBO file (cl_obo_file) from http://www.obofoundry.org/ontology/cl.html
- Cell type network file built from the ontology 'is_a' hierarchy (cell_type_network_file)
- Cell type NLP text embeddings file (cl.ontology.nlp.emb)
- Training cell-by-gene expression matrix (train_feature) with gene list (train_genes)
- Training cell type labels as Cell Ontology IDs (train_label)
- Optional test expression matrix and gene list

## Outputs

- Low-dimensional cell type embedding matrix (rows = Cell Ontology terms, e.g. saved to cell_type_embeddings.npz)
- Projected training feature matrix (cor_train_feature) in the shared embedding space
- Projected test feature matrix (cor_test_feature) with matching embedding dimensionality

## How to apply

Load the Cell Ontology with read_ontology_file('cell ontology', ontology_data_dir) to obtain cl_obo_file, the cell type network file, and the cl.ontology.nlp.emb NLP embeddings. Build an undirected similarity graph from the Cell Ontology 'is_a' hierarchical structure, then instantiate OnClassModel (OnClass 1.2, TensorFlow 2.0, Python 3.6) with the embedding and network files. Call OnClass_train_obj.EmbedCellTypes(train_label) to embed seen cell types via random walk with restart on the ontology graph (restart probabilities in the 0.5-0.8 range, clusDCA-style) followed by SVD into low-dimensional vectors. Then call ProcessTrainFeature(train_feature, train_label, train_genes, test_feature, test_genes) to project the cell-by-gene expression matrices into the same embedding space via a nonlinear bilinear transformation, optionally after Scanorama batch correction. Rationale: placing transcriptomes and ontology terms in one space lets downstream scoring extend predictions to unseen Cell Ontology terms through the graph structure.

## Related tools

- **OnClass** (Implements EmbedCellTypes (random walk with restart + SVD on the Cell Ontology graph) and ProcessTrainFeature (nonlinear bilinear projection of transcriptomes into the same space)) — https://github.com/wangshenguiuc/OnClass
- **OnClass 1.2** (Pinned package version for the embedding pipeline (pip install OnClass==1.2)) — https://github.com/wangshenguiuc/OnClass
- **tensorflow 2.0** (Deep learning backend used by OnClass 1.2 for the embedding/model training)
- **Scanorama** (Optional batch correction of expression matrices before projecting them into the ontology embedding space) — https://github.com/brianhie/scanorama
- **Python 3.6** (Required interpreter version for running OnClass 1.2)

## Examples

```
OnClass_train_obj.EmbedCellTypes(train_label); cor_train_feature, cor_test_feature, cor_train_genes, cor_test_genes = OnClass_train_obj.ProcessTrainFeature(train_feature, train_label, train_genes, test_feature, test_genes)
```

## Evaluation signals

- Every Cell Ontology ID in train_label receives a finite, nonzero row in the cell type embedding matrix
- The projected cell feature matrices (cor_train_feature, cor_test_feature) share the same embedding dimensionality as the cell type embedding matrix
- EmbedCellTypes runs to completion on the ontology graph built from 'is_a' relations with restart probabilities within the 0.5-0.8 range
- Downstream OnClass scores extend to unseen Cell Ontology terms via the Cell Ontology graph, matching the paper's classification behavior
- Saved cell_type_embeddings.npz contains both the cell type embedding matrix and the projected cell feature matrices

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- The Cell Ontology was not developed specifically for scRNA-seq and therefore likely misses cell types and cell states
- The random-walk embedding is restricted to seen cell types during training; unseen terms only receive scores in the later graph-propagation step

## Evidence

- [methods] embedding seen cell types via random walk on ontology graph: "OnClass_train_obj.EmbedCellTypes(train_label)"
- [intro] project transcriptomes into the same low-dimensional space: "by projecting single-cell transcriptomes and the Cell Ontology into the same low-dimensional space, OnClass advances other important applications"
- [discussion] extend scores to unseen cell types via ontology graph: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [readme] OnClass 1.2 upgrade: "Update on July 4th, 2021. OnClass is now upgraded to 1.2 version!"
- [methods] bilinear projection step: "cor_train_feature, cor_test_feature, cor_train_genes, cor_test_genes = OnClass_train_obj.ProcessTrainFeature(train_feature, train_label, train_genes"
