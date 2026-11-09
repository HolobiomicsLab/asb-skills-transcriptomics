---
name: cross-modal-similarity-validation
description: Use when you need to check that structurally-derived relationships (ontology
  graph distances) are recapitulated by an independent data modality (gene expression
  or text embeddings) before trusting the structure for downstream inference.
when_to_use_negative:
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses
  cell types and cell states, so do not treat expression-vs-ontology correlation as
  a completeness check of the ontology itself
- OnClass-style ontology structure cannot distinguish topologically identical unseen
  cell types (siblings with the same parents), so similarity based purely on graph
  position will saturate for such siblings regardless of expression data
- Avoid mixing tissues into a single expression similarity matrix — correlations are
  computed separately per tissue (lung vs pancreas) because tissue context changes
  expression profiles
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0089
tools:
- name: OnClass
  role: Provides the Cell Ontology files (cl.obo, cl.ontology.nlp.emb) and ontology-embedding
    machinery used to derive graph- and text-based similarity matrices
  repo: https://github.com/wangshenguiuc/OnClass
- name: Python 3.6
  role: Runtime environment in which the similarity matrices and Pearson correlations
    are computed
  repo: https://github.com/wangshenguiuc/OnClass
- name: pip
  role: Installs OnClass 1.2 and tensorflow 2.0 dependencies for loading ontology
    embeddings
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:88ffaecc95151ad7005d0ae6d414b863ec8b98be11503a38a26a58bc84cdcb48
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
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Cross-modal similarity validation

## Summary

Quantitatively validates whether two independent representations of the same entities (e.g., Cell Ontology graph structure, NLP text embeddings of cell type definitions, and scRNA-seq expression profiles) agree, by computing per-modality similarity matrices and correlating them with Pearson correlation. As demonstrated in the OnClass paper, this tests the guilt-by-association principle underlying ontology-based cell type classification.

## When to use

Use when you need to check that structurally-derived relationships (ontology graph distances) are recapitulated by an independent data modality (gene expression or text embeddings) before trusting the structure for downstream inference.

## When NOT to use

- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states, so do not treat expression-vs-ontology correlation as a completeness check of the ontology itself
- OnClass-style ontology structure cannot distinguish topologically identical unseen cell types (siblings with the same parents), so similarity based purely on graph position will saturate for such siblings regardless of expression data
- Avoid mixing tissues into a single expression similarity matrix — correlations are computed separately per tissue (lung vs pancreas) because tissue context changes expression profiles

## Inputs

- Cell Ontology OBO file (cl.obo) defining the hierarchical cell type graph
- Cell Ontology NLP text embeddings (cl.ontology.nlp.emb) from the OnClass_data_public_minimal.tar.gz figshare deposit
- Single-cell gene expression matrices for Tabula Muris Senis lung and pancreas (figshare scRNA_data parts)
- Cell type label assignments mapping cells to Cell Ontology terms

## Outputs

- Ontology graph-based cell type similarity matrix
- Text-embedding-based cell type similarity matrix (direct-neighbor and four-hop variants)
- Gene-expression-based cell type similarity matrices (one per tissue)
- Pearson correlation values between modality pairs (e.g., 0.67 direct-neighbor vs 0.49 four-hop text; 0.65 lung and 0.93 pancreas expression-vs-ontology)
- CSV of matrices and correlations (cell_type_similarity_results.csv)

## How to apply

Build one similarity matrix per modality over the same set of entities. For the ontology modality, traverse the Cell Ontology hierarchical structure from cl.obo, treating edges as undirected with uniform weights. For the text modality, compute cosine similarities between cl.ontology.nlp.emb embeddings of cell type definitions, evaluating at both direct-neighbor and four-hop graph distances. For the expression modality, correlate aggregate expression profiles across cell types separately per tissue (Tabula Muris Senis lung and pancreas). Finally compute Pearson correlations between matrix pairs with numpy/scipy, flatten upper triangles appropriately, and report values per tissue and per graph-distance condition.

## Related tools

- **OnClass** (Provides the Cell Ontology files (cl.obo, cl.ontology.nlp.emb) and ontology-embedding machinery used to derive graph- and text-based similarity matrices) — https://github.com/wangshenguiuc/OnClass
- **Python 3.6** (Runtime environment in which the similarity matrices and Pearson correlations are computed) — https://github.com/wangshenguiuc/OnClass
- **pip** (Installs OnClass 1.2 and tensorflow 2.0 dependencies for loading ontology embeddings)

## Examples

```
pip install OnClass==1.2 tensorflow==2.0; python compute_similarity_correlations.py --obo cl.obo --emb cl.ontology.nlp.emb --tissue lung --tissue pancreas --out cell_type_similarity_results.csv
```

## Evaluation signals

- Pearson correlation between expression-based and ontology-based similarity should fall in the reported range: 0.65 for Tabula Muris Senis lung and 0.93 for pancreas
- Text-embedding similarity correlation with ontology graph similarity should be higher at direct-neighbor distance (0.67) than at four-hop distance (0.49)
- All three similarity matrices must be square, symmetric, and indexed by the same set of Cell Ontology terms
- Reproduced correlation values must match the reported values to within tolerance for the result to be accepted
- Output CSV (cell_type_similarity_results.csv) must contain the three similarity matrices and all correlation values

## Limitations

- The Cell Ontology was not developed specifically for scRNA-seq and, therefore, likely misses cell types and cell states
- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- Ontology-graph similarity uses uniform edge weights and undirected traversal, which is a simplification of true semantic distance

## Evidence

- [results] Cell types deemed similar by the Cell Ontology show strongly correlated gene expression profiles: the correlation between gene expression-based similarity and Cell Ontology graph-based similarity was 0.65 in lung and 0.93 in pancreas: [evidence span withheld — anchor status: not_found]
- [methods] Ontology similarity construction: "we use the hierarchical ontology structure from the `Cell Ontology <http://www.obofoundry.org/ontology/cl.html>`__"
- [abstract] Ontology graph drives downstream inference: "A key advantage of OnClass is its capability to classify cells into cell types not present in the training data because it uses the Cell Ontology graph to infer cell type relationships."
- [discussion] Implementation availability: "The algorithm is implemented in Python (https://github.com/wangshenguiuc/OnClass) and it has been included in PyPy to facilitate its integration with current workflows."
- [methods] Install commands for the toolchain: "pip install OnClass==1.2"
