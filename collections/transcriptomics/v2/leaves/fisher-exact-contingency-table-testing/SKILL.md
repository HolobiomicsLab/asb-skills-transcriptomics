---
name: fisher-exact-contingency-table-testing
description: Use when two WGCNA/PyWGCNA co-expression networks from different datasets
  must be compared at the module level — i.e., you need to identify which module pairs
  share significantly more genes than expected by chance.
when_to_use_negative:
- You only have one WGCNA network — module comparison requires two PyWGCNA objects
  from different datasets (or one PyWGCNA object plus an external gene/marker list,
  which is a separate workflow)
- You need module-level functional interpretation rather than cross-network conservation
  testing — use PyWGCNA's functional enrichment workflow (GO, KEGG, REACTOME) instead
- Your input is raw RNA-seq count data that has not yet been through PyWGCNA module
  construction — build the networks first (Quick Start workflow), then compare them
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: PyWGCNA
  role: Provides the module-comparison routine ('Compare two PyWGCNA objects', tutorials/Comparison_two_PyWGCNAs.ipynb)
    that builds module-pair contingency tables for Fisher exact testing; also hosts
    the 5xFAD_paper tutorial data
  repo: github.com/mortazavilab/PyWGCNA
- name: Python
  role: Runtime environment for PyWGCNA; the article requires Python 3.9 or greater,
    the README requires 3.10 or greater
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:4be469dba9f9cb8e9a4ede47373121076221726c5943981e47eef2faa85d8882
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyWGCNA
  - Python
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# fisher-exact-contingency-table-testing

## Summary

Applies Fisher's exact test to module-pair gene-membership contingency tables to detect significant overlap between co-expression networks built from different datasets (e.g., the 5xFAD coral module vs. the 3xTg-AD darkgray module), as implemented in PyWGCNA's module-comparison workflow.

## When to use

Use when two WGCNA/PyWGCNA co-expression networks from different datasets must be compared at the module level — i.e., you need to identify which module pairs share significantly more genes than expected by chance.

## When NOT to use

- You only have one WGCNA network — module comparison requires two PyWGCNA objects from different datasets (or one PyWGCNA object plus an external gene/marker list, which is a separate workflow)
- You need module-level functional interpretation rather than cross-network conservation testing — use PyWGCNA's functional enrichment workflow (GO, KEGG, REACTOME) instead
- Your input is raw RNA-seq count data that has not yet been through PyWGCNA module construction — build the networks first (Quick Start workflow), then compare them

## Inputs

- Two PyWGCNA objects (or their module-assignment tables) from distinct datasets, e.g., the 5xFAD brain bulk RNA-seq network and the 3xTg-AD brain bulk RNA-seq network
- Python >= 3.9 environment with PyWGCNA installed via 'pip install PyWGCNA'

## Outputs

- module_overlap.csv — complete pairwise table of Jaccard coefficients, overlap gene counts/proportions, and Fisher exact-test P-values for every module-pair
- List of significantly overlapping module pairs (conserved co-expression signatures), e.g., coral (5xFAD) ~ darkgray (3xTg-AD) and white ~ white

## How to apply

Load the two module-assignment tables (e.g., the 5xFAD network from task_001 and the 3xTg-AD network from task_003) into PyWGCNA objects (Python >= 3.9; the README states 3.10+, installed via 'pip install PyWGCNA'). Use PyWGCNA's module-comparison routine (per the 'Compare two PyWGCNA objects' tutorial, Comparison_two_PyWGCNAs.ipynb) to build the gene-membership contingency table for every module-pair between the networks. From each 2x2 table compute the Jaccard similarity coefficient and the overlap gene counts/proportions, then apply Fisher's exact test to obtain an overlap P-value for each pair. Flag pairs with significant Fisher P-values as conserved modules — in the PyWGCNA paper's MODEL-AD analysis this recovered the 5xFAD coral ~ 3xTg-AD darkgray overlap and white ~ white. Write the complete pairwise table (Jaccard coefficients, overlap counts/proportions, Fisher P-values) to module_overlap.csv.

## Related tools

- **PyWGCNA** (Provides the module-comparison routine ('Compare two PyWGCNA objects', tutorials/Comparison_two_PyWGCNAs.ipynb) that builds module-pair contingency tables for Fisher exact testing; also hosts the 5xFAD_paper tutorial data) — github.com/mortazavilab/PyWGCNA
- **Python** (Runtime environment for PyWGCNA; the article requires Python 3.9 or greater, the README requires 3.10 or greater)

## Examples

```
pip install PyWGCNA
```

## Evaluation signals

- The 5xFAD coral ~ 3xTg-AD darkgray pair and the white ~ white pair appear among the significantly overlapping module pairs, matching the paper's reported conserved modules
- module_overlap.csv contains a row for every 5xFAD module x 3xTg-AD module combination, with Jaccard coefficients bounded in [0, 1] and finite Fisher P-values
- Significant pairs are consistent with the finding that 'modules with significant overlap across the datasets' indicate shared co-expression signatures conserved across the two familial AD mouse models
- Overlap gene counts/proportions in each 2x2 contingency table sum to the total number of genes shared across the two networks' membership tables

## Limitations

- PyWGCNA module comparison assesses conservation/overlap of gene-membership; it does not by itself establish functional equivalence — pair it with GO/KEGG/REACTOME enrichment for interpretation
- Fisher exact test significance on small modules or on network pairs with few shared genes can be unstable; check overlap counts/proportions alongside P-values
- Environment versions differ between sources: the article requires Python 3.9+, while the README states Python 3.10+ is required — use the stricter version to avoid installation failures

## Evidence

- [abstract] We compare the resulting modules to each other to find shared co-expression signatures in the form of modules with significant overlap across the datasets.: "We compare the resulting modules to each other to find shared co-expression signatures in the form of modules with significant overlap across the datasets."
- [intro] Users can also compare WGCNA networks from different datasets, or to external gene lists, to assess the conservation or functional enrichment of each module.: "Users can also compare WGCNA networks from different datasets, or to external gene lists, to assess the conservation or functional enrichment of each module."
- [readme] Compare two PyWGCNA objects](tutorials/Comparison_two_PyWGCNAs.ipynb): How to compare two PyWGCNA objects: [evidence span withheld — anchor status: external_doc]
- [readme] To install PyWGCNA, Python version 3.10 or greater is required.: [evidence span withheld — anchor status: external_doc]
