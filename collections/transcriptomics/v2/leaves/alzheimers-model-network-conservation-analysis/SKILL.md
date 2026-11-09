---
name: alzheimers-model-network-conservation-analysis
description: Use when two (or more) independently constructed co-expression networks
  exist — e.g. 5xFAD vs 3xTg-AD module-assignment tables — and the goal is to flag
  module pairs with significant gene overlap indicating conserved co-expression.
when_to_use_negative:
- Only one WGCNA network is available and there is no second dataset or external gene
  list to compare against — module comparison requires two or more objects.
- You are still constructing the networks themselves (module detection) rather than
  comparing already-built module assignments.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PyWGCNA
  role: Python library whose two-object comparison routine builds module-pair contingency
    tables and computes overlap significance between the 5xFAD and 3xTg-AD networks
  repo: https://github.com/mortazavilab/PyWGCNA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:36b1086edb4b2f46832db68b2b4267f7b081c7f7fc5051f7779020ad02f4adeb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - PyWGCNA
  - Python
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# alzheimers-model-network-conservation-analysis

## Summary

Compare co-expression modules built from two independent WGCNA networks — here the 5xFAD and 3xTg-AD MODEL-AD brain bulk RNA-seq datasets — using PyWGCNA's module-comparison routine to detect shared co-expression signatures via Jaccard similarity and Fisher's exact test overlap significance.

## When to use

Use when two (or more) independently constructed co-expression networks exist — e.g. 5xFAD vs 3xTg-AD module-assignment tables — and the goal is to flag module pairs with significant gene overlap indicating conserved co-expression.

## When NOT to use

- Only one WGCNA network is available and there is no second dataset or external gene list to compare against — module comparison requires two or more objects.
- You are still constructing the networks themselves (module detection) rather than comparing already-built module assignments.

## Inputs

- 5xFAD PyWGCNA object / module-assignment table (from task_001)
- 3xTg-AD PyWGCNA object / module-assignment table (from task_003)

## Outputs

- module_overlap.csv with the complete pairwise table: Jaccard coefficients, overlap counts/proportions, and Fisher exact test P-values for every 5xFAD-module x 3xTg-AD-module pair
- List of significant module-pair overlaps indicating conserved co-expression signatures

## How to apply

Load the pre-built 5xFAD and 3xTg-AD PyWGCNA objects (Python >= 3.9 installed via 'pip install PyWGCNA'), then run the PyWGCNA comparison of two PyWGCNA objects, which builds the gene-membership contingency table for every 5xFAD-module x 3xTg-AD-module pair. From each pair's 2x2 contingency table compute the Jaccard similarity coefficient, the overlap gene counts/proportions, and a Fisher's exact test P-value to score the overlap. Flag module pairs with significant Fisher P-values — the expected conserved pair is 5xFAD coral with 3xTg-AD dark gray (with white~white also appearing). The rationale is that significant module overlap across datasets, as demonstrated in the PyWGCNA MODEL-AD application, indicates shared co-expression signatures conserved between the familial AD mouse models.

## Related tools

- **PyWGCNA** (Python library whose two-object comparison routine builds module-pair contingency tables and computes overlap significance between the 5xFAD and 3xTg-AD networks) — https://github.com/mortazavilab/PyWGCNA

## Examples

```
import PyWGCNA; p5xfad = PyWGCNA.readWGCNA('5xFAD.p'); p3xtg = PyWGCNA.readWGCNA('3xTgAD.p'); comparison = PyWGCNA.compareNetworks(p5xfad, p3xtg)
```

## Evaluation signals

- The coral (5xFAD) ~ darkgray (3xTg-AD) pair appears among the significant Fisher P-value overlaps, matching the finding that this pair 'significantly overlap one another'.
- The white~white pair also appears among the significant overlaps reported in the paper.
- module_overlap.csv contains a full pairwise grid: every 5xFAD module x every 3xTg-AD module with Jaccard coefficient, overlap count/proportion, and Fisher P-value columns.
- Jaccard and Fisher P-values are internally consistent with the overlap counts in each 2x2 contingency table (recomputable from module membership assignments).
- Tutorial 'Comparison_two_PyWGCNAs.ipynb' from the PyWGCNA repo reproduces the same workflow on the 5xFAD_paper data.

## Limitations

- Overlap significance depends on module sizes: very small modules can yield unstable Fisher's exact test P-values and low Jaccard coefficients even when biologically related.
- Conservation is measured only by gene-membership overlap; the skill does not directly assess conservation of module eigengene-trait associations across the two datasets.
- Python version requirement differs by source (article methods: >= 3.9; current README: >= 3.10) — check the installed environment against the version actually used.

## Evidence

- [intro] conservation via cross-dataset module comparison: "Users can also compare WGCNA networks from different datasets, or to external gene lists, to assess the conservation or functional enrichment of each module."
- [abstract] MODEL-AD shared signatures: "We compare the resulting modules to each other to find shared co-expression signatures in the form of modules with significant overlap across the datasets."
- [readme] tutorial for two-object comparison: [evidence span withheld — anchor status: external_doc]
- [methods] installation requirement: "To install PyWGCNA, python version 3.9 or greater is required."
