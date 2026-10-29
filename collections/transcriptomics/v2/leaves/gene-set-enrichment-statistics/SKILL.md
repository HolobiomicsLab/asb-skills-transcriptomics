---
name: gene-set-enrichment-statistics
description: Use when a gene list derived from a co-expression module (e.g., a PyWGCNA
  module correlated with an external trait at P < 0.05) needs GO/KEGG/REACTOME functional
  enrichment to identify the biological functions the module represents.
when_to_use_negative:
- Input is raw RNA-seq count data with no co-expression modules identified yet — module
  discovery (PyWGCNA Quick Start) must precede enrichment
- The analysis goal is module-trait correlation itself rather than functional interpretation
  of module membership
- The gene list lacks identifiers compatible with the GO/KEGG/REACTOME background,
  e.g., when comparing networks to external marker lists without ID mapping
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: PyWGCNA
  role: Python WGCNA package whose built-in functional enrichment analysis runs GO/KEGG/REACTOME
    tests on module gene lists and relates modules to external sample traits via eigengene
    correlations
  repo: https://github.com/mortazavilab/PyWGCNA
- name: GSEApy
  role: Used in the task workflow to run GO functional enrichment on the coral and
    white module gene lists outside the PyWGCNA object
- name: Python
  role: Runtime environment for PyWGCNA (Python 3.10 or greater required per README;
    article states 3.9+)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:f4d7f0f486129c98773beb12bcf3bb8604e365c2055a9e5ee25d74f4add2bf31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - GSEApy
  - PyWGCNA
  - Python
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# gene-set-enrichment-statistics

## Summary

Functional enrichment testing of co-expression module gene lists against GO, KEGG, and REACTOME databases, as implemented in PyWGCNA and demonstrated via GSEApy on the 5xFAD MODEL-AD bulk RNA-seq modules (coral: 1335 genes enriched for immune response/neutrophil activation; white: cilium organization terms).

## When to use

Use when a gene list derived from a co-expression module (e.g., a PyWGCNA module correlated with an external trait at P < 0.05) needs GO/KEGG/REACTOME functional enrichment to identify the biological functions the module represents.

## When NOT to use

- Input is raw RNA-seq count data with no co-expression modules identified yet — module discovery (PyWGCNA Quick Start) must precede enrichment
- The analysis goal is module-trait correlation itself rather than functional interpretation of module membership
- The gene list lacks identifiers compatible with the GO/KEGG/REACTOME background, e.g., when comparing networks to external marker lists without ID mapping

## Inputs

- gene-to-module assignment table from a PyWGCNA run (task_001 output)
- sample trait metadata for the 5xFAD MODEL-AD bulk RNA-seq dataset (age progression, brain region)
- significant module-trait correlation table (module_trait_correlations.csv), filtered at P < 0.05
- gene list for each significant module (e.g., coral module, 1335 genes)
- GO Biological Process gene sets (via GSEApy or PyWGCNA's GO/KEGG/REACTOME enrichment)

## Outputs

- module_go_enrichment.csv containing GO terms, statistics, and significance for each enriched module
- Biological interpretation per module: coral module -> immune response / neutrophil activation terms; white module -> cilium movement / organization / assembly terms

## How to apply

First establish which modules are biologically relevant: compute correlations between module eigengenes and external sample traits (e.g., age progression, hippocampal brain region) in PyWGCNA, retaining only significant associations at P < 0.05 — in the 5xFAD example, coral against age progression and white against hippocampal region. Extract the gene list for each significant module from the gene-to-module assignment table (e.g., the coral module contains 1335 genes). Run functional enrichment on that gene list using the GO Biological Process ontology (PyWGCNA's built-in functional enrichment analysis supports GO, KEGG, and REACTOME; the task workflow also uses GSEApy). Persist significant terms to a results table (module_go_enrichment.csv) alongside the module-trait correlation table (module_trait_correlations.csv). Validate by checking that the expected term categories appear: immune response/neutrophil activation for the coral module and cilium movement/organization/assembly for the white module.

## Related tools

- **PyWGCNA** (Python WGCNA package whose built-in functional enrichment analysis runs GO/KEGG/REACTOME tests on module gene lists and relates modules to external sample traits via eigengene correlations) — https://github.com/mortazavilab/PyWGCNA
- **GSEApy** (Used in the task workflow to run GO functional enrichment on the coral and white module gene lists outside the PyWGCNA object)
- **Python** (Runtime environment for PyWGCNA (Python 3.10 or greater required per README; article states 3.9+))

## Examples

```
import gseapy as gp; res = gp.enrichr(gene_list=coral_module_genes, gene_sets='GO_Biological_Process', organism='Mouse')
```

## Evaluation signals

- Coral-age and white-hippocampus module-trait correlations each meet the P < 0.05 criterion before enrichment is attempted
- The coral module gene list contains 1335 genes before enrichment is run
- module_go_enrichment.csv contains immune response / neutrophil activation GO terms for the coral module
- module_go_enrichment.csv contains cilium movement / organization / assembly terms for the white module
- Both output CSVs (module_trait_correlations.csv and module_go_enrichment.csv) are written and parseable

## Limitations

- Enrichment results depend on the chosen database — PyWGCNA supports GO, KEGG, and REACTOME, so term categories differ across choices
- Significant module selection is threshold-dependent (P < 0.05); modules failing this cutoff are excluded from enrichment
- The provided article text has no discussion section content ('_No changelog found._'), so limitations beyond the abstract/methods are not documented

## Evidence

- [abstract] We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets.: "We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets."
- [intro] Users can also compare WGCNA networks from different datasets, or to external gene lists, to assess the conservation or functional enrichment of each module.: "Users can also compare WGCNA networks from different datasets, or to external gene lists, to assess the conservation or functional enrichment of each module."
- [methods] How to perform functional enrichment analysis using databases such as GO, KEGG, and REACTOME: "How to perform functional enrichment analysis using databases such as GO, KEGG, and REACTOME"
- [readme] How to perform functional enrichment analysis using databases such as GO, KEGG, and REACTOME in PyWGCNA object: "How to perform functional enrichment analysis using databases such as GO, KEGG, and REACTOME in PyWGCNA object"
