---
name: ontology-leaf-node-cell-type-mapping
description: Use when DCNet abundance predictions must be restricted to or interpreted
  for specific terminal cell types (e.g. FGC, somatic cells) in a Cell Ontology hierarchy,
  rather than the full 434-type landscape.
when_to_use_negative:
- When the goal is the full 434-cell-type landscape rather than specific Cell Ontology
  leaf nodes — DCNet already outputs all embedded cell types directly.
- When the cell type of interest is not among the 434 cell types embedded in the DCNet
  network (e.g. cell types whose marker genes were not detected and were discarded
  during filtering).
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: DCNet
  role: Deep learning model whose output neurons correspond to the 434 embedded cell
    types; leaf-node abundances are extracted from its predictions
  repo: https://github.com/xindd/DCNet
- name: mxnet
  role: Python framework in which the DCNet model is coded and run to produce cell-type
    abundances
- name: Python
  role: Language used to load index.pkl/rix.pkl, extract leaf-node abundances, and
    compute Pearson correlations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1080/2162402x.2022.2043662
    title: Deep learning using bulk RNA-seq data expands cell landscape identification
      in tumor microenvironment
  source_content_hash: sha256:48e714f738733e25274d1e8fe9569ec4c522f0500532c339b1e7c5d4ce2b6480
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/xindd/DCNet
  tools:
  - DCNet
  - Python
  - mxnet
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# ontology-leaf-node-cell-type-mapping

## Summary

Maps the 434 cell-type neurons embedded in DCNet's network structure to their leaf-node positions in the Cell Ontology hierarchy (e.g. fetal germ cells and somatic cells), so predicted abundances can be extracted and validated against ground-truth single-cell-derived abundances.

## When to use

Use when DCNet abundance predictions must be restricted to or interpreted for specific terminal cell types (e.g. FGC, somatic cells) in a Cell Ontology hierarchy, rather than the full 434-type landscape.

## When NOT to use

- When the goal is the full 434-cell-type landscape rather than specific Cell Ontology leaf nodes — DCNet already outputs all embedded cell types directly.
- When the cell type of interest is not among the 434 cell types embedded in the DCNet network (e.g. cell types whose marker genes were not detected and were discarded during filtering).

## Inputs

- DCNet index.pkl file mapping input and output neurons to gene symbols
- relation_matrix.pkl / rix.pkl zero-one gene-cell correspondence matrix
- DCNet predicted abundance matrix for the 434 cell types (rows = cell types, columns = samples)
- Cell Ontology hierarchy defining parent/leaf relationships among the 434 embedded cell types
- True cell-type abundances aggregated from single-cell data (e.g. GSE86146: 2167 individual PGCs and their gonadal niche cells)

## Outputs

- Leaf-node-restricted abundance table (e.g. FGC and somatic cell predictions per sample)
- Pearson correlation values with p-values against true abundances (expected: FGC ~0.87, somatic ~0.79, p < .01)
- Results table saved as gse86146_correlations.csv

## How to apply

DCNet's architecture encodes a zero-one gene-cell correspondence matrix covering 434 cell types, and the index.pkl file stores the set of genes mapped to each input and output neuron, so the first step is loading index.pkl and the relation matrix (rix.pkl/relation_matrix.pkl) to identify which output neurons correspond to which cell types. Using the Cell Ontology hierarchy, select the leaf-node cell types of interest — in the GSE86146 embryonic validation these were the fetal germ cell (FGC) leaf node and its somatic cell siblings among the gonadal niche cells. Extract the predicted abundances for exactly those leaf-node neurons from the DCNet output matrix (columns are samples, rows are cell types) rather than aggregating internal or parent-node values. Compute Pearson correlations of the extracted leaf-node predictions against true abundances derived from aggregated single-cell data; in the published GSE86146 run this yielded r = 0.87 for FGC and r = 0.79 for somatic cells, both with p < .01.

## Related tools

- **DCNet** (Deep learning model whose output neurons correspond to the 434 embedded cell types; leaf-node abundances are extracted from its predictions) — https://github.com/xindd/DCNet
- **mxnet** (Python framework in which the DCNet model is coded and run to produce cell-type abundances)
- **Python** (Language used to load index.pkl/rix.pkl, extract leaf-node abundances, and compute Pearson correlations)

## Examples

```
import pandas as pd; idx = pd.read_pickle('index.pkl'); rix = pd.read_pickle('relation_matrix.pkl')
```

## Evaluation signals

- Pearson correlation between predicted and true FGC abundance is approximately 0.87 (p < .01), matching Figure 3c
- Pearson correlation for somatic cells is approximately 0.79 (p < .01), matching Figure 3d
- Extracted abundances correspond only to leaf-node cell types of the Cell Ontology hierarchy, confirmed against index.pkl output-neuron mappings
- Results table is saved as gse86146_correlations.csv with per-cell-type correlation entries

## Limitations

- Only 434 of the original 467 CellMarker cell types are embedded; 33 cell types were discarded because none of their marker genes were detected, so unrepresented leaf nodes cannot be mapped or predicted
- index.pkl and relation_matrix.pkl document the neuron-to-gene and gene-to-cell mappings, but no explicit cell-type-to-ontology-URI lookup table is provided in the repository, so leaf-node identification relies on the Cell Ontology hierarchy and the CellMarker-derived cell labels
- No random seed or fine-tuning procedure for the recommended dropout-0.1 DCNet is specified, which can affect reproducibility of per-neuron abundance estimates

## Evidence

- [methods] leaf-node extraction: [evidence span withheld — anchor status: not_found]
- [results] correlation finding: [evidence span withheld — anchor status: not_found]
