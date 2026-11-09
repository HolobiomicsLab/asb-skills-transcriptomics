---
name: bulk-rna-seq-co-expression-network-analysis
description: Use when a bulk RNA-seq expression matrix (e.g., a 192-sample count/TPM
  matrix) plus sample trait metadata (age, genotype, tissue, sex) must be partitioned
  into co-expression modules and those modules related to traits.
when_to_use_negative:
- Input is single-cell expression data rather than bulk RNA-seq — single-cell data
  appears in this workflow only as external marker-gene lists for comparison, not
  as network input
- The question is differential expression between two conditions, not discovering
  clusters of co-expressed genes across many samples
- Input is already a module assignment or eigengene table — use the comparison/enrichment
  tutorials instead of re-running module detection
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: PyWGCNA
  role: 'Core library: filters low-count genes, computes scaled-correlation adjacency
    and TOM, detects modules by dynamic tree cut, correlates eigengenes with traits,
    and runs GO/KEGG/REACTOME enrichment and PPI analysis'
  repo: https://github.com/mortazavilab/PyWGCNA
- name: Python 3
  role: Runtime environment required to install and run PyWGCNA
- name: WGCNA (R)
  role: Original R reference implementation; recommended reading for users unfamiliar
    with WGCNA methodology
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad415
    title: 'PyWGCNA: a Python package for weighted gene co-expression network analysis'
  source_content_hash: sha256:4a84a9009a4c77b5feb791580c4d99147548094c2aa514d2d457d71a23a4f261
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mortazavilab/PyWGCNA
  tools:
  - PyWGCNA
  - Python
  - Python 3
  - WGCNA (R)
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# bulk-rna-seq-co-expression-network-analysis

## Summary

Detects gene co-expression modules from bulk RNA-seq data using PyWGCNA, a Python implementation of weighted gene co-expression network analysis (WGCNA) that is faster than the R version and adds downstream functional enrichment, PPI, and module-comparison analyses. Applied to the 192-sample 5xFAD/C57BL/6J cortex-and-hippocampus dataset it recovered 17 modules associated with age, genotype, tissue, and sex.

## When to use

Use when a bulk RNA-seq expression matrix (e.g., a 192-sample count/TPM matrix) plus sample trait metadata (age, genotype, tissue, sex) must be partitioned into co-expression modules and those modules related to traits.

## When NOT to use

- Input is single-cell expression data rather than bulk RNA-seq — single-cell data appears in this workflow only as external marker-gene lists for comparison, not as network input
- The question is differential expression between two conditions, not discovering clusters of co-expressed genes across many samples
- Input is already a module assignment or eigengene table — use the comparison/enrichment tutorials instead of re-running module detection

## Inputs

- Bulk RNA-seq count or TPM matrix (genes x samples), e.g., the 192-sample 5xFAD/C57BL/6J cortex and hippocampus dataset from github.com/mortazavilab/PyWGCNA/tutorials/5xFAD_paper
- Sample trait/metadata table (e.g., age, genotype, tissue, sex for each of the 192 samples)
- Optional: external gene-marker list or a second PyWGCNA object for comparison

## Outputs

- Gene-to-module assignment table (module_assignments.csv with columns for gene ID and assigned module color; 17 modules in the 5xFAD case)
- Module eigengene–trait correlation table/heatmap (module associations with age, genotype, tissue, sex)
- Soft-thresholding power selection plot (scale-free topology fit)
- Optional: GO/KEGG/REACTOME functional enrichment per module, STRING PPI networks, and cross-dataset module overlap comparisons

## How to apply

Load the count matrix and sample trait table into a PyWGCNA object, preprocessing per the Data_format tutorial by filtering genes with low counts so only well-expressed genes enter network construction. Select the soft-thresholding power by scanning candidate powers for the best scale-free topology fit using pickSoftThreshold, then compute the adjacency from the scaled correlations, build the topological overlap matrix (TOM), and cut modules with dynamic tree cut on the TOM dissimilarity — on the 5xFAD dataset this yields 17 modules (e.g., coral with 1335 genes, white with 435 genes). Export the gene-to-module color assignment table as module_assignments.csv with columns for gene ID and assigned module. Quantify module–trait associations by correlating module eigengenes with the trait table (the module_trait_relationships_heatmap tutorial), and optionally run GO/KEGG/REACTOME enrichment, STRING-based PPI recovery, or comparison against other PyWGCNA objects or external gene-marker lists. Rationale: module eigengenes summarize each correlated gene cluster so modules can be related to one another and to external sample traits via eigengene network methodology.

## Related tools

- **PyWGCNA** (Core library: filters low-count genes, computes scaled-correlation adjacency and TOM, detects modules by dynamic tree cut, correlates eigengenes with traits, and runs GO/KEGG/REACTOME enrichment and PPI analysis) — https://github.com/mortazavilab/PyWGCNA
- **Python 3** (Runtime environment required to install and run PyWGCNA)
- **WGCNA (R)** (Original R reference implementation; recommended reading for users unfamiliar with WGCNA methodology)

## Examples

```
pip install PyWGCNA
```

## Evaluation signals

- Module count and sizes are reproducible: the 5xFAD 192-sample run recovers 17 modules, with coral at 1335 genes and white at 435 genes
- The chosen soft-thresholding power achieves a good scale-free topology fit in the pickSoftThreshold scan
- module_assignments.csv has exactly two columns (gene ID and assigned module) and every well-expressed gene maps to one module color
- Module–trait correlation heatmap shows significant associations with age, genotype, tissue, and sex
- Dynamic tree cut operates on the TOM dissimilarity (not the raw adjacency), matching the documented workflow order

## Limitations

- Version requirement discrepancy: methods state Python 3.9 or greater is required, while the README states 3.10 or greater — check the release you install
- Gene filtering of low-count genes before network construction changes which genes/modules are recovered, so preprocessing choices must be reported for reproducibility
- WGCNA requires many samples to estimate reliable correlations; the reference implementation is the slower R version, and users unfamiliar with it are advised to read the original WGCNA publication

## Evidence

- [abstract] abstract: "We introduce the PyWGCNA Python package, which is designed to identify co-expression modules from large RNA-seq datasets."
- [readme] readme: "can be used for finding clusters (modules) of highly correlated genes, for summarizing such clusters using the module eigengene"
- [methods] methods: "How to load data into PyWGCNA, find modules, and analyze them"
- [readme] readme-traits: [evidence span withheld — anchor status: external_doc]
- [readme] readme-install: [evidence span withheld — anchor status: external_doc]
