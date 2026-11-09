---
name: cell-type-term-mapping
description: Use when scRNA-seq datasets carry heterogeneous free-text cell annotations
  (e.g., Tabula Muris Senis-style labels) that must be normalized to Cell Ontology
  terms before ontology-based classification or cross-dataset annotation.
when_to_use_negative:
- Input cell labels are already valid Cell Ontology term IDs, so no uncontrolled-vocabulary
  normalization is needed
- The goal is classifying cells (extending probability scores to unseen cell types
  via the Cell Ontology graph) rather than normalizing label terminology
- The task requires distinguishing topologically identical unseen cell types (siblings
  with the same parents in the Cell Ontology graph), which OnClass cannot do
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: OnClass
  role: Provides the char-level LSTM NLP model that embeds cell type descriptions
    and maps free-text annotations to Cell Ontology terms
  repo: https://github.com/wangshenguiuc/OnClass
- name: OnClass 1.2
  role: Pinned pip-installable version of the OnClass software used for the NLP terminology
    mapping
  repo: https://github.com/wangshenguiuc/OnClass
- name: tensorflow 2.0
  role: Backend for the char-level LSTM siamese network used to embed cell type text
- name: Python 3.6
  role: Runtime environment required to run OnClass NLP mapping
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:41db5d51ca23beab505ca6c0449f6a3ad738a22882505e8de84a1216c76643ae
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# cell-type-term-mapping

## Summary

Maps uncontrolled free-text cell type labels (with synonyms, misspellings, and abbreviations) onto Cell Ontology terms using OnClass's char-level LSTM siamese network, which embeds cell type descriptions into a low-dimensional space and retrieves the nearest controlled-vocabulary term by text similarity plus maximum-weight bipartite matching.

## When to use

Use when scRNA-seq datasets carry heterogeneous free-text cell annotations (e.g., Tabula Muris Senis-style labels) that must be normalized to Cell Ontology terms before ontology-based classification or cross-dataset annotation.

## When NOT to use

- Input cell labels are already valid Cell Ontology term IDs, so no uncontrolled-vocabulary normalization is needed
- The goal is classifying cells (extending probability scores to unseen cell types via the Cell Ontology graph) rather than normalizing label terminology
- The task requires distinguishing topologically identical unseen cell types (siblings with the same parents in the Cell Ontology graph), which OnClass cannot do

## Inputs

- Uncontrolled free-text cell type annotations from user datasets (e.g., Tabula Muris Senis labels)
- Cell Ontology files: cl.ontology.nlp.emb text embeddings of cell type definitions and the cell type network
- Small hand-mapped free-text-to-CL-term ground-truth label set for validation

## Outputs

- cell_type_nlp_mapping_table.csv of accepted free-text-to-Cell-Ontology-term pairs
- Per-label mapping accuracy scores against the hand-mapped ground-truth label set

## How to apply

Load the Cell Ontology NLP assets (cl.ontology.nlp.emb text embeddings of cell type definitions and the cell type network) together with a small hand-mapped free-text label set such as Tabula Muris Senis annotations. Embed cell types into the low-dimensional space with OnClass_train_obj.EmbedCellTypes(train_label), which invokes the char-level LSTM siamese network; enable this mode by setting nlp_mapping = True when reading training/test data. For each free-text annotation, compute similarity against all Cell Ontology term definitions, retrieve the nearest controlled-vocabulary term, then refine pairings with maximum-weight bipartite matching so each label is assigned its best overall Cell Ontology term. Validate the resulting mapping against the hand-mapped ground truth, targeting the paper-reported 93.7% char-level LSTM accuracy on synonyms, misspellings, and abbreviations, and persist accepted pairs to cell_type_nlp_mapping_table.csv.

## Related tools

- **OnClass** (Provides the char-level LSTM NLP model that embeds cell type descriptions and maps free-text annotations to Cell Ontology terms) — https://github.com/wangshenguiuc/OnClass
- **OnClass 1.2** (Pinned pip-installable version of the OnClass software used for the NLP terminology mapping) — https://github.com/wangshenguiuc/OnClass
- **tensorflow 2.0** (Backend for the char-level LSTM siamese network used to embed cell type text)
- **Python 3.6** (Runtime environment required to run OnClass NLP mapping)

## Examples

```
pip install OnClass==1.2 tensorflow==2.0; python -c "onclass_train_obj.EmbedCellTypes(train_label)"  # with nlp_mapping=True when reading data
```

## Evaluation signals

- Mapping accuracy on the hand-mapped ground-truth label set reaches the paper-reported 93.7% for synonyms, misspellings, and abbreviations
- Every free-text label in the input is paired with exactly one Cell Ontology term after maximum-weight bipartite matching (one-to-one assignment)
- Output file cell_type_nlp_mapping_table.csv is produced with accepted free-text-to-CL-term pairs
- Embedded cell types occupy the same low-dimensional space as the cl.ontology.nlp.emb definition embeddings, so nearest-neighbor retrieval returns CL terms
- nlp_mapping = True was set when reading data so the char-level LSTM (not default vocabulary handling) performs the mapping

## Limitations

- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states, so some free-text labels may have no correct CL target
- Planned improvements to consider the Cell Ontology graph structure (not only text data) in the NLP models remain future work
- Automated mapping quality depends on a small hand-mapped set of free-text annotations for benchmarking

## Evidence

- [methods] OnClass uses char-level LSTM NLP mapping mode: "Read training and test data. Set nlp_mapping = True to use the Char-level LSTM that maps uncontrolled vocabulary to controlled vocabulary."
- [methods] Embedding Cell ontology via the training object: "OnClass_train_obj.EmbedCellTypes(train_label)"
- [discussion] NLP model maps free text annotations to Cell Ontology terms: "Here we already offer an NLP model that is able to map free text annotations to Cell Ontology terms"
- [abstract] OnClass purpose as software for the controlled vocabulary: "Here, we present OnClass, an algorithm and accompanying software for automatically classifying cells into cell types that are part of the controlled vocabulary that forms the Cell Ontology."
