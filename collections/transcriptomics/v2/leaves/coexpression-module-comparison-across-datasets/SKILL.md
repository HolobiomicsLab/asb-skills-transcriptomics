---
name: coexpression-module-comparison-across-datasets
description: Use when two independently constructed WGCNA networks (or a network and
  an external gene list) must be compared module-to-module to find shared co-expression
  signatures, as done for 5xFAD vs 3xTg-AD PyWGCNA objects.
when_to_use_negative:
- You have only one WGCNA network and need within-dataset analyses (module detection,
  eigengenes, module-trait correlation) rather than cross-dataset module overlap
- You need functional enrichment of a module (GO/KEGG/REACTOME) rather than conservation
  between datasets — PyWGCNA's enrichment workflow is the appropriate skill
- Input is a network and a single external marker-gene list when you want gene-list
  comparison only; use the 'Compare PyWGCNA objects to gene marker list' workflow
  instead of full two-network comparison
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PyWGCNA
  role: Provides compareModules and comparison workflows (compare two or more PyWGCNA
    objects, compare to gene marker lists) that build contingency tables for module
    pairs; used to detect significant cross-dataset module overlap
  repo: github.com/mortazavilab/PyWGCNA
- name: WGCNA
  role: Upstream R reference method whose module concept PyWGCNA reproduces; familiarity
    recommended before comparing networks
  repo: https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-9-559
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:c6a9bd81b2d6a2ce9228640b33a3c8c5d0579e604e18634008b64d1edcd262c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyWGCNA
  - Python
  - WGCNA
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# coexpression-module-comparison-across-datasets

## Summary

Quantify conservation of WGCNA co-expression modules between two datasets (e.g., the 5xFAD and 3xTg-AD MODEL-AD brain bulk RNA-seq networks) using PyWGCNA's module-comparison routine, Fisher's exact test, and Jaccard similarity to identify modules with significant overlap.

## When to use

Use when two independently constructed WGCNA networks (or a network and an external gene list) must be compared module-to-module to find shared co-expression signatures, as done for 5xFAD vs 3xTg-AD PyWGCNA objects.

## When NOT to use

- You have only one WGCNA network and need within-dataset analyses (module detection, eigengenes, module-trait correlation) rather than cross-dataset module overlap
- You need functional enrichment of a module (GO/KEGG/REACTOME) rather than conservation between datasets — PyWGCNA's enrichment workflow is the appropriate skill
- Input is a network and a single external marker-gene list when you want gene-list comparison only; use the 'Compare PyWGCNA objects to gene marker list' workflow instead of full two-network comparison

## Inputs

- PyWGCNA objects (or module-assignment tables) for each dataset to compare, e.g. the 5xFAD and 3xTg-AD MODEL-AD brain bulk RNA-seq networks
- Gene-membership module assignments per dataset (gene ID -> color/module label)

## Outputs

- Pairwise module overlap table (module_overlap.csv) containing Jaccard similarity coefficients, overlap gene counts and proportions, and Fisher's exact test P-values for every module pair
- List of significantly overlapping module pairs (e.g., 5xFAD coral ~ 3xTg-AD darkgray) indicating conserved co-expression signatures

## How to apply

Load each dataset's module-assignment table into a PyWGCNA object (the article's 5xFAD tutorial objects from task_001 and task_003 serve as the exemplars; Python >= 3.9 required, install via 'pip install PyWGCNA'). Run PyWGCNA's compareModules routine to build the gene-membership contingency table for every module pair (e.g., every 5xFAD module x 3xTg-AD module). From each pair's 2x2 contingency table, compute Jaccard similarity coefficients, overlap gene counts/proportions, and a Fisher's exact test P-value; flag pairs with significant Fisher P-values. Confirm expected pairs appear among significant overlaps — in the source analysis, 5xFAD coral ~ 3xTg-AD darkgray was the conserved cross-model module. Write the full pairwise table (Jaccard, counts/proportions, P-values) to module_overlap.csv. The rationale is that significant cross-dataset overlap indicates a co-expression network conserved across the compared conditions or models, which PyWGCNA exposes as a first-class downstream analysis on top of its faster-than-R WGCNA implementation.

## Related tools

- **PyWGCNA** (Provides compareModules and comparison workflows (compare two or more PyWGCNA objects, compare to gene marker lists) that build contingency tables for module pairs; used to detect significant cross-dataset module overlap) — github.com/mortazavilab/PyWGCNA
- **WGCNA** (Upstream R reference method whose module concept PyWGCNA reproduces; familiarity recommended before comparing networks) — https://bmcbioinformatics.biomedcentral.com/articles/10.1186/1471-2105-9-559

## Examples

```
import PyWGCNA; wgcna5xfad = PyWGCNA.readWGCNA('5xFAD.p'); wgcna3xTg = PyWGCNA.readWGCNA('3xTg-AD.p'); cmp = wgcna5xfad.compareModules(wgcna3xTg)
```

## Evaluation signals

- The output contingency table covers all pairs (every 5xFAD module x every 3xTg-AD module) with non-negative integer overlap counts that are consistent with module gene counts
- Each pair's Fisher's exact test P-value is reproducible from the reported 2x2 overlap counts (counts in overlap, in module1 only, in module2 only, in neither)
- Jaccard coefficients equal overlap_count / (union count) for each pair and fall in [0, 1]
- The known significant pair — 5xFAD coral ~ 3xTg-AD darkgray — appears among the significant Fisher P-value overlaps, matching the published finding
- module_overlap.csv contains all three metrics (Jaccard, counts/proportions, Fisher P-values) for the complete pairwise comparison

## Limitations

- Requires Python 3.9 or greater (article methods) / 3.10 or greater (README installation section) — check the installed environment
- Interpretation depends on Fisher significance calls on 2x2 tables; large module-count imbalances or many simultaneous pairwise tests can affect which overlaps are flagged
- The workflow was demonstrated on two MODEL-AD brain bulk RNA-seq datasets; module labels (e.g., coral, darkgray, white) are dataset-specific and must not be assumed to transfer across runs

## Evidence

- [abstract] We compare the resulting modules to each other to find shared co-expression signatures in the form of modules with significant overlap across the datasets.: "We compare the resulting modules to each other to find shared co-expression signatures in the form of modules with significant overlap across the datasets."
- [intro] Users can also compare WGCNA networks from different datasets, or to external gene lists, to assess the conservation or functional enrichment of each module.: "Users can also compare WGCNA networks from different datasets, or to external gene lists, to assess the conservation or functional enrichment of each module."
- [methods] To install PyWGCNA, python version 3.9 or greater is required.: "To install PyWGCNA, python version 3.9 or greater is required."
- [readme] Compare two PyWGCNA objects ... How to compare two PyWGCNA objects: [evidence span withheld — anchor status: external_doc]
- [abstract] PyWGCNA has a faster implementation than the R version of WGCNA and several additional downstream analysis modules for functional enrichment analysis using GO, KEGG, and REACTOME: "PyWGCNA has a faster implementation than the R version of WGCNA and several additional downstream analysis modules for functional enrichment analysis using GO, KEGG, and REACTOME"
