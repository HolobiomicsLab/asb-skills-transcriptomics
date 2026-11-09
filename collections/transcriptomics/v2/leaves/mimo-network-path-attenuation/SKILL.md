---
name: mimo-network-path-attenuation
description: Use when pairwise cell-cell communication networks (e.g., from CPPLS-MLP
  on Seq-Scope or seqFISH+ data) must be integrated into a single MIMO directed graph
  with attenuated multi-cell-type paths.
when_to_use_negative:
- Input is still raw gene expression and coordinate matrices — CPPLS regression modeling
  and coefficient filtering must be completed before MIMO aggregation
- You only need a single pairwise cell-type communication network rather than an integrated
  multi-cell-type MIMO system
- The data lack single-cell spatial resolution (coordinates and cell type labels),
  since path attenuation requires tracing multi-hop cell-type pathways
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CPPLS-MLP
  role: Produces the filtered bipartite communication graphs and implements path attenuation
    when aggregating them into the MIMO graph
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: networkx 3.2.1
  role: Python library used to build and export the MIMO system as a directed graph
  repo: https://github.com/wuzhenao/CPPLS-MLP
- name: cluster
  role: R package providing k-means clustering of filtered HVG coefficients (k in
    2-15 via Silhouette) prior to MIMO graph construction
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bib/bbae198
    title: 'CPPLS-MLP: a method for constructing cell–cell communication networks
      and identifying related highly variable genes based on single-cell sequencing
      and spatial transcriptomics data'
  source_content_hash: sha256:6f6765a8406e25bc6616503bd5309c6b2725973bb38918739cdd6d8d039196e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wuzhenao/CPPLS-MLP
  tools:
  - CPPLS-MLP
  - R
  - Seurat
  - circlize
  - cluster
  - networkx 3.2.1
  - pls
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# mimo-network-path-attenuation

## Summary

Aggregates pairwise cell-type communication (bipartite) graphs produced by CPPLS-MLP into a Multiple Input Multiple Output (MIMO) directed graph, applying path attenuation (10^(1-i) decay per intermediate cell type) so that indirect multi-hop communication paths contribute less weight than direct edges.

## When to use

Use when pairwise cell-cell communication networks (e.g., from CPPLS-MLP on Seq-Scope or seqFISH+ data) must be integrated into a single MIMO directed graph with attenuated multi-cell-type paths.

## When NOT to use

- Input is still raw gene expression and coordinate matrices — CPPLS regression modeling and coefficient filtering must be completed before MIMO aggregation
- You only need a single pairwise cell-type communication network rather than an integrated multi-cell-type MIMO system
- The data lack single-cell spatial resolution (coordinates and cell type labels), since path attenuation requires tracing multi-hop cell-type pathways

## Inputs

- Bipartite cell-type communication graphs (filtered CPPLS regression coefficient matrices between cell types)
- Filtered HVG coefficient matrices (zero-coefficient HVGs removed)
- Spatial transcriptomics datasets: Seq-Scope mouse colon (2023_01_Seq-Scope.zip) and seqFISH+ mouse cortex (2023_01_seqFISH+.zip)
- Gene cluster assignments from k-means (k in 2-15, Silhouette-selected)

## Outputs

- MIMO directed graph integrating pairwise bipartite graphs with path-attenuated weights (10^(1-i) per intermediate cell)
- Network edge-list file (e.g., predicted immature B cell/IgA B cell-fibroblast communication edges)
- Standardized gene cluster counts across cell types

## How to apply

After CPPLS regression modeling and two-step coefficient filtering (t-tests on factor loadings with Benjamini-Hochberg FDR at delta = 0.05), first filter out HVGs with coefficients equal to 0, then cluster the remaining HVG coefficients with k-means, selecting k from 2 to 15 via the Silhouette method, and standardize gene cluster counts across cell types. Next, calculate all pathways from cell type a to every other cell type from the bipartite graphs and apply attenuation to each path, decaying the weight by 10^(1-i) for each intermediate cell type i, so longer indirect paths contribute proportionally less. Merge the attenuated paths into a MIMO system and export it as a directed graph using Python with networkx 3.2.1 (the README delivers this step via the code in compare_hvg.zip). The resulting MIMO graph can then be rendered as a directional network (cf. Supplementary Figure S8A) and its edges (e.g., immature B cell/IgA B cell to fibroblast) exported as an edge-list file.

## Related tools

- **CPPLS-MLP** (Produces the filtered bipartite communication graphs and implements path attenuation when aggregating them into the MIMO graph) — https://github.com/wuzhenao/CPPLS-MLP
- **networkx 3.2.1** (Python library used to build and export the MIMO system as a directed graph) — https://github.com/wuzhenao/CPPLS-MLP
- **cluster** (R package providing k-means clustering of filtered HVG coefficients (k in 2-15 via Silhouette) prior to MIMO graph construction)

## Examples

```
unzip compare_hvg.zip  # then run the directed-graph code file (networkx 3.2.1) to obtain the MIMO system
```

## Evaluation signals

- The MIMO directed graph exists and contains edges connecting both the B cell (immature/IgA) and fibroblast cell types
- The MIMO graph is renderable as a directional graph (cf. Supplementary Figure S8A)
- Path weights decay with the number of intermediate cells according to the 10^(1-i) attenuation factor
- HVGs with coefficients equal to 0 are absent from the clustered/merged graph
- Gene cluster counts are standardized across cell types before graph aggregation

## Limitations

- Attenuation assumes indirect paths through intermediate cell types should be exponentially down-weighted (10^(1-i)); this decay choice is not benchmarked in the article
- Performance was assessed on only two datasets (Seq-Scope mouse colon and seqFISH+ mouse cortex), so generalization to other tissues/platforms is untested
- The README requires manually unzipping compare_hvg.zip and locating the directed-graph code file; no single end-to-end script automates the MIMO step
- The tool naming is inconsistent across the text (CPPLS-MLP vs CPPLS_MLP), and no versioned release accompanies the repository

## Evidence

- [methods] CPPLS-MLP calculates pathways from cell type a to all other cell types and applies attenuation to each path k: "CPPLS-MLP calculates pathways from cell type a to all other cell types and applies attenuation to each path k"
- [readme] MIMO construction from pairwise networks in README: "After building a communication network between two cells, we have integrated the cell communication network into a MIMO system."
- [readme] compare_hvg.zip contains path solutions and directed-graph code: "you need to unzip the compare_hvg.zip, which contains the solution for all paths for each cell, and get the MIMO system in a code file called a directed graph"
- [methods] k-means clustering with Silhouette-selected k precedes MIMO aggregation: "CPPLS-MLP employs the k-means method to cluster the filtered coefficients of HVGs [38]. The optimal number of clusters k is determined using the Silhouette method"
- [methods] Zero-coefficient HVGs removed before clustering: "Before clustering, all the HVGs with coefficients equal to 0 are filtered out."
