---
name: jaccard-similarity-module-overlap-quantification
description: Use when two PyWGCNA objects (e.g., the 5xFAD and 3xTg-AD networks) have
  been built with module assignments and you need to find which module pairs significantly
  overlap, indicating conserved co-expression signatures across datasets.
when_to_use_negative:
- Only one network exists — comparing a PyWGCNA object to an external gene marker
  list uses the Comparison_PyWGCNA_geneMarker workflow instead
- You need to compare more than two PyWGCNA objects — the article provides a separate
  multi-comparison tutorial (Comparison_multi_PyWGCNAs.ipynb)
- Your goal is functional enrichment (GO, KEGG, REACTOME) or PPI (STRING) analysis
  of a single module rather than cross-dataset module overlap
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: PyWGCNA
  role: Provides the compareModules routine that builds the gene-membership contingency
    table for every module pair across two PyWGCNA objects
  repo: github.com/mortazavilab/PyWGCNA
- name: Python 3
  role: Runtime environment required to install and run PyWGCNA (version 3.9+ per
    the article, 3.10+ per the README)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:4345ec8530cb88bf09cc5ca059fafc6687cbd30029d1139948efa0151105f647
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyWGCNA
  - Python
  - Python 3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# jaccard-similarity-module-overlap-quantification

## Summary

Quantifies co-expression module overlap between two WGCNA networks (here the 5xFAD and 3xTg-AD MODEL-AD brain bulk RNA-seq networks) using Jaccard similarity, gene-membership contingency tables, and Fisher's exact test P-values to identify modules with significant overlap conserved across datasets.

## When to use

Use when two PyWGCNA objects (e.g., the 5xFAD and 3xTg-AD networks) have been built with module assignments and you need to find which module pairs significantly overlap, indicating conserved co-expression signatures across datasets.

## When NOT to use

- Only one network exists — comparing a PyWGCNA object to an external gene marker list uses the Comparison_PyWGCNA_geneMarker workflow instead
- You need to compare more than two PyWGCNA objects — the article provides a separate multi-comparison tutorial (Comparison_multi_PyWGCNAs.ipynb)
- Your goal is functional enrichment (GO, KEGG, REACTOME) or PPI (STRING) analysis of a single module rather than cross-dataset module overlap

## Inputs

- Two PyWGCNA objects with completed module assignments (here: 5xFAD network from task_001 and 3xTg-AD network from task_003)
- Module-assignment tables mapping each gene to its color-labeled module in each network

## Outputs

- module_overlap.csv containing the full pairwise table of Jaccard coefficients, overlap gene counts/proportions, and Fisher's exact test P-values for every module pair
- List of module pairs with significant Fisher overlap P-values (expected to include coral (5xFAD)~darkgray (3xTg-AD) and white~white)

## How to apply

Load the 5xFAD (task_001) and 3xTg-AD (task_003) module-assignment tables into PyWGCNA objects (Python >= 3.9 per the article; >= 3.10 per the README, installed via 'pip install PyWGCNA'). Run PyWGCNA's compareModules routine to build a gene-membership contingency table for every 5xFAD-module x 3xTg-AD-module pair. From each pair's 2x2 contingency table, compute the Jaccard similarity coefficient and the overlap gene counts/proportions, and apply Fisher's exact test to obtain an overlap P-value. Flag pairs with significant Fisher P-values and confirm that coral (5xFAD)~darkgray (3xTg-AD) and white~white appear among the significant overlaps reported in the paper. Write the complete pairwise table to module_overlap.csv.

## Related tools

- **PyWGCNA** (Provides the compareModules routine that builds the gene-membership contingency table for every module pair across two PyWGCNA objects) — github.com/mortazavilab/PyWGCNA
- **Python 3** (Runtime environment required to install and run PyWGCNA (version 3.9+ per the article, 3.10+ per the README))

## Examples

```
PyWGCNA.compareModules(5xFAD_wgcna, tg3xTgAD_wgcna, outputfile="module_overlap.csv")
```

## Evaluation signals

- module_overlap.csv contains a row for every 5xFAD-module x 3xTg-AD-module combination with a Jaccard coefficient, overlap counts/proportions, and a Fisher P-value
- All Jaccard coefficients fall in the [0, 1] range and overlap counts are consistent with the gene-membership contingency tables
- Coral (5xFAD)~darkgray (3xTg-AD) appears among the significant Fisher P-value overlaps, matching the paper's finding
- White~white appears among the significant overlaps reported in the paper
- Jaccard coefficients computed from the contingency tables match the values reported in module_overlap.csv

## Limitations

- Python version requirement is inconsistent between sources: the article methods state Python 3.9 or greater is required, while the README states 3.10 or greater
- The comparison tutorial pair covers two PyWGCNA objects; comparing three or more requires the separate multi-comparison workflow
- Overlap significance depends on Fisher's exact test P-values from 2x2 contingency tables, so very small modules may yield underpowered tests

## Evidence

- [readme] Users can also compare WGCNA networks from different datasets, or to external gene lists, to assess the conservation or functional enrichment of each module.: "Users can also compare WGCNA networks from different datasets, or to external gene lists, to assess the conservation or functional enrichment of each module."
- [abstract] We compare the resulting modules to each other to find shared co-expression signatures in the form of modules with significant overlap across the datasets.: "We compare the resulting modules to each other to find shared co-expression signatures in the form of modules with significant overlap across the datasets."
- [methods] To install PyWGCNA, python version 3.9 or greater is required.: "To install PyWGCNA, python version 3.9 or greater is required."
- [readme] To install PyWGCNA, Python version 3.10 or greater is required.: [evidence span withheld — anchor status: external_doc]
