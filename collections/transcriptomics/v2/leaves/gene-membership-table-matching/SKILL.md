---
name: gene-membership-table-matching
description: Use when two WGCNA/PyWGCNA networks built from different datasets (e.g.,
  5xFAD and 3xTg-AD MODEL-AD mouse RNA-seq) must be compared module-by-module to find
  conserved co-expression signatures via significant gene-set overlap.
when_to_use_negative:
- Goal is comparing a PyWGCNA object to an external gene/marker list rather than another
  co-expression network — PyWGCNA provides a separate 'Compare PyWGCNA objects to
  gene marker list' workflow
- Only one dataset's modules exist (single-network module detection or trait correlation),
  so there is no second network to compare against
- The two networks were built from non-overlapping gene sets, making membership contingency
  tables and Jaccard coefficients uninterpretable
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: PyWGCNA
  role: Provides the compareModules routine (and the 'Compare two PyWGCNA objects'
    tutorial) to build gene-membership contingency tables and statistically test module
    overlap between two networks
  repo: https://github.com/mortazavilab/PyWGCNA
- name: Python
  role: Runtime environment for PyWGCNA; the paper requires Python 3.9 or greater
    (README states 3.10 or greater), installed via 'pip install PyWGCNA'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:0f1258c1a4897694094e0d62358affa6746d3761eb16591c01a34a5476ddc72d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - PyWGCNA
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-membership-table-matching

## Summary

Cross-compare co-expression modules from two PyWGCNA networks (here, 5xFAD and 3xTg-AD brain bulk RNA-seq) by building gene-membership contingency tables for every module pair, quantifying Jaccard similarity and overlap counts/proportions, and testing each pair with Fisher's exact test to flag significantly overlapping (conserved) modules.

## When to use

Use when two WGCNA/PyWGCNA networks built from different datasets (e.g., 5xFAD and 3xTg-AD MODEL-AD mouse RNA-seq) must be compared module-by-module to find conserved co-expression signatures via significant gene-set overlap.

## When NOT to use

- Goal is comparing a PyWGCNA object to an external gene/marker list rather than another co-expression network — PyWGCNA provides a separate 'Compare PyWGCNA objects to gene marker list' workflow
- Only one dataset's modules exist (single-network module detection or trait correlation), so there is no second network to compare against
- The two networks were built from non-overlapping gene sets, making membership contingency tables and Jaccard coefficients uninterpretable

## Inputs

- 5xFAD PyWGCNA object / module-assignment table (from task_001)
- 3xTg-AD PyWGCNA object / module-assignment table (from task_003)
- Gene-membership mapping shared across both networks

## Outputs

- Gene-membership contingency table for every 5xFAD module x 3xTg-AD module pair
- Pairwise table with Jaccard similarity coefficients, overlap gene counts/proportions, and Fisher's exact test P-values
- module_overlap.csv containing the complete pairwise comparison
- List of significantly overlapping module pairs (e.g., coral~darkgray, white~white)

## How to apply

Load the module-assignment tables from each dataset (5xFAD from task_001, 3xTg-AD from task_003) into PyWGCNA objects (Python >= 3.9 per the paper; the README states 3.10 or greater, so install with 'pip install PyWGCNA' on a modern interpreter). Run PyWGCNA's compareModules routine, following the 'Compare two PyWGCNA objects' tutorial, to build the gene-membership contingency table for every 5xFAD module x 3xTg-AD module pair. From each 2x2 contingency table (module-pair overlap vs. genes in only one module), compute the Jaccard similarity coefficient and the overlap gene counts/proportions. Apply Fisher's exact test to each pair's 2x2 table to obtain an overlap P-value, and flag pairs with significant P-values. Correct application is confirmed when coral (5xFAD) ~ dark gray (3xTg-AD) and white ~ white appear among the significant overlaps, matching the reported finding that the coral and dark gray modules are conserved across the two familial AD mouse models. Write the full pairwise table (Jaccard coefficients, overlap counts/proportions, Fisher P-values) to module_overlap.csv.

## Related tools

- **PyWGCNA** (Provides the compareModules routine (and the 'Compare two PyWGCNA objects' tutorial) to build gene-membership contingency tables and statistically test module overlap between two networks) — https://github.com/mortazavilab/PyWGCNA
- **Python** (Runtime environment for PyWGCNA; the paper requires Python 3.9 or greater (README states 3.10 or greater), installed via 'pip install PyWGCNA')

## Examples

```
from PyWGCNA import PyWGCNA; pywgcna_5xFAD = PyWGCNA.readWGCNA('5xFAD.p'); pywgcna_3xTgAD = PyWGCNA.readWGCNA('3xTgAD.p'); pywgcna_5xFAD.compareModules(pywgcna_3xTgAD)
```

## Evaluation signals

- The coral (5xFAD) ~ dark gray (3xTg-AD) and white ~ white module pairs appear among the significant Fisher P-value overlaps, reproducing the paper's reported conserved modules
- module_overlap.csv contains a Jaccard coefficient, overlap count/proportion, and Fisher P-value for every 5xFAD module x 3xTg-AD module pair (no missing cells)
- Each module pair's 2x2 contingency table sums to the total number of genes shared by the two networks (overlapped + unique genes)
- Jaccard coefficients fall in [0, 1] and Fisher P-values fall in [0, 1]; overlap proportions match the contingency-table counts
- The significant-overlap flags match the paper's conclusion that modules with significant overlap indicate shared co-expression signatures across the MODEL-AD datasets

## Limitations

- Interpreter version conflict: the article's methods state 'python version 3.9 or greater is required' while the README states 'Python version 3.10 or greater is required' — verify against the installed PyWGCNA release
- Overlap significance via Fisher's exact test depends on module sizes and the shared gene background; small modules or large module counts inflate the multiple-testing burden across all pairwise comparisons
- Comparison assumes both networks were built with the PyWGCNA pipeline on comparable gene sets; no guidance in the provided context for networks built by other tools

## Evidence

- [intro] Users can also compare WGCNA networks from different datasets, or to external gene lists, to assess the conservation or functional enrichment of each module.: "Users can also compare WGCNA networks from different datasets, or to external gene lists, to assess the conservation or functional enrichment of each module."
- [abstract] We compare the resulting modules to each other to find shared co-expression signatures in the form of modules with significant overlap across the datasets.: "We compare the resulting modules to each other to find shared co-expression signatures in the form of modules with significant overlap across the datasets."
- [methods] Compare two PyWGCNA objects ... How to compare two PyWGCNA objects: "`Compare two PyWGCNA objects <https://github.com/mortazavilab/PyWGCNA/tree/main/tutorials/Comparison_two_PyWGCNAs.ipynb>`_"
- [readme] To install PyWGCNA, Python version 3.10 or greater is required.: [evidence span withheld — anchor status: external_doc]
