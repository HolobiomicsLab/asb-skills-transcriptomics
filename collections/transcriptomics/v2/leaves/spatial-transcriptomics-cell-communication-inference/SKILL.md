---
name: spatial-transcriptomics-cell-communication-inference
description: Use when you have spatial transcriptomics data with single-cell resolution
  (gene expression matrix U, spatial coordinate matrix O, cell type labels D) and
  need to infer which neighboring cell types regulate HVG expression and build a directional
  MIMO communication network.
when_to_use_negative:
- Input data lacks spatial coordinate information — CPPLS-MLP requires the coordinate
  matrix O to compute adjacent cell type scores, so non-spatial scRNA-seq alone is
  insufficient
- Goal is ligand-receptor database lookup rather than data-driven inference of communication-linked
  HVGs — the MLP classification criterion is specific to CPPLS-MLP's approach
- Image-derived cell locations only exist as a proposal, not an implemented component
  — do not rely on this skill for image-processing-based cell segmentation from tissue
  slice images
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CPPLS-MLP
  role: Core method combining Constrained Partial Least Squares regression and Multilayer
    Perceptron to construct MIMO cell-cell communication networks and classify communication-associated
    HVGs
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: R (version <= 4.2.3)
  role: Runtime environment for running the 2023_01_Seq-Scope.RMD pipeline that performs
    CPPLS regression modeling and network construction
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Seurat
  role: Provides FindVariableFeatures() for selecting the top 2000 HVGs as dependent
    variables in the CPPLS regression
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: pls
  role: R package supplying the partial least squares machinery used in the constrained
    CPPLS regression modeling step
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: cluster
  role: R package used for k-means clustering of filtered HVG coefficients and Silhouette-based
    selection of optimal k (2-15)
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: circlize
  role: R package installed as a dependency for visualization of the cell-cell communication
    results
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: Python (>= 3.7)
  role: Used with networkx 3.2.1 for MIMO system integration into a directed graph
    in the 'a directed graph' code file
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: networkx 3.2.1
  role: Python library used to build the directional MIMO graph from cell-cell communication
    pathways
  repo: https://github.com/wuzhenao/CPPLS-MLP
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:3453eb2deac4688cffa5e11c67e137b3cca48022f0cef47309aea93adfd79f6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - Python (>= 3.7)
  - R
  - R (version <= 4.2.3)
  - Seurat
  - circlize
  - cluster
  - networkx 3.2.1
  - pls
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# spatial-transcriptomics-cell-communication-inference

## Summary

Constructs MIMO cell-cell communication networks from single-cell resolution spatial transcriptomics data using CPPLS-MLP, which combines Constrained Partial Least Squares regression with Multilayer Perceptron-based classification to identify highly variable genes (HVGs) involved in intercellular communication.

## When to use

Use when you have spatial transcriptomics data with single-cell resolution (gene expression matrix U, spatial coordinate matrix O, cell type labels D) and need to infer which neighboring cell types regulate HVG expression and build a directional MIMO communication network.

## When NOT to use

- Input data lacks spatial coordinate information — CPPLS-MLP requires the coordinate matrix O to compute adjacent cell type scores, so non-spatial scRNA-seq alone is insufficient
- Goal is ligand-receptor database lookup rather than data-driven inference of communication-linked HVGs — the MLP classification criterion is specific to CPPLS-MLP's approach
- Image-derived cell locations only exist as a proposal, not an implemented component — do not rely on this skill for image-processing-based cell segmentation from tissue slice images

## Inputs

- Seq-Scope gene expression matrix U (single-cell resolution spatial transcriptomics)
- Spatial coordinate matrix O
- Cell type label vector D
- 2023_01_Seq-Scope.zip package from CPPLS-MLP GitHub repository
- 2023_01_seqFISH+.zip package (alternative dataset)

## Outputs

- Network edge-list file of predicted cell-cell communication edges (e.g., immature B cell/IgA B cell-fibroblast)
- Directional MIMO graph aggregating bipartite communication pathways
- List of cell-communication-related HVGs (e.g., 345 genes identified on Seq-Scope)
- Gene cluster assignments from k-means clustering of HVG coefficients

## How to apply

Load the gene expression matrix U, spatial coordinate matrix O, and cell type label vector D in R (version <= 4.2.3). Preprocess per cell type by removing genes with zero expression across all cells of that type, z-score normalizing remaining genes, and computing raw adjacent cell type scores from coordinates via log10(dist/dist0) with per-cell-type z-score transformation. Run CPPLS regression with 10-fold cross-validation to estimate the coefficient matrix W of neighboring-cell-type regulation on the top 2000 HVGs (selected via FindVariableFeatures()), then apply two-step coefficient filtering: t-tests on factor loadings with Benjamini-Hochberg FDR at delta = 0.05, followed by a second P-value filter on remaining genes. Filter out HVGs with zero coefficients, cluster the rest with k-means (k chosen from 2 to 15 via the Silhouette method), aggregate bipartite graphs into the MIMO graph with path attenuation (10^(1-i) decay per intermediate cell), and standardize gene cluster counts across cell types.

## Related tools

- **CPPLS-MLP** (Core method combining Constrained Partial Least Squares regression and Multilayer Perceptron to construct MIMO cell-cell communication networks and classify communication-associated HVGs) — https://github.com/wuzhenao/CPPLS-MLP
- **R (version <= 4.2.3)** (Runtime environment for running the 2023_01_Seq-Scope.RMD pipeline that performs CPPLS regression modeling and network construction) — https://github.com/wuzhenao/CPPLS-MLP
- **Seurat** (Provides FindVariableFeatures() for selecting the top 2000 HVGs as dependent variables in the CPPLS regression) — https://github.com/wuzhenao/CPPLS-MLP
- **pls** (R package supplying the partial least squares machinery used in the constrained CPPLS regression modeling step) — https://github.com/wuzhenao/CPPLS-MLP
- **cluster** (R package used for k-means clustering of filtered HVG coefficients and Silhouette-based selection of optimal k (2-15)) — https://github.com/wuzhenao/CPPLS-MLP
- **circlize** (R package installed as a dependency for visualization of the cell-cell communication results) — https://github.com/wuzhenao/CPPLS-MLP
- **Python (>= 3.7)** (Used with networkx 3.2.1 for MIMO system integration into a directed graph in the 'a directed graph' code file) — https://github.com/wuzhenao/CPPLS-MLP
- **networkx 3.2.1** (Python library used to build the directional MIMO graph from cell-cell communication pathways) — https://github.com/wuzhenao/CPPLS-MLP

## Examples

```
run 2023_01_Seq-Scope.RMD
```

## Evaluation signals

- The 2023_01_Seq-Scope.RMD script runs to completion in R (<= 4.2.3) without errors
- The output network edge-list file exists and contains edges connecting both the B cell (immature/IgA) and fibroblast cell types
- The resulting MIMO graph is renderable as a directional graph (cf. Supplementary Figure S8A)
- Two-step coefficient filtering retains only coefficients with BH-adjusted P-values below the delta = 0.05 threshold
- HVGs with coefficients equal to 0 are excluded before k-means clustering, and Silhouette method selects k within 2-15

## Limitations

- No accession IDs or download sources are provided in the article for the Seq-Scope and seqFISH+ datasets; they must be obtained from the GitHub zip packages
- No implementation details (architecture, hyperparameters, training procedure) are given for the MLP component
- Only two benchmark datasets (Seq-Scope mouse colon and seqFISH+ mouse cortex) were used for performance assessment
- Label extraction for filtering genes is a manual database-querying process against GeneCards and GO with no scripted or automated procedure
- Inconsistent naming of the tool across the text (CPPLS-MLP vs CPPLS_MLP)
- Requires R version <= 4.2.3, which constrains the environment

## Evidence

- [methods] Two-step coefficient filtering with BH FDR at delta = 0.05: [evidence span withheld — anchor status: not_found]
- [methods] MIMO graph with path attenuation: "CPPLS-MLP calculates pathways from cell type a to all other cell types and applies attenuation to each path k"
- [methods] Top 2000 HVGs via FindVariableFeatures(): "CPPLS-MLP initially filters the top 2000 HVGs using the FindVariableFeatures() method"
- [methods] k-means with Silhouette k selection 2-15: "CPPLS-MLP employs the k-means method to cluster the filtered coefficients of HVGs [38]. The optimal number of clusters k is determined using the Silhouette method"
- [intro] Datasets applied: "Applied to Seq-scope and seqFISH+ datasets, CPPLS-MLP unveils the regulatory role of intercellular communication across various cell types on HVG expression."
- [readme] Repo and R version: "R package for estimating cell-cell communications from spatial transcriptome data with single-cell resolution."
- [readme] Run instructions: "If you want to use CPPLS to build a cell-to-cell communication network, you can decompress the 2023_01_seqFISH+.zip and 2023_01_Seq-Scope.zip packages and run them in R"
- [readme] MIMO integration step: "If you want to get this result, you need to unzip the compare_hvg.zip, which contains the solution for all paths for each cell, and get the MIMO system in a code file called a directed graph."
- [discussion] GPX1 validation finding: "we visualized the expression levels of the GPX1 gene in both cell types and its spatial distribution. This visualization confirmed the accuracy of our predictions"
- [methods] Gene count finding: "our CPPLS-MLP method identified 354 and 345 genes related to cell communication, respectively"
