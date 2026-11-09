---
name: coexpression-module-biological-interpretation
description: Use when you already have gene-to-module assignments from a co-expression
  network (e.g., PyWGCNA modules like the coral module) and need to relate modules
  to sample traits and identify enriched GO biological functions per module.
when_to_use_negative:
- The goal is to build the co-expression network itself (detect modules from raw RNA-seq
  counts) rather than interpret already-assigned modules
- You need to compare PyWGCNA objects against each other or against external gene
  marker lists for conservation/overlap rather than per-module functional enrichment
- Input lacks sample trait metadata, so module-trait eigengene correlations cannot
  be computed
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: PyWGCNA
  role: Python library used to relate modules to external sample traits via eigengene
    network methodology and to perform functional enrichment analysis using GO, KEGG,
    and REACTOME on module gene lists
  repo: https://github.com/mortazavilab/PyWGCNA
- name: GSEApy
  role: Runs GO functional enrichment on the coral and white module gene lists to
    recover the expected term categories (neutrophil activation; cilium organization)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:805281432537e192ae3824252a0b6764d4d8e4b873593cb66d65ea2b91e4d276
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - GSEApy
  - PyWGCNA
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# coexpression-module-biological-interpretation

## Summary

Interprets co-expression modules produced by WGCNA by correlating module eigengenes with external sample traits (e.g., age progression, brain region) and running GO/KEGG/REACTOME functional enrichment on module gene lists, using PyWGCNA with GSEApy-style enrichment. This converts gene clusters from a model like the 5xFAD MODEL-AD bulk RNA-seq network into testable biological functions.

## When to use

Use when you already have gene-to-module assignments from a co-expression network (e.g., PyWGCNA modules like the coral module) and need to relate modules to sample traits and identify enriched GO biological functions per module.

## When NOT to use

- The goal is to build the co-expression network itself (detect modules from raw RNA-seq counts) rather than interpret already-assigned modules
- You need to compare PyWGCNA objects against each other or against external gene marker lists for conservation/overlap rather than per-module functional enrichment
- Input lacks sample trait metadata, so module-trait eigengene correlations cannot be computed

## Inputs

- Gene-to-module assignment table from a prior WGCNA run (e.g., PyWGCNA module assignments from task_001)
- Sample trait metadata for the bulk RNA-seq dataset (e.g., age progression, brain region for the 5xFAD MODEL-AD data)
- Module eigengene matrix or PyWGCNA object containing detected modules
- Gene lists per module (e.g., the 1335-gene coral module)

## Outputs

- module_trait_correlations.csv — significant module-eigengene vs. trait correlations with P values (retaining P < 0.05)
- module_go_enrichment.csv — GO (and optionally KEGG/REACTOME) enrichment results per module gene list

## How to apply

Load the gene-to-module table from the network step and the sample trait metadata (e.g., age progression, brain region) from the bulk RNA-seq dataset, then use PyWGCNA to compute correlations between module eigengenes and external traits via the eigengene network methodology, retaining associations with P < 0.05 (e.g., coral module vs. age progression, white module vs. hippocampal region). Export these significant associations to a table such as module_trait_correlations.csv. Next, run functional enrichment on each module's gene list using databases such as GO, KEGG, and REACTOME (e.g., via GSEApy for GO biological process terms), writing results to module_go_enrichment.csv. In the 5xFAD example, this recovers immune response and neutrophil activation terms for the 1335-gene coral module and cilium movement/organization/assembly terms for the white module.

## Related tools

- **PyWGCNA** (Python library used to relate modules to external sample traits via eigengene network methodology and to perform functional enrichment analysis using GO, KEGG, and REACTOME on module gene lists) — https://github.com/mortazavilab/PyWGCNA
- **GSEApy** (Runs GO functional enrichment on the coral and white module gene lists to recover the expected term categories (neutrophil activation; cilium organization))

## Evaluation signals

- The coral–age progression and white–hippocampus module-trait correlations each satisfy the P < 0.05 significance criterion
- The GO enrichment table for the coral module contains immune response / neutrophil activation terms
- The GO enrichment table for the white module contains cilium movement / organization / assembly terms
- The coral module gene count matches the expected 1335 genes
- Both module_trait_correlations.csv and module_go_enrichment.csv are written with the expected term categories present

## Limitations

- Functional enrichment depends on the availability of the chosen annotation databases (GO, KEGG, REACTOME); results differ per database
- Module–trait associations are limited to external sample traits supplied as metadata; without trait data, only enrichment-based interpretation is possible
- The paper's benchmark is bulk RNA-seq from MODEL-AD mouse brain datasets (e.g., 5xFAD); transfer to other data types is not demonstrated in the provided context

## Evidence

- [abstract] We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets.: "We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets."
- [intro] for relating modules to one another and to external sample traits (using eigengene network methodology): "for relating modules to one another and to external sample traits (using eigengene network methodology)"
- [readme] How to perform functional enrichment analysis using databases such as GO, KEGG, and REACTOME in PyWGCNA object: "How to perform functional enrichment analysis using databases such as GO, KEGG, and REACTOME in PyWGCNA object"
- [intro] Users can also compare WGCNA networks from different datasets, or to external gene lists, to assess the conservation or functional enrichment of each module.: "Users can also compare WGCNA networks from different datasets, or to external gene lists, to assess the conservation or functional enrichment of each module."
- [readme] How to calculate correlation between modules and traits: [evidence span withheld — anchor status: not_found]
