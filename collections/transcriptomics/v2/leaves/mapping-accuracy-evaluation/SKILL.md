---
name: mapping-accuracy-evaluation
description: Use when a scRNA-seq dataset carries free-text cell type annotations
  that must be converted to Cell Ontology terms, and you need to verify the text-to-ontology
  mapping accuracy before trusting downstream annotation.
when_to_use_negative:
- Annotations are already valid Cell Ontology term IDs, so no text-to-ontology mapping
  is needed
- The goal is annotating cells to unseen Cell Ontology terms from expression profiles
  — that is the classification step, not the terminology mapping step
- Cell types or cell states of interest are absent from the Cell Ontology, since the
  ontology was not developed specifically for scRNA-seq and likely misses them
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: OnClass
  role: Performs text description-based similarity and bipartite matching to map free-text
    cell type labels to Cell Ontology terms; benchmarked via the hand-mapped label
    set
  repo: https://github.com/wangshenguiuc/OnClass
- name: OnClass 1.2
  role: Pinned pip-installable version containing the NLP mapping functionality and
    char-level LSTM model
  repo: https://github.com/wangshenguiuc/OnClass
- name: tensorflow 2.0
  role: Backend for the char-level LSTM siamese network that embeds cell type descriptions
- name: Python 3.6
  role: Required runtime for OnClass 1.2 and the mapping workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:81ba393a64bb201e054bd91ac7845a9b0140be81539cef7e5bc57bcc410bd47e
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# mapping-accuracy-evaluation

## Summary

Evaluates how accurately OnClass maps uncontrolled free-text cell type labels onto Cell Ontology terms, using the char-level LSTM siamese network and benchmarking against the hand-mapped ground-truth label set (e.g., Tabula Muris Senis annotations) at the paper-reported 93.7% accuracy for synonyms, misspellings, and abbreviations.

## When to use

Use when a scRNA-seq dataset carries free-text cell type annotations that must be converted to Cell Ontology terms, and you need to verify the text-to-ontology mapping accuracy before trusting downstream annotation.

## When NOT to use

- Annotations are already valid Cell Ontology term IDs, so no text-to-ontology mapping is needed
- The goal is annotating cells to unseen Cell Ontology terms from expression profiles — that is the classification step, not the terminology mapping step
- Cell types or cell states of interest are absent from the Cell Ontology, since the ontology was not developed specifically for scRNA-seq and likely misses them

## Inputs

- Free-text cell type annotations from user datasets (e.g., Tabula Muris Senis labels)
- cl.ontology.nlp.emb — char-level LSTM text embeddings of Cell Ontology term definitions
- Cell type network derived from the Cell Ontology (http://www.obofoundry.org/ontology/cl.html)
- Small hand-mapped free-text-to-CL-term ground-truth label set

## Outputs

- cell_type_nlp_mapping_table.csv — accepted free-text-to-Cell-Ontology-term pairs
- Mapping accuracy score against the hand-mapped ground truth (benchmark: 93.7% for char-level LSTM on synonyms/misspellings/abbreviations)

## How to apply

Load the Cell Ontology NLP assets (cl.ontology.nlp.emb text embeddings of cell type definitions plus the cell type network) and a small hand-mapped set of free-text annotations such as Tabula Muris Senis labels. Embed cell types into the low-dimensional space with OnClass_train_obj.EmbedCellTypes(train_label), which invokes the char-level LSTM siamese network. For each free-text annotation, compute text description-based similarity against all Cell Ontology term definitions, retrieve the nearest controlled-vocabulary term, then refine candidates with a maximum-weight bipartite matching so each free-text label pairs to its best overall Cell Ontology term. Enable the mapping by passing nlp_mapping = True to ProcessTrainFeature. Score the resulting pairs against the hand-mapped ground truth; accept the mapping if accuracy reaches the reported 93.7% for synonyms of Cell Ontology terms, and write accepted pairs to cell_type_nlp_mapping_table.csv.

## Related tools

- **OnClass** (Performs text description-based similarity and bipartite matching to map free-text cell type labels to Cell Ontology terms; benchmarked via the hand-mapped label set) — https://github.com/wangshenguiuc/OnClass
- **OnClass 1.2** (Pinned pip-installable version containing the NLP mapping functionality and char-level LSTM model) — https://github.com/wangshenguiuc/OnClass
- **tensorflow 2.0** (Backend for the char-level LSTM siamese network that embeds cell type descriptions)
- **Python 3.6** (Required runtime for OnClass 1.2 and the mapping workflow)

## Examples

```
OnClass_train_obj.ProcessTrainFeature(train_feature, train_label, train_genes, nlp_mapping=True)  # Char-level LSTM maps uncontrolled to controlled vocabulary
```

## Evaluation signals

- Mapping accuracy on the hand-mapped label set reaches the paper-reported 93.7% char-level LSTM benchmark for synonyms, misspellings, and abbreviations
- Every free-text label in the input set appears exactly once in cell_type_nlp_mapping_table.csv with a valid Cell Ontology term (bijective matching, not many-to-many)
- The maximum-weight bipartite matching improves (or at least does not degrade) total similarity score relative to greedy nearest-term assignment
- process runs with nlp_mapping = True confirmed, so the Char-level LSTM is engaged rather than raw label passthrough
- cl.ontology.nlp.emb embeddings load without error and cover the Cell Ontology terms queried during similarity search

## Limitations

- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states, so some free-text labels may have no correct target term
- Automated mapping of free text is largely a proposed follow-up: the authors plan to further improve NLP models to consider not only text data but also the Cell Ontology graph structure
- Mapping quality depends on the hand-mapped ground truth (e.g., Tabula Muris Senis labels); limited hand-curated pairs restrict evaluation coverage

## Evidence

- [discussion] NLP-based mapping of free text annotations to Cell Ontology terms: "Here we already offer an NLP model that is able to map free text annotations to Cell Ontology terms"
- [methods] Enable char-level LSTM mapping via nlp_mapping flag: "Read training and test data. Set nlp_mapping = True to use the Char-level LSTM that maps uncontrolled vocabulary to controlled vocabulary."
- [methods] OnClass available via pip and GitHub with example script: "We provide a `run_OnClass_example.py <https://github.com/wangshenguiuc/OnClass/blob/master/run_OnClass_example.py>`__"
- [readme] OnClass version pinned for reproduction: "Update on July 4th, 2021. OnClass is now upgraded to 1.2 version!"
- [methods] Ontology embedding step used before similarity mapping: "OnClass_train_obj.EmbedCellTypes(train_label)"
