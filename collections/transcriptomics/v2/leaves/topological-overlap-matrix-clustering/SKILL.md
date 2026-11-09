---
name: topological-overlap-matrix-clustering
description: Use when a multi-sample bulk RNA-seq count matrix (e.g., 192-sample 5xFAD/C57BL/6J
  cortex and hippocampus data with age, genotype, tissue, sex traits) needs to be
  partitioned into co-expression modules tied to sample traits.
when_to_use_negative:
- The goal is comparison against an external gene list or another WGCNA object rather
  than de novo module detection — use PyWGCNA's comparison tutorials instead
- The input is not a raw/well-expressed count matrix (e.g., already-aggregated module
  eigengenes), since preprocessing per the Data_format tutorial is a prerequisite
  step
- Only a handful of samples are available, so scale-free topology fitting and TOM-based
  module detection are not well supported
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: PyWGCNA
  role: Python library that computes the soft-thresholded adjacency, TOM, and dynamic
    tree cut module detection, plus eigengene-based module-trait analysis
  repo: github.com/mortazavilab/PyWGCNA
- name: Python
  role: 'Runtime environment required to install and run PyWGCNA (README: Python 3.10
    or greater; article: Python 3.9 or greater)'
- name: WGCNA
  role: Original R reference implementation whose methodology (modules, eigengenes,
    TOM) PyWGCNA reproduces; users unfamiliar with it are directed to the original
    WGCNA publication
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:bbc557b41908a4cbe7348625b5785d671e9533d65dfdd9824b159cd1f76c291f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyWGCNA
  - Python
  - WGCNA
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# topological-overlap-matrix-clustering

## Summary

Identifies gene co-expression modules from bulk RNA-seq count matrices by building a soft-thresholded correlation adjacency, converting it to a topological overlap matrix (TOM), and cutting TOM dissimilarity with dynamic tree cut, as implemented in PyWGCNA.

## When to use

Use when a multi-sample bulk RNA-seq count matrix (e.g., 192-sample 5xFAD/C57BL/6J cortex and hippocampus data with age, genotype, tissue, sex traits) needs to be partitioned into co-expression modules tied to sample traits.

## When NOT to use

- The goal is comparison against an external gene list or another WGCNA object rather than de novo module detection — use PyWGCNA's comparison tutorials instead
- The input is not a raw/well-expressed count matrix (e.g., already-aggregated module eigengenes), since preprocessing per the Data_format tutorial is a prerequisite step
- Only a handful of samples are available, so scale-free topology fitting and TOM-based module detection are not well supported

## Inputs

- Gene-by-sample bulk RNA-seq count matrix (e.g., 192-sample 5xFAD/C57BL/6J cortex and hippocampus dataset)
- Sample trait table with metadata such as age, genotype, tissue, and sex

## Outputs

- Gene-to-module color assignment table (module_assignments.csv with gene ID and assigned module columns)
- Set of co-expression modules (e.g., 17 modules such as coral with 1335 genes and white with 435 genes)
- Module eigengenes and module-trait relationship correlations
- Soft-thresholding power selection (scale-free topology fit diagnostics)

## How to apply

Load the count matrix and sample trait table into a PyWGCNA object, first cleaning and preprocessing the data per the Data_format tutorial to filter genes with low counts so only well-expressed genes enter network construction. Choose the soft-thresholding power by scanning candidate powers with pickSoftThreshold for the best scale-free topology fit. Compute the adjacency from the scaled correlations, build the topological overlap matrix (TOM), and identify modules by applying dynamic tree cut to the TOM dissimilarity. Summarize each module by its eigengene, relate modules to external sample traits (age, genotype, tissue, sex) via eigengene network methodology and module-trait correlation, and export the gene-to-module color assignment table.

## Related tools

- **PyWGCNA** (Python library that computes the soft-thresholded adjacency, TOM, and dynamic tree cut module detection, plus eigengene-based module-trait analysis) — github.com/mortazavilab/PyWGCNA
- **Python** (Runtime environment required to install and run PyWGCNA (README: Python 3.10 or greater; article: Python 3.9 or greater))
- **WGCNA** (Original R reference implementation whose methodology (modules, eigengenes, TOM) PyWGCNA reproduces; users unfamiliar with it are directed to the original WGCNA publication)

## Evaluation signals

- Module count and composition are reproducible: rerunning on the 192-sample 5xFAD dataset should recover the 17 modules reported (coral: 1335 genes, white: 435 genes)
- Every gene passing the low-count filter receives a module color assignment, and module_assignments.csv contains exactly gene ID and assigned module columns
- The chosen soft-thresholding power corresponds to the best scale-free topology fit across the scanned candidate powers
- Module eigengenes can be correlated with the sample trait table (age, genotype, tissue, sex) to produce module-trait relationships
- Downstream checks such as GO/KEGG/REACTOME functional enrichment return biologically interpretable terms for the identified modules

## Limitations

- Genes with low counts must be filtered during preprocessing (Data_format tutorial) or the correlation network and TOM will be noisy
- Results depend on the soft-thresholding power chosen via scale-free topology fit; no fixed default is guaranteed to fit every dataset
- Practitioners unfamiliar with the R WGCNA framework are advised to read the original WGCNA publication before using PyWGCNA

## Evidence

- [abstract] We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets.: "We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets."
- [intro] It can be used for finding clusters (modules) of highly correlated genes, for summarizing such clusters using the module eigengene: "It can be used for finding clusters (modules) of highly correlated genes, for summarizing such clusters using the module eigengene"
- [methods] PyWGCNA is a Python library designed to do Weighted correlation network analysis (WGCNA): "PyWGCNA is a Python library designed to do Weighted correlation network analysis (WGCNA)"
- [readme] for relating modules to one another and to external sample traits (using eigengene network methodology), and for calculating module membership measures.: "for relating modules to one another and to external sample traits (using eigengene network methodology), and for calculating module membership measures."
