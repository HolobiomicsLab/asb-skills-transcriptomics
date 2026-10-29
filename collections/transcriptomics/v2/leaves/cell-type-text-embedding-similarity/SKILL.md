---
name: cell-type-text-embedding-similarity
description: Use when you have Cell Ontology text embeddings (cl.ontology.nlp.emb),
  the ontology graph (cl.obo), and per-cell-type expression profiles, and you want
  to test whether ontology/text/expression similarity agree.
when_to_use_negative:
- Your goal is to distinguish topologically identical unseen cell types (siblings
  sharing the same parents in the Cell Ontology graph) — both graph-based and text-embedding
  similarities cannot separate them
- Your input is free-text cell annotations that first need mapping to Cell Ontology
  terms — use OnClass's NLP mapping model instead, not similarity analysis
- You lack text embeddings (cl.ontology.nlp.emb) or ontology structure and only have
  raw expression clusters — direct expression clustering is more appropriate
edam_topics:
- http://edamontology.org/topic_0089
tools:
- name: OnClass
  role: Source of the Cell Ontology files (cl.obo, cl.ontology.nlp.emb) whose graph
    structure and text embeddings are compared against expression-based similarity
  repo: https://github.com/wangshenguiuc/OnClass
- name: Python
  role: Implementation language for computing the three similarity matrices and Pearson
    correlations with numpy/scipy
  repo: https://github.com/wangshenguiuc/OnClass
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:a871b0aa4e9bb73491f34d9763e9b220ea7816b78e2b3256e1d1d62458f19619
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - TensorFlow
  - git
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cell-type-text-embedding-similarity

## Summary

Quantifies how similar cell types are in NLP text-embedding space (cl.ontology.nlp.emb), Cell Ontology graph space, and gene-expression space, by building pairwise similarity matrices and computing Pearson correlations between them. It validates the guilt-by-association principle underlying OnClass.

## When to use

Use when you have Cell Ontology text embeddings (cl.ontology.nlp.emb), the ontology graph (cl.obo), and per-cell-type expression profiles, and you want to test whether ontology/text/expression similarity agree.

## When NOT to use

- Your goal is to distinguish topologically identical unseen cell types (siblings sharing the same parents in the Cell Ontology graph) — both graph-based and text-embedding similarities cannot separate them
- Your input is free-text cell annotations that first need mapping to Cell Ontology terms — use OnClass's NLP mapping model instead, not similarity analysis
- You lack text embeddings (cl.ontology.nlp.emb) or ontology structure and only have raw expression clusters — direct expression clustering is more appropriate

## Inputs

- Cell Ontology OBO file (cl.obo) defining the hierarchical graph of cell type terms
- Cell Ontology NLP text embeddings (cl.ontology.nlp.emb) from the OnClass_data_public_minimal.tar.gz deposit
- Tabula Muris Senis lung and pancreas single-cell gene expression matrices (figshare scRNA_data parts)
- Cell type label assignments per cell for aggregating expression profiles per term

## Outputs

- Cell Ontology graph-based similarity matrix (cell types x cell types)
- Text-embedding-based similarity matrix at direct-neighbor and four-hop distances
- Gene-expression-based similarity matrices for lung and pancreas
- cell_type_similarity_results.csv containing the three similarity matrices and Pearson correlation values

## How to apply

Download the Cell Ontology files (cl.obo and cl.ontology.nlp.emb) from the OnClass_data_public_minimal.tar.gz figshare deposit and the Tabula Muris Senis lung and pancreas scRNA-seq matrices from the figshare scRNA_data parts. Build three cell-type-by-cell-type similarity matrices: (1) ontology graph similarity by traversing the hierarchical cl.obo structure with undirected edges and uniform weights; (2) text similarity from cl.ontology.nlp.emb embeddings, evaluated at direct-neighbor and four-hop graph distances; (3) expression similarity by correlating cell-type-level expression profiles for lung and pancreas separately. Compute Pearson correlations with numpy/scipy between the ontology matrix and both text matrices, and between expression and ontology matrices per tissue, writing matrices and correlations to cell_type_similarity_results.csv. Correct application reproduces correlations of 0.67 (direct-neighbor) vs 0.49 (four-hop) for text-vs-ontology, and 0.65 (lung) / 0.93 (pancreas) for expression-vs-ontology.

## Related tools

- **OnClass** (Source of the Cell Ontology files (cl.obo, cl.ontology.nlp.emb) whose graph structure and text embeddings are compared against expression-based similarity) — https://github.com/wangshenguiuc/OnClass
- **Python** (Implementation language for computing the three similarity matrices and Pearson correlations with numpy/scipy) — https://github.com/wangshenguiuc/OnClass

## Examples

```
import numpy as np; from scipy.stats import pearsonr; r = pearsonr(ont_sim.flatten(), expr_sim.flatten())  # after building matrices from cl.obo, cl.ontology.nlp.emb, and Tabula Muris expression
```

## Evaluation signals

- Pearson correlation between text-based (direct-neighbor) and ontology-based similarity matrices reproduces the reported value of ~0.67
- Pearson correlation between text-based (four-hop) and ontology-based similarity matrices reproduces ~0.49
- Expression-vs-ontology correlation reproduces ~0.65 for Tabula Muris Senis lung and ~0.93 for pancreas
- All three similarity matrices are symmetric with unit diagonal and dimensions equal to the number of Cell Ontology terms analyzed
- Results file cell_type_similarity_results.csv contains both the matrices and the correlation values

## Limitations

- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states
- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- Text-vs-ontology similarity decays at longer graph distances (0.49 at four hops vs 0.67 at direct neighbors), so text embeddings best capture only local ontology relationships

## Evidence

- [methods] Text-embedding similarity vs ontology graph structure: [evidence span withheld — anchor status: not_found]
- [discussion] OnClass ontology graph usage: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [abstract] OnClass software description: "we present OnClass, an algorithm and accompanying software for automatically classifying cells into cell types that are part of the controlled vocabulary"
- [discussion] Implementation language: "The algorithm is implemented in Python (https://github.com/wangshenguiuc/OnClass) and it has been included in PyPy"
- [discussion] OnClass limitation on sibling terms: "Another limitation of OnClass is that it cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph."
