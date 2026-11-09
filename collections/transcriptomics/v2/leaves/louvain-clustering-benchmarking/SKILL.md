---
name: louvain-clustering-benchmarking
description: Use when a simulated or ground-truth-labelled scRNA-Seq dataset is available
  and you need to quantify whether cell-attribute aware EVA community detection recovers
  known labels better than a Louvain-analog, via ARI and chi-square DA statistics.
when_to_use_negative:
- Datasets with strong clustering structure — the article notes ELVAR's improvement
  over its Louvain-analog was more pronounced in studies with weaker clustering structure
  and less noticeable in stronger ones
- Input is a real dataset with no simulated or trusted ground-truth labels, making
  adjusted Rand Index against truth impossible to compute
- Purity parameter (alpha) cannot be varied — EVA only outperformed Louvain at larger
  purity parameter values in the benchmark
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ELVAR
  role: R-package providing Eva_partitions (cell-attribute aware EVA clustering),
    SelOptAlpha (optimal alpha selection), ProcessEVA, and DoDA for the benchmark
    pipeline
  repo: https://github.com/aet21/ELVAR
- name: EVA
  role: Node-attribute aware community detection algorithm, a generalization of Louvain,
    applied across the alpha purity grid
  repo: https://github.com/aet21/ELVAR
- name: Seurat
  role: Builds the Seurat object, identifies variable features, scales data, runs
    PCA, and constructs the nearest-neighbour graph prior to benchmarking
- name: igraph
  role: Builds the undirected cell-cell state-manifold graph (graph.adjacency) and
    hosts the condition vertex attribute
- name: R
  role: Implementation environment for the entire benchmark pipeline (ELVAR, Seurat,
    Matrix, dplyr)
  repo: https://github.com/aet21/ELVAR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-39017-z
    title: Cell-attribute aware community detection improves differential abundance
      testing from single-cell RNA-Seq data
  source_content_hash: sha256:26fd58bfbbda171d7c00cddfe6715b8f208cf0f015262dd2a94535c9b2e26cf5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aet21/ELVAR
  tools:
  - ELVAR
  - EVA
  - Matrix
  - R
  - Seurat
  - dplyr
  - igraph
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# louvain-clustering-benchmarking

## Summary

Benchmarks EVA (cell-attribute aware, Extended Louvain) clustering against the standard Louvain baseline on a simulated 200-cell Tabula Muris Senis monocyte scRNA-Seq dataset, quantifying recovery of simulated perturbation labels via adjusted Rand Index and chi-square differential-abundance tests across the alpha purity-parameter grid (0.1–0.9).

## When to use

Use when a simulated or ground-truth-labelled scRNA-Seq dataset is available and you need to quantify whether cell-attribute aware EVA community detection recovers known labels better than a Louvain-analog, via ARI and chi-square DA statistics.

## When NOT to use

- Datasets with strong clustering structure — the article notes ELVAR's improvement over its Louvain-analog was more pronounced in studies with weaker clustering structure and less noticeable in stronger ones
- Input is a real dataset with no simulated or trusted ground-truth labels, making adjusted Rand Index against truth impossible to compute
- Purity parameter (alpha) cannot be varied — EVA only outperformed Louvain at larger purity parameter values in the benchmark

## Inputs

- Simulated scRNA-Seq count matrix (e.g. 200 cells: 100 perturbed vs 100 normal classical monocytes derived from Tabula Muris Senis lung data)
- Ground-truth condition labels and simulated cell-state assignments per cell
- Cell-cell nearest-neighbour state-manifold graph (igraph object) with condition label as vertex attribute
- Alpha purity-parameter grid (seq(0.1, 0.9, 0.1))

## Outputs

- EVA partitions per alpha value from Eva_partitions (eva.o objects)
- Optimal-alpha selection table from SelOptAlpha (purity and modularity values)
- Adjusted Rand Index values comparing each inferred partition to ground-truth labels (EVA vs Louvain)
- Chi-square statistics and P-values from chisq.test on condition-by-state contingency tables for enriched communities

## How to apply

Build the cell-cell k-nearest-neighbour state-manifold graph in igraph (undirected, self-loops removed) from a Seurat pipeline (FindVariableFeatures → ScaleData → RunPCA → FindNeighbors) and attach the simulated condition label as a vertex attribute. Run ELVAR's Eva_partitions across the alpha purity grid (alpha = 0.1 to 0.9, where alpha trades purity against modularity) with resolution=1 and threshold=1e-7, then run the Louvain baseline (alpha=0 behaviour) as reference. Use SelOptAlpha to select the optimal alpha from purity/modularity tables, and ProcessEVA to extract significant communities. Compute adjusted Rand Index against the ground-truth simulated labels for each partition and compare condition-by-state contingency tables between EVA and Louvain with chisq.test for DA sensitivity. Expect EVA to show higher ARI and more significant chi-square P-values only at larger purity (alpha) values, consistent with Supplementary Fig. S1.

## Related tools

- **ELVAR** (R-package providing Eva_partitions (cell-attribute aware EVA clustering), SelOptAlpha (optimal alpha selection), ProcessEVA, and DoDA for the benchmark pipeline) — https://github.com/aet21/ELVAR
- **EVA** (Node-attribute aware community detection algorithm, a generalization of Louvain, applied across the alpha purity grid) — https://github.com/aet21/ELVAR
- **Seurat** (Builds the Seurat object, identifies variable features, scales data, runs PCA, and constructs the nearest-neighbour graph prior to benchmarking)
- **igraph** (Builds the undirected cell-cell state-manifold graph (graph.adjacency) and hosts the condition vertex attribute)
- **R** (Implementation environment for the entire benchmark pipeline (ELVAR, Seurat, Matrix, dplyr)) — https://github.com/aet21/ELVAR

## Examples

```
eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha=0.8, Vattr.name="condition"); selAlpha.o <- SelOptAlpha(nC.m,nc0,pur.m,mod.m,alpha.v=seq(0.1,0.9,0.1));
```

## Evaluation signals

- EVA at the optimal alpha should yield a higher adjusted Rand Index against the simulated ground-truth labels than the Louvain baseline
- Chi-square P-values from chisq.test on EVA's enriched communities should be more significant than Louvain's for detecting the perturbation state
- Improvement over Louvain should appear only at larger alpha (purity) values in the 0.1–0.9 grid, not across all alphas
- SelOptAlpha should return a well-defined optimal alpha from the purity/modularity tables (nC.m, nc0, pur.m, mod.m)
- The graph used must be undirected with self-loops removed and carry the condition label as a vertex attribute before Eva_partitions is called

## Limitations

- The improvement over Louvain appeared only at larger purity parameter (alpha) values in the simulated monocyte benchmark
- Cell-attribute aware community detection comes at increased computational complexity and runtimes compared to Milo or the Louvain-analog (deemed not practically prohibitive)
- ELVAR is sensitive to false positives in the clustering attribute (sample/phenotype label), which can affect the quality of inferred communities
- Identifying an optimal alpha may require ~100 runs per alpha across nine alphas (900 runs) unless parallelized

## Evidence

- [methods] eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha= 0.8, Vattr.name= "age");: "eva.o <- Eva_partitions(gr.o, resolution=1, threshold=0.0000001, alpha= 0.8, Vattr.name= "age");"
- [methods] selAlpha.o <- SelOptAlpha(nC.m,nc0,pur.m,mod.m,alpha.v=seq(0.1,0.9,0.1));: "selAlpha.o <- SelOptAlpha(nC.m,nc0,pur.m,mod.m,alpha.v=seq(0.1,0.9,0.1));"
- [discussion] the improvement of ELVAR over its Louvain-analog was more pronounced in studies displaying weaker clustering structure: "the improvement of ELVAR over its Louvain-analog was more pronounced in studies displaying weaker clustering structure"
- [abstract] Using simulated and real single-cell and single-nucleus RNA-Seq datasets, we benchmark ELVAR against an analogous algorithm that uses Louvain: "Using simulated and real single-cell and single-nucleus RNA-Seq datasets, we benchmark ELVAR against an analogous algorithm that uses Louvain"
- [intro] we here adapt a node-attribute aware community detection algorithm called EVA17, which is a generalization of the very popular Louvain clustering method: "we here adapt a node-attribute aware community detection algorithm called EVA17, which is a generalization of the very popular Louvain clustering method"
