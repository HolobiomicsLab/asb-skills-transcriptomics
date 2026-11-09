---
name: hvg-feature-selection
description: Use when spatial or single-cell expression data (e.g., seqFISH+ mouse
  cortex, Seq-Scope mouse colon) must be reduced to a variable-gene set before MIMO
  network construction or gene classification.
when_to_use_negative:
- The downstream goal is labeling or classifying already-selected HVGs as communication-related
  — that is the MLP classification step, not feature selection
- The analysis requires all genes (e.g., exhaustive database querying of every gene
  via the search_gene criterion) rather than only the top 2000 most variable ones
- Genes with near-zero expression are biologically important for the question, since
  zero-expression genes are removed per cell type after HVG selection
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Provides FindVariableFeatures(), the default method used to filter the top
    2000 HVGs
- name: CPPLS-MLP
  role: R/Python package whose compare_hvg.zip bundles the HVG extraction code for
    all four selection methods and the downstream consumption of the HVG set
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: M3drop
  role: Alternative HVG extraction method bundled in compare_hvg.zip for comparison
    with FindVariableFeatures()
- name: scanpy
  role: Python-based alternative HVG extraction method bundled in compare_hvg.zip
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:9d4e05370dfa5c59f654994e03e975ae9b93e39939b5fbc5826c5d9a0c869c31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - M3drop
  - Python
  - Seurat
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# hvg-feature-selection

## Summary

Selects the top highly variable genes (HVGs) from single-cell/spatial transcriptomics datasets using Seurat's FindVariableFeatures() with a 2000-gene threshold, with variance-, scanpy-, and M3drop-based extraction as alternative criteria, as the gene universe for CPPLS-MLP communication-network modeling.

## When to use

Use when spatial or single-cell expression data (e.g., seqFISH+ mouse cortex, Seq-Scope mouse colon) must be reduced to a variable-gene set before MIMO network construction or gene classification.

## When NOT to use

- The downstream goal is labeling or classifying already-selected HVGs as communication-related — that is the MLP classification step, not feature selection
- The analysis requires all genes (e.g., exhaustive database querying of every gene via the search_gene criterion) rather than only the top 2000 most variable ones
- Genes with near-zero expression are biologically important for the question, since zero-expression genes are removed per cell type after HVG selection

## Inputs

- seqFISH+ mouse cortex spatial transcriptomics count data
- Seq-Scope mouse colon spatial transcriptomics count data
- compare_hvg.zip from the CPPLS-MLP GitHub repository (packaged HVG-extraction code for FindVariableFeatures(), variance, scanpy, and M3drop)

## Outputs

- List of top 2000 HVGs per dataset, ranked by the chosen selection criterion
- HVG set restricted to genes expressed within each cell type, feeding CPPLS-MLP regression and MIMO network construction
- Per-cell-type HVG sets used to compute overlap ratios between cell types

## How to apply

Load the count matrix for the dataset of interest (seqFISH+ mouse cortex or Seq-Scope mouse colon) and extract the top 2000 HVGs with Seurat's FindVariableFeatures() using its default threshold of 2000, which is the CPPLS-MLP default. Alternatively, extract HVGs by one of the other three methods the authors provide in compare_hvg.zip — raw variance, scanpy's HVG routine, or M3drop — when a sensitivity comparison across selection criteria is desired. Restrict the selected HVG set to genes expressed in the cell types under analysis (genes with zero expression across all cells of a cell type are removed downstream). The resulting HVG list is the input universe for the CPPLS regression, MIMO graph construction, and the MLP communication-relevance classification (which identified 354 and 345 communication-related genes on seqFISH+ and Seq-Scope respectively).

## Related tools

- **Seurat** (Provides FindVariableFeatures(), the default method used to filter the top 2000 HVGs)
- **CPPLS-MLP** (R/Python package whose compare_hvg.zip bundles the HVG extraction code for all four selection methods and the downstream consumption of the HVG set) — https://github.com/wuzhenao/CPPLS-MLP
- **M3drop** (Alternative HVG extraction method bundled in compare_hvg.zip for comparison with FindVariableFeatures())
- **scanpy** (Python-based alternative HVG extraction method bundled in compare_hvg.zip)

## Evaluation signals

- The extracted HVG list contains approximately 2000 genes, consistent with FindVariableFeatures()' default threshold
- HVG extraction succeeds independently for both packaged datasets (2023_01_seqFISH+.zip and 2023_01_Seq-Scope.zip contents)
- Per-cell-type HVG overlap ratios can be computed and should be near the reported averages (0.31 in Seq-Scope, 0.25 in seqFISH+)
- The HVG set excludes genes with zero expression across all cells of a given cell type before regression modeling
- The selected HVGs suffice to reproduce the reported communication-related gene counts (354 seqFISH+, 345 Seq-Scope) in the downstream MLP classification

## Limitations

- The default top-2000 threshold is Seurat's default rather than a dataset- or tissue-optimized cutoff; alternative criteria (variance, scanpy, M3drop) may yield different HVG sets
- Only two spatial datasets (seqFISH+ and Seq-Scope) were used to assess the pipeline, so HVG-selection behavior on other tissues/technologies is unvalidated
- Version numbers for the datasets and for most tools other than R (<= 4.2.3) and Python (>= 3.7) are not documented

## Evidence

- [methods] CPPLS-MLP initially filters the top 2000 HVGs using the FindVariableFeatures() method: "CPPLS-MLP initially filters the top 2000 HVGs using the FindVariableFeatures() method"
- [readme] Four HVG extraction methods bundled in the repo: "we extracted the HVGs on the dataset based on four methods: FindVariableFeatures(), variance, scannpy, and M3drop"
- [readme] Where the HVG extraction code lives: "you can get the specific code in the folder where the HVG was extracted from the compare_hvg.zip"
- [methods] Zero-expression gene removal per cell type: "where genes with zero expression across all cells of cell type a are removed"
- [intro] Tool versions constraining the environment for HVG extraction: "dependence: R version <= 4.2.3."
