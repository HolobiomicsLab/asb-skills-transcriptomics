---
name: dynamic-tree-cut-module-detection
description: Use when you have a gene-by-sample RNA-seq expression matrix (e.g., the
  192-sample 5xFAD/C57BL/6J cortex and hippocampus bulk RNA-seq dataset) and need
  to detect clusters of highly correlated genes that can be related to sample traits
  such as age, genotype, tissue, and sex.
when_to_use_negative:
- Input data has not been filtered for low-count genes — poorly expressed genes will
  degrade the correlation network and should be removed first per the Data_format
  preprocessing tutorial
- The goal is differential expression of individual genes rather than detecting co-expression
  modules or relating them to external sample traits
- Samples are too few to estimate a stable weighted correlation network across conditions
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: PyWGCNA
  role: 'Python library that computes the weighted correlation network: adjacency
    from scaled correlations, TOM construction, and dynamic tree cut module identification,
    with downstream eigengene-trait correlation and module comparison'
  repo: https://github.com/mortazavilab/PyWGCNA
- name: Python
  role: Runtime for PyWGCNA (Python 3.9/3.10 or greater required) used to run the
    module detection workflow
  repo: https://github.com/mortazavilab/PyWGCNA
- name: WGCNA
  role: Original R reference implementation of weighted correlation network analysis
    that PyWGCNA re-implements; suggested reading when unfamiliar with the methodology
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:334e15abc13e93711177345a0dda71b147bba46cb525481e3e2b9d0fc5c1e21f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - PyWGCNA
  - Python
  - WGCNA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# dynamic-tree-cut-module-detection

## Summary

Identifies co-expression modules by building a weighted adjacency from scaled gene correlations, converting it to a topological overlap matrix (TOM), and applying dynamic tree cut on the TOM dissimilarity to assign genes to modules. Implemented in PyWGCNA as a faster Python alternative to R WGCNA.

## When to use

Use when you have a gene-by-sample RNA-seq expression matrix (e.g., the 192-sample 5xFAD/C57BL/6J cortex and hippocampus bulk RNA-seq dataset) and need to detect clusters of highly correlated genes that can be related to sample traits such as age, genotype, tissue, and sex.

## When NOT to use

- Input data has not been filtered for low-count genes — poorly expressed genes will degrade the correlation network and should be removed first per the Data_format preprocessing tutorial
- The goal is differential expression of individual genes rather than detecting co-expression modules or relating them to external sample traits
- Samples are too few to estimate a stable weighted correlation network across conditions

## Inputs

- Gene-by-sample RNA-seq count matrix (e.g., 192-sample 5xFAD/C57BL/6J cortex and hippocampus bulk RNA-seq)
- Sample trait/annotation table (four ages, both sexes, genotype, tissue)
- Candidate soft-thresholding powers to scan

## Outputs

- Gene-to-module color assignment table (module_assignments.csv with gene ID and assigned module)
- Module eigengenes for relating modules to sample traits
- Detected module set (e.g., 17 modules; coral 1335 genes, white 435 genes)
- Module-trait association results

## How to apply

Load the count matrix and sample trait table into a PyWGCNA object and preprocess per the Data_format tutorial, filtering low-count genes to retain only well-expressed genes for network construction. Select the soft-thresholding power by scanning candidate powers for the best scale-free topology fit using pickSoftThreshold. Compute the adjacency from the scaled correlations, build the topological overlap matrix (TOM), and identify modules with dynamic tree cut on the TOM dissimilarity. Export the gene-to-module color assignments (e.g., as module_assignments.csv with gene ID and assigned module columns) and summarize each module with its eigengene for downstream trait correlation, functional enrichment (GO/KEGG/REACTOME), and cross-dataset module comparison.

## Related tools

- **PyWGCNA** (Python library that computes the weighted correlation network: adjacency from scaled correlations, TOM construction, and dynamic tree cut module identification, with downstream eigengene-trait correlation and module comparison) — https://github.com/mortazavilab/PyWGCNA
- **Python** (Runtime for PyWGCNA (Python 3.9/3.10 or greater required) used to run the module detection workflow) — https://github.com/mortazavilab/PyWGCNA
- **WGCNA** (Original R reference implementation of weighted correlation network analysis that PyWGCNA re-implements; suggested reading when unfamiliar with the methodology)

## Examples

```
pip install PyWGCNA
```

## Evaluation signals

- The soft-thresholding power was chosen by scanning candidates for the best scale-free topology fit (pickSoftThreshold), not set arbitrarily
- The output gene-to-module table (module_assignments.csv) contains a column for gene ID and a column for the assigned module color for every gene entering the network
- On the 5xFAD tutorial data, PyWGCNA recovers 17 gene co-expression modules associated with age, genotype, tissue, and sex (e.g., coral module with 1335 genes, white module with 435 genes)
- Module eigengenes can be correlated with external sample traits, and modules support downstream functional enrichment (GO, KEGG, REACTOME) without errors
- Networks built from different datasets can be compared and show significant module overlap where co-expression signatures are shared

## Limitations

- Module results depend on preprocessing choices: low-count genes must be filtered before network construction to retain only well-expressed genes
- The Python implementation requires Python 3.9 (article) / 3.10 (README) or greater
- Users unfamiliar with WGCNA are advised to read the original R WGCNA publication, since the package assumes understanding of the underlying methodology

## Evidence

- [readme] It can be used for finding clusters (modules) of highly correlated genes, for summarizing such clusters using the module eigengene: "It can be used for finding clusters (modules) of highly correlated genes, for summarizing such clusters using the module eigengene"
- [readme] for relating modules to sample traits: "for relating modules to one another and to external sample traits (using eigengene network methodology), and for calculating module membership measures"
- [other] Compute the adjacency from scaled correlations, build the TOM, and run dynamic tree cut: [evidence span withheld — anchor status: not_found]
- [other] 17 modules recovered from the 192-sample 5xFAD dataset: [evidence span withheld — anchor status: not_found]
- [other] Selecting the soft-thresholding power: [evidence span withheld — anchor status: not_found]
