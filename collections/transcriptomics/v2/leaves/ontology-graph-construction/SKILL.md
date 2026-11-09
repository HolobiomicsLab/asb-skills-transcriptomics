---
name: ontology-graph-construction
description: Use when a scRNA-seq classifier must reason over a controlled vocabulary
  of cell type terms — e.g. to score unseen Cell Ontology terms or compute marker
  genes for all ontology categories — and raw obo/network files are available.
when_to_use_negative:
- The task only needs classifying into a fixed, flat label set with no ontology vocabulary
  — a standard classifier (e.g. ACTINN) suffices and graph embedding adds overhead
- Cell types of interest are topologically identical siblings (same parents in the
  Cell Ontology graph) that must be distinguished — OnClass cannot separate these
- The Cell Ontology lacks terms for the cell states being studied, since it was not
  developed specifically for scRNA-seq and likely misses cell types and cell states
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3308
tools:
- name: OnClass
  role: Provides the ontology graph construction and EmbedCellTypes routine (random
    walk with restart + SVD) that embeds Cell Ontology terms and projects transcriptomes
    into the shared space
  repo: https://github.com/wangshenguiuc/OnClass
- name: OnClass 1.2
  role: Pinned package version whose OnClassModel consumes cell_type_nlp_emb_file
    and cell_type_network_file to run the embedding
  repo: https://github.com/wangshenguiuc/OnClass
- name: tensorflow 2.0
  role: Backend dependency for running the OnClassModel embedding and bilinear projection
- name: Python 3.6
  role: Required runtime per the paper's installation instructions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:69909804e30c396c148b0ac4de6655e4165afd74fb9c3624a6de444e2dc22c71
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - OnClass 1.2
  - Python
  - Python 3.6
  - TensorFlow
  - tensorflow 2.0
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# ontology-graph-construction

## Summary

Constructs an undirected cell type similarity graph from the Cell Ontology 'is_a' hierarchy and embeds its terms into a low-dimensional space (random walk with restart followed by SVD, OnClass 1.2/TensorFlow 2.0), so that single-cell transcriptomes can be projected into the same shared embedding space.

## When to use

Use when a scRNA-seq classifier must reason over a controlled vocabulary of cell type terms — e.g. to score unseen Cell Ontology terms or compute marker genes for all ontology categories — and raw obo/network files are available.

## When NOT to use

- The task only needs classifying into a fixed, flat label set with no ontology vocabulary — a standard classifier (e.g. ACTINN) suffices and graph embedding adds overhead
- Cell types of interest are topologically identical siblings (same parents in the Cell Ontology graph) that must be distinguished — OnClass cannot separate these
- The Cell Ontology lacks terms for the cell states being studied, since it was not developed specifically for scRNA-seq and likely misses cell types and cell states

## Inputs

- Cell Ontology obo file (cl_obo_file) from http://www.obofoundry.org/ontology/cl.html
- Cell type network file (cell_type_network_file) built from 'is_a' hierarchical structure
- Cell type NLP embedding text file (cl.ontology.nlp.emb)
- Training labels: cell-by-Cell-Ontology-ID annotation matrix (train_label)
- Cell-by-gene expression matrices with gene lists (train_feature, train_genes, test_feature, test_genes)

## Outputs

- Low-dimensional cell type embedding matrix (one finite row per ontology term), savable to cell_type_embeddings.npz
- Projected cell feature matrices (cor_train_feature, cor_test_feature) sharing the cell type embedding dimensionality
- Cell type similarity graph from 'is_a' edges used by EmbedCellTypes

## How to apply

Load the ontology artifacts with read_ontology_file('cell ontology', ontology_data_dir) to obtain the cl_obo_file, cell_type_network_file, and cell_type_nlp_emb_file (cl.ontology.nlp.emb text embeddings). Build an undirected similarity graph from the Cell Ontology 'is_a' parent-child edges and instantiate OnClassModel (OnClass 1.2, TensorFlow 2.0, Python 3.6) with the NLP embedding and network files. Call OnClass_train_obj.EmbedCellTypes(train_label) to embed seen cell types via random walk with restart on the ontology graph (restart probability in the 0.5-0.8 range, clusDCA-style) followed by SVD into the low-dimensional cell type embedding matrix. Then call ProcessTrainFeature(...) with the cell-by-gene matrices to project expression into that same embedding space via the nonlinear bilinear transformation, so probabilities can later be extended to unseen terms along the ontology graph.

## Related tools

- **OnClass** (Provides the ontology graph construction and EmbedCellTypes routine (random walk with restart + SVD) that embeds Cell Ontology terms and projects transcriptomes into the shared space) — https://github.com/wangshenguiuc/OnClass
- **OnClass 1.2** (Pinned package version whose OnClassModel consumes cell_type_nlp_emb_file and cell_type_network_file to run the embedding) — https://github.com/wangshenguiuc/OnClass
- **tensorflow 2.0** (Backend dependency for running the OnClassModel embedding and bilinear projection)
- **Python 3.6** (Required runtime per the paper's installation instructions)

## Examples

```
OnClass_train_obj = OnClassModel(cell_type_nlp_emb_file, cell_type_network_file); OnClass_train_obj.EmbedCellTypes(train_label)
```

## Evaluation signals

- Every Cell Ontology ID present in train_label receives a finite, nonzero row in the cell type embedding matrix
- Projected feature matrices (cor_train_feature, cor_test_feature) have the same embedding dimensionality as the cell type embedding matrix
- Undirected similarity graph connectivity matches the 'is_a' parent-child structure extracted from the obo file
- Downstream, OnClass's third step must be able to extend probability scores to unseen cell types using the built graph

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states
- NLP-based mapping of free text to ontology terms is limited; the authors plan to improve NLP models to also consider the Cell Ontology graph structure

## Evidence

- [methods] ontology structure from obo file: "we use the hierarchical ontology structure from the `Cell Ontology <http://www.obofoundry.org/ontology/cl.html>`__"
- [methods] embedding entry point: "OnClass_train_obj.EmbedCellTypes(train_label)"
- [intro] shared low-dimensional space rationale: "by projecting single-cell transcriptomes and the Cell Ontology into the same low-dimensional space, OnClass advances other important applications, such as robust computation of marker genes"
- [discussion] graph used to extend scores to unseen terms: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [readme] version pinning: "Update on July 4th, 2021. OnClass is now upgraded to 1.2 version!"
