---
name: cell-ontology-graph-similarity-computation
description: Use when you need to test whether ontology-defined cell type relatedness
  (Cell Ontology graph structure) predicts expression-profile or text-description
  similarity, or to validate ontology-based annotation methods like OnClass.
when_to_use_negative:
- The goal is cell type annotation of a new scRNA-seq dataset rather than characterizing
  ontology-vs-expression concordance — use the OnClass classification pipeline (ProcessTrainFeature,
  BuildModel, Train) instead.
- No Cell Ontology term mapping exists for the cell types in the expression data,
  so graph-based similarity cannot be traversed.
- Comparing topologically identical sibling cell types (same parents in the Cell Ontology
  graph), which OnClass-related ontology structure cannot distinguish.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0605
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0089
tools:
- name: OnClass
  role: Provides the Cell Ontology files (cl.obo, cl.ontology.nlp.emb) and the ontology
    graph structure used to compute graph-based cell type similarity
  repo: https://github.com/wangshenguiuc/OnClass
- name: Python 3.6
  role: Implementation environment for computing the three similarity matrices and
    Pearson correlations with numpy/scipy
  repo: https://github.com/wangshenguiuc/OnClass
- name: pip
  role: Installs OnClass (pip install OnClass==1.2) and dependencies used to obtain
    the ontology data
- name: git
  role: Clones the OnClass repository to access ontology data and example scripts
  repo: https://github.com/wangshenguiuc/OnClass
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:7f9852ef866055531e934e211ddb2124606a2ab06397291840852eb455ff4685
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - Python 3.6
  - TensorFlow
  - git
  - pip
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0605
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# cell-ontology-graph-similarity-computation

## Summary

Quantify how similar cell types are according to the Cell Ontology graph, text embeddings of cell type definitions (cl.ontology.nlp.emb), and gene expression profiles, by building three pairwise similarity matrices and computing Pearson correlations between them (e.g., 0.65 lung / 0.93 pancreas for expression-vs-ontology). This validates the guilt-by-association principle underlying OnClass.

## When to use

Use when you need to test whether ontology-defined cell type relatedness (Cell Ontology graph structure) predicts expression-profile or text-description similarity, or to validate ontology-based annotation methods like OnClass.

## When NOT to use

- The goal is cell type annotation of a new scRNA-seq dataset rather than characterizing ontology-vs-expression concordance — use the OnClass classification pipeline (ProcessTrainFeature, BuildModel, Train) instead.
- No Cell Ontology term mapping exists for the cell types in the expression data, so graph-based similarity cannot be traversed.
- Comparing topologically identical sibling cell types (same parents in the Cell Ontology graph), which OnClass-related ontology structure cannot distinguish.

## Inputs

- Cell Ontology OBO file (cl.obo) from OnClass_data_public_minimal.tar.gz
- Cell type definition text embeddings (cl.ontology.nlp.emb)
- Tabula Muris Senis lung and pancreas single-cell gene expression matrices (figshare scRNA_data parts)

## Outputs

- Ontology graph-based cell type similarity matrix
- Text-embedding-based similarity matrix (direct-neighbor and four-hop distances)
- Gene-expression-based similarity matrices for lung and pancreas
- Pearson correlation values saved to cell_type_similarity_results.csv

## How to apply

Download the Cell Ontology files (cl.obo and cl.ontology.nlp.emb) from the OnClass_data_public_minimal.tar.gz figshare deposit and the Tabula Muris Senis lung and pancreas scRNA-seq data from the figshare scRNA_data parts. Compute the Cell Ontology graph-based similarity matrix by traversing the hierarchical ontology structure, treating edges as undirected similarities with uniform weights. Compute text-description-based similarity from the cl.ontology.nlp.emb embeddings at both direct-neighbor and four-hop graph distances, and compute expression-based similarity by correlating per-cell-type expression profiles for lung and pancreas separately. Finally, compute Pearson correlations with numpy/scipy between the ontology matrix and each text matrix, and between the expression and ontology matrices, saving matrices and values to cell_type_similarity_results.csv.

## Related tools

- **OnClass** (Provides the Cell Ontology files (cl.obo, cl.ontology.nlp.emb) and the ontology graph structure used to compute graph-based cell type similarity) — https://github.com/wangshenguiuc/OnClass
- **Python 3.6** (Implementation environment for computing the three similarity matrices and Pearson correlations with numpy/scipy) — https://github.com/wangshenguiuc/OnClass
- **pip** (Installs OnClass (pip install OnClass==1.2) and dependencies used to obtain the ontology data)
- **git** (Clones the OnClass repository to access ontology data and example scripts) — https://github.com/wangshenguiuc/OnClass

## Examples

```
pip install OnClass==1.2 && python -c "import numpy, scipy; from scipy.stats import pearsonr; r = pearsonr(ontology_sim.flatten(), expression_sim.flatten())"
```

## Evaluation signals

- Pearson correlation between text-embedding similarity and ontology graph similarity should approximate the reported 0.67 at direct-neighbor distance vs 0.49 at four-hop distance.
- Correlation between expression-based and ontology-based similarity should reproduce ~0.65 in lung and ~0.93 in pancreas.
- The three similarity matrices should be saved and match the dimensions of the cell type set from cl.obo.
- All correlation values and matrices should be present in cell_type_similarity_results.csv.
- Ontology graph traversal treats edges as undirected, so the resulting similarity matrix must be symmetric.

## Limitations

- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states.
- Topologically identical unseen cell types (siblings with the same parents in the Cell Ontology graph) cannot be distinguished by ontology structure.
- Treating ontology edges as uniform undirected similarities is a simplification of the hierarchical structure; text similarity degrades at greater graph distances (0.67 direct-neighbor vs 0.49 four-hop).

## Evidence

- [results] Cell types deemed similar by the Cell Ontology show strongly correlated gene expression profiles: [evidence span withheld — anchor status: not_found]
- [abstract] OnClass uses Cell Ontology graph to infer cell type relationships: "A key advantage of OnClass is its capability to classify cells into cell types not present in the training data because it uses the Cell Ontology graph to infer cell type relationships."
- [discussion] OnClass implemented in Python: "The algorithm is implemented in Python (https://github.com/wangshenguiuc/OnClass) and it has been included in PyPy to facilitate its integration with current workflows."
- [methods] Hierarchical ontology structure from Cell Ontology: "we use the hierarchical ontology structure from the `Cell Ontology <http://www.obofoundry.org/ontology/cl.html>`__"
- [discussion] Cell Ontology misses cell types and states: "One current limitation of using the Cell Ontology is that it was not developed specifically for scRNA-seq and, therefore, likely misses cell types and cell states."
