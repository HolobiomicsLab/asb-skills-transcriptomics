---
name: ontology-graph-distance-traversal
description: Use when you need to quantify how similar cell types are according to
  ontology structure (direct-neighbor vs multi-hop graph distance) and test whether
  ontology similarity tracks expression similarity.
when_to_use_negative:
- When the goal is annotating/query cells with a trained OnClass model rather than
  characterizing ontology structure itself (use OnClass prediction workflow instead)
- When the cell type vocabulary has no formal ontology (e.g., free-text labels only)
  — traversal requires the cl.obo graph structure
edam_topics: []
tools:
- name: OnClass
  role: Provides the Cell Ontology files (cl.obo, cl.ontology.nlp.emb) and the ontology
    graph over which distances are traversed
  repo: https://github.com/wangshenguiuc/OnClass
- name: pip
  role: Installs the OnClass Python package used to access ontology data
- name: git
  role: Clones the OnClass repository containing ontology files and example scripts
  repo: https://github.com/wangshenguiuc/OnClass
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:d759c5ab0e09d6b0eb1b60901948c06337661648688d1d08f5c670515463cde3
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# ontology-graph-distance-traversal

## Summary

Computes cell-type similarity matrices by traversing the Cell Ontology hierarchical graph (treating edges as undirected similarities with uniform weights) and validates them against text-embedding and gene-expression similarity, as done for OnClass on Tabula Muris Senis lung and pancreas data.

## When to use

Use when you need to quantify how similar cell types are according to ontology structure (direct-neighbor vs multi-hop graph distance) and test whether ontology similarity tracks expression similarity.

## When NOT to use

- When the goal is annotating/query cells with a trained OnClass model rather than characterizing ontology structure itself (use OnClass prediction workflow instead)
- When the cell type vocabulary has no formal ontology (e.g., free-text labels only) — traversal requires the cl.obo graph structure

## Inputs

- Cell Ontology OBO file (cl.obo) defining the hierarchical cell-type graph
- cl.ontology.nlp.emb text embeddings of cell type definitions
- Single-cell gene expression matrices (Tabula Muris Senis lung and pancreas, FACS cells) from figshare scRNA_data parts
- Cell type label assignments for each cell

## Outputs

- Ontology graph-based cell type similarity matrix
- Text-embedding-based similarity matrix (direct-neighbor and four-hop variants)
- Gene-expression-based similarity matrices for lung and pancreas
- Pearson correlation values written to cell_type_similarity_results.csv

## How to apply

Download the Cell Ontology files (cl.obo and cl.ontology.nlp.emb) from the OnClass_data_public_minimal.tar.gz figshare deposit and the Tabula Muris Senis lung/pancreas scRNA-seq data from the figshare scRNA_data parts. Traverse the hierarchical ontology structure treating edges as undirected similarities with uniform weights to build an ontology-based similarity matrix. Build a text-description similarity matrix from the cl.ontology.nlp.emb embeddings of cell type definitions, evaluating similarity at direct-neighbor and four-hop graph distances. Separately compute gene-expression similarity matrices for lung and pancreas by correlating expression profiles across cell types, then compute Pearson correlations between the expression-based and ontology-based matrices (reported: 0.65 lung, 0.93 pancreas) and between text-based and ontology-based matrices (0.67 direct-neighbor vs 0.49 four-hop). Rationale: this quantifies the guilt-by-association principle that underlies OnClass's extension of scores to unseen cell types via the Cell Ontology graph. Implement with numpy/scipy in Python and save the three similarity matrices plus correlation values to cell_type_similarity_results.csv.

## Related tools

- **OnClass** (Provides the Cell Ontology files (cl.obo, cl.ontology.nlp.emb) and the ontology graph over which distances are traversed) — https://github.com/wangshenguiuc/OnClass
- **pip** (Installs the OnClass Python package used to access ontology data)
- **git** (Clones the OnClass repository containing ontology files and example scripts) — https://github.com/wangshenguiuc/OnClass

## Examples

```
pip install OnClass==1.2; git clone https://github.com/wangshenguiuc/OnClass.git
```

## Evaluation signals

- Pearson correlation between gene-expression and ontology-based similarity matrices falls in the reported range: 0.65 for lung and 0.93 for pancreas
- Pearson correlation between text-embedding and ontology similarity is higher at direct-neighbor distance (0.67) than at four-hop distance (0.49)
- The three similarity matrices are square and indexed over the same set of Cell Ontology terms present in cl.obo
- cell_type_similarity_results.csv contains all three matrices plus the correlation values as specified in the workflow

## Limitations

- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states
- OnClass cannot distinguish topologically identical unseen cell types (siblings with the same parents in the Cell Ontology graph), which limits graph-distance-based similarity resolution
- Text-embedding similarity degrades with graph distance (0.67 at direct-neighbor vs 0.49 at four-hop), so multi-hop similarity should be interpreted cautiously

## Evidence

- [discussion] Ontology graph provides similarity basis for OnClass: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [methods] Hierarchical ontology structure source: "we use the hierarchical ontology structure from the `Cell Ontology <http://www.obofoundry.org/ontology/cl.html>`__"
- [intro] Guilt-by-association validated by expression correlation: "by projecting single-cell transcriptomes and the Cell Ontology into the same low-dimensional space, OnClass advances other important applications, such as robust computation of marker genes"
- [readme] OnClass documentation entry point: "Please see the document of OnClass at https://onclass.readthedocs.io/en/latest/"
