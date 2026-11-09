---
name: brain-rnaseq-module-functional-annotation
description: Use when a brain RNA-seq WGCNA analysis has produced gene-to-module assignments
  and you need the biological functions enriched per module plus significant module–trait
  correlations (e.g., coral module vs. age progression at P < 0.05).
when_to_use_negative:
- Input is raw counts without module assignments — run WGCNA module detection (Quick
  Start) first before annotating modules.
- Goal is single-gene differential expression rather than module-level function or
  eigengene–trait association.
- Module comparison against an external gene list where network conservation, not
  functional enrichment, is the question (use the Compare PyWGCNA objects workflow
  instead).
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0610
tools:
- name: PyWGCNA
  role: Computes module eigengene–trait correlations and runs GO/KEGG/REACTOME functional
    enrichment on module gene lists within the PyWGCNA object
  repo: https://github.com/mortazavilab/PyWGCNA
- name: GSEApy
  role: Performs GO functional enrichment on the coral and white module gene lists
- name: Python
  role: Runtime for PyWGCNA (Python 3.9+ per the article; README says 3.10+) and for
    orchestrating the workflow steps
  repo: https://github.com/mortazavilab/PyWGCNA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:59d6a23f3e970a9028eb19018a03bec5740b12a5dd6f2d5b5f6627eb0f0ff141
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - GSEApy
  - PyWGCNA
  - Python
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# brain-rnaseq-module-functional-annotation

## Summary

Annotates co-expression modules discovered in brain bulk RNA-seq (e.g., MODEL-AD 5xFAD) with functional meaning by running GO/KEGG/REACTOME enrichment on module gene lists in PyWGCNA and relating module eigengenes to external sample traits such as age progression and brain region.

## When to use

Use when a brain RNA-seq WGCNA analysis has produced gene-to-module assignments and you need the biological functions enriched per module plus significant module–trait correlations (e.g., coral module vs. age progression at P < 0.05).

## When NOT to use

- Input is raw counts without module assignments — run WGCNA module detection (Quick Start) first before annotating modules.
- Goal is single-gene differential expression rather than module-level function or eigengene–trait association.
- Module comparison against an external gene list where network conservation, not functional enrichment, is the question (use the Compare PyWGCNA objects workflow instead).

## Inputs

- gene-to-module assignment table from a PyWGCNA run (e.g., coral = 1335 genes, white module)
- sample trait metadata from the 5xFAD MODEL-AD bulk RNA-seq dataset (age progression, brain region)
- PyWGCNA object holding the network and module assignments

## Outputs

- module_trait_correlations.csv (significant module–trait correlations, P < 0.05)
- module_go_enrichment.csv (GO/KEGG/REACTOME enrichment results per module)
- module eigengene–trait correlation heatmap

## How to apply

Load the gene-to-module table (e.g., from task_001) and the sample trait metadata (age progression, brain region) from the 5xFAD MODEL-AD bulk RNA-seq dataset in the PyWGCNA tutorials. Compute module eigengene vs. trait correlations in PyWGCNA, retaining only significant associations (P < 0.05) — specifically coral module vs. age progression and white module vs. hippocampal region — and export them to module_trait_correlations.csv. Then run functional enrichment analysis with the module gene lists using databases such as GO, KEGG, and REACTOME in the PyWGCNA object, recovering immune response / neutrophil activation terms for the coral module (1335 genes) and cilium movement / organization / assembly terms for the white module; write results to module_go_enrichment.csv. The rationale is that eigengene network methodology relates whole modules to external sample traits, and module-level enrichment (rather than single-gene analysis) reveals the shared biological function of hundreds of co-expressed genes.

## Related tools

- **PyWGCNA** (Computes module eigengene–trait correlations and runs GO/KEGG/REACTOME functional enrichment on module gene lists within the PyWGCNA object) — https://github.com/mortazavilab/PyWGCNA
- **GSEApy** (Performs GO functional enrichment on the coral and white module gene lists)
- **Python** (Runtime for PyWGCNA (Python 3.9+ per the article; README says 3.10+) and for orchestrating the workflow steps) — https://github.com/mortazavilab/PyWGCNA

## Examples

```
from PyWGCNA import PyWGCNA; pyWGCNA_5xFAD = PyWGCNA.read('5xFAD.p'); pyWGCNA_5xFAD.analyseWGCNA(); pyWGCNA_5xFAD.functional_enrichment_analysis()
```

## Evaluation signals

- The coral module–age progression correlation and white module–hippocampal region correlation each meet the P < 0.05 criterion.
- Neutrophil activation / immune response GO terms appear in the enrichment table for the coral module (1335 genes).
- Cilium movement / organization / assembly terms appear in the enrichment table for the white module.
- module_trait_correlations.csv and module_go_enrichment.csv are produced with the expected significant associations.
- Enrichment is run at the module level using GO, KEGG, and REACTOME databases as supported in the PyWGCNA functional enrichment tutorial.

## Limitations

- Python version is required to be 3.9 or greater per the article (README states 3.10 or greater) — installation fails on older interpreters.
- The 5xFAD module names and trait associations are dataset-specific (MODEL-AD brain bulk RNA-seq); coral/white modules and their expected GO terms will not transfer to other datasets.
- Traits must be provided as external sample traits (eigengene network methodology); unsupervised networks without trait metadata cannot yield module–trait correlations.

## Evidence

- [intro] for relating modules to one another and to external sample traits (using eigengene network methodology): "for relating modules to one another and to external sample traits (using eigengene network methodology)"
- [methods] How to perform functional enrichment analysis using databases such as GO, KEGG, and REACTOME in PyWGCNA object: "`Functional enrichment analysis  <https://github.com/mortazavilab/PyWGCNA/tree/main/tutorials/functional_enrichment_analysis.ipynb>`_"
- [abstract] PyWGCNA has a faster implementation than the R version of WGCNA and several additional downstream analysis modules for functional enrichment analysis using GO, KEGG, and REACTOME: "PyWGCNA has a faster implementation than the R version of WGCNA and several additional downstream analysis modules for functional enrichment analysis using GO, KEGG, and REACTOME"
- [abstract] The data underlying this article are available in GitHub at github.com/mortazavilab/PyWGCNA/tutorials/5xFAD_paper.: "The data underlying this article are available in GitHub at github.com/mortazavilab/PyWGCNA/tutorials/5xFAD_paper."
- [readme] How to calculate correlation between modules and traits: [evidence span withheld — anchor status: external_doc]
