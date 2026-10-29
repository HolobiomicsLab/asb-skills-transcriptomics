---
name: maximum-weight-bipartite-matching
description: Use when a set of uncontrolled free-text cell type annotations (synonyms,
  misspellings, abbreviations) must be converted one-to-one into Cell Ontology terms,
  and per-label nearest-neighbor retrieval alone may yield inconsistent or duplicated
  assignments.
when_to_use_negative:
- Cell labels are already controlled Cell Ontology term IDs — no NLP mapping or matching
  refinement is needed
- Cell types are absent from the Cell Ontology entirely (the Cell Ontology 'likely
  misses cell types and cell states'), since no valid CL target term exists for matching
- The analysis goal is classifying cells by expression (BuildModel/Train on single-cell
  transcriptomes) rather than harmonizing label terminology
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: OnClass
  role: Implements the char-level LSTM siamese network that embeds cell type text
    and computes similarities refined by maximum-weight bipartite matching to map
    free text to Cell Ontology terms
  repo: https://github.com/wangshenguiuc/OnClass
- name: OnClass 1.2
  role: Pinned pip-installable version of OnClass providing the NLP mapping functionality
    (pip install OnClass==1.2)
  repo: https://github.com/wangshenguiuc/OnClass
- name: tensorflow 2.0
  role: Deep learning backend for the char-level LSTM embedding network used to compute
    label-to-term similarities
- name: Python 3.6
  role: Runtime environment required by OnClass for executing the mapping pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:7276463fd1e8383f31689fdbdac316c61a971f52f52a3dd61139e567de3beb1f
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# maximum-weight-bipartite-matching

## Summary

Global assignment step used inside OnClass's NLP terminology-mapping stage: after computing text-embedding similarities between user free-text cell type labels and Cell Ontology term definitions, maximum-weight bipartite matching pairs each free-text label to its best overall controlled-vocabulary CL term.

## When to use

Use when a set of uncontrolled free-text cell type annotations (synonyms, misspellings, abbreviations) must be converted one-to-one into Cell Ontology terms, and per-label nearest-neighbor retrieval alone may yield inconsistent or duplicated assignments.

## When NOT to use

- Cell labels are already controlled Cell Ontology term IDs — no NLP mapping or matching refinement is needed
- Cell types are absent from the Cell Ontology entirely (the Cell Ontology 'likely misses cell types and cell states'), since no valid CL target term exists for matching
- The analysis goal is classifying cells by expression (BuildModel/Train on single-cell transcriptomes) rather than harmonizing label terminology

## Inputs

- Free-text cell type annotations from user datasets (e.g., Tabula Muris Senis labels, including synonyms, misspellings, and abbreviations)
- Cell Ontology NLP text embeddings of cell type definitions (cl.ontology.nlp.emb) and the cell type network
- Small hand-mapped ground-truth set of free-text to Cell Ontology term pairs for validation

## Outputs

- cell_type_nlp_mapping_table.csv containing accepted free-text-to-Cell-Ontology-term pairs
- Per-label mapping accuracy against the hand-mapped ground-truth label set

## How to apply

First embed cell types into the low-dimensional text space by calling OnClass_train_obj.EmbedCellTypes(train_label), which uses the char-level LSTM siamese network over the cl.ontology.nlp.emb text embeddings of cell type definitions. For each free-text annotation (e.g., Tabula Muris Senis labels), compute similarity against all Cell Ontology term definitions and retrieve the nearest controlled-vocabulary term candidates. Instead of accepting the greedy nearest term per label, run a maximum-weight bipartite matching over the label-by-term similarity graph so each free-text label is paired to its best overall Cell Ontology term globally, resolving conflicts where multiple labels compete for the same CL term. Score the resulting mapping against a small hand-mapped ground-truth label set; accept the mapping if accuracy reaches the reported 93.7% char-level LSTM benchmark for mapping synonyms, misspellings, and abbreviations. Write accepted free-text-to-CL-term pairs to cell_type_nlp_mapping_table.csv for downstream OnClass annotation steps.

## Related tools

- **OnClass** (Implements the char-level LSTM siamese network that embeds cell type text and computes similarities refined by maximum-weight bipartite matching to map free text to Cell Ontology terms) — https://github.com/wangshenguiuc/OnClass
- **OnClass 1.2** (Pinned pip-installable version of OnClass providing the NLP mapping functionality (pip install OnClass==1.2)) — https://github.com/wangshenguiuc/OnClass
- **tensorflow 2.0** (Deep learning backend for the char-level LSTM embedding network used to compute label-to-term similarities)
- **Python 3.6** (Runtime environment required by OnClass for executing the mapping pipeline)

## Examples

```
OnClass_train_obj.EmbedCellTypes(train_label)
```

## Evaluation signals

- Mapping accuracy on the hand-mapped ground-truth label set reaches the reported 93.7% char-level LSTM benchmark for mapping synonyms of Cell Ontology terms
- The output cell_type_nlp_mapping_table.csv assigns exactly one Cell Ontology term per free-text label (a valid one-to-one matching with no duplicated targets)
- Mapped terms exist in the Cell Ontology graph loaded from cl.ontology.nlp.emb and its cell type network
- Spot-check that known synonyms, misspellings, and abbreviations (e.g., Tabula Muris Senis label variants) resolve to the expected CL terms
- Setting nlp_mapping = True during data reading correctly activates the Char-level LSTM uncontrolled-to-controlled vocabulary mapping path

## Limitations

- OnClass's NLP model maps only text data; the authors plan to further improve it to consider the Cell Ontology graph structure as well (future work, not yet implemented)
- Automated mapping of free-text annotations to the Cell Ontology is a proposed follow-up in the paper, with current NLP model improvement remaining future work
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states, so some free-text labels may have no correct CL target

## Evidence

- [discussion] NLP-based mapping of free text annotations to Cell Ontology terms: "Here we already offer an NLP model that is able to map free text annotations to Cell Ontology terms"
- [intro] Low-dimensional embedding underpins the similarity computation used for matching: "by projecting single-cell transcriptomes and the Cell Ontology into the same low-dimensional space"
- [methods] Cell type embedding step that precedes matching: "Embedding Cell ontology - OnClass_train_obj.EmbedCellTypes(train_label)"
- [readme] Package version grounding for the OnClass implementation: "Update on July 4th, 2021. OnClass is now upgraded to 1.2 version!"
