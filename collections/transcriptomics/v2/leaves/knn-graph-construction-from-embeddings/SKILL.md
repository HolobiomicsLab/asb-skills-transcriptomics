---
name: knn-graph-construction-from-embeddings
description: Use when you need a subpopulation-aware network over single-cell RNA-seq
  data (e.g., hematopoietic progenitor snapshots) so that trajectory search between
  any chosen source and target subpopulations, including rare cell types, can be performed.
when_to_use_negative:
- The goal is a simple linear pseudotime ordering of cells (e.g., Monocle 1 / Waterfall-style)
  rather than multi-state transitions between chosen subpopulations
- Only a single branch point needs to be identified and the number of fates is known
  a priori (Wishbone/DPT-style analyses)
- The analysis goal is pseudotemporal ordering via diffusion pseudotime as in the
  theislab/scAnalysisTutorial, not subpopulation-aware flow-network trajectories
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CellRouter
  role: Single-cell analysis platform that consumes the weighted kNN/community graph
    and runs the flow-network (minimum cost flow) trajectory search between subpopulations
  repo: https://github.com/edroaldo/cellrouter
- name: Rtsne
  role: Computes the dimensionality-reduced (t-SNE) embedding over which the kNN graph
    is built
- name: cccd
  role: Constructs the k-nearest-neighbor graph over the embedded cells
- name: igraph
  role: Stores the Jaccard similarity-weighted graph object and provides the community
    detection algorithms used to identify subpopulations
- name: mclust
  role: Model-based clustering used, together with community detection, to learn the
    data structure and define subpopulations on the graph
- name: R
  role: Implementation environment; required R packages are listed in the CellRouter
    README
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:ccc39b3650708fd8f1b71101f70abedee1d06ba9ced8fddb7a8ca04f6693849c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/edroaldo/cellrouter
  tools:
  - CellRouter
  - R
  - Rtsne
  - cccd
  - igraph
  - mclust
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# kNN-graph construction from single-cell embeddings (CellRouter)

## Summary

Build a k-nearest-neighbor graph over dimensionality-reduced single-cell embeddings, convert kNN edges into Jaccard similarity-weighted igraph edges, and use community detection on this graph to define subpopulations that CellRouter's flow-network trajectory search operates on. This graph representation is what enables trajectory reconstruction between any two subpopulations, including rare or intermediate cell states.

## When to use

Use when you need a subpopulation-aware network over single-cell RNA-seq data (e.g., hematopoietic progenitor snapshots) so that trajectory search between any chosen source and target subpopulations, including rare cell types, can be performed.

## When NOT to use

- The goal is a simple linear pseudotime ordering of cells (e.g., Monocle 1 / Waterfall-style) rather than multi-state transitions between chosen subpopulations
- Only a single branch point needs to be identified and the number of fates is known a priori (Wishbone/DPT-style analyses)
- The analysis goal is pseudotemporal ordering via diffusion pseudotime as in the theislab/scAnalysisTutorial, not subpopulation-aware flow-network trajectories

## Inputs

- Single-cell RNA-seq expression matrix (raw count/UMI data can be provided as input per the CellRouter README)
- Dimensionality-reduced cell embedding (e.g., t-SNE embedding computed with Rtsne)
- Choice of k for nearest-neighbor construction and requested source-target subpopulation pairs

## Outputs

- Jaccard similarity-weighted igraph graph object over cells
- Community/subpopulation annotations on graph vertices
- Ranked trajectory list (trajectories.tsv) with per-trajectory subpopulation paths and flow scores

## How to apply

In R, first compute a dimensionality-reduced embedding of the single-cell expression matrix using the Rtsne (or tsne) package; CellRouter is independent of the dimensionality-reduction technique used. Construct the k-nearest-neighbor graph over the embedded cells with the cccd package, then transform the kNN graph edges into Jaccard similarity-weighted edges stored in an igraph graph object. Identify subpopulations on this graph by combining data-structure learning with mclust (model-based clustering) and community detection algorithms from igraph. The resulting community-annotated graph is the substrate for CellRouter's minimum cost flow network search, which ranks high-resolution transition trajectories between any source and target subpopulation, including intermediate and mature cell states. Rationale: linear or branching-only methods (Waterfall, Monocle 1, Wishbone, DPT) are limited in capturing multi-state transitions, whereas the flow-network formulation on the weighted kNN graph relaxes the requirement of identifying branching points a priori.

## Related tools

- **CellRouter** (Single-cell analysis platform that consumes the weighted kNN/community graph and runs the flow-network (minimum cost flow) trajectory search between subpopulations) — https://github.com/edroaldo/cellrouter
- **Rtsne** (Computes the dimensionality-reduced (t-SNE) embedding over which the kNN graph is built)
- **cccd** (Constructs the k-nearest-neighbor graph over the embedded cells)
- **igraph** (Stores the Jaccard similarity-weighted graph object and provides the community detection algorithms used to identify subpopulations)
- **mclust** (Model-based clustering used, together with community detection, to learn the data structure and define subpopulations on the graph)
- **R** (Implementation environment; required R packages are listed in the CellRouter README)

## Examples

```
list.of.packages <- c('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd'); new.packages <- list.of.packages[!(list.of.packages %in% installed.packages()[,"Package"])]
```

## Evaluation signals

- trajectories.tsv exists, is a well-formed TSV, and contains at least one ranked trajectory path for each requested source-target subpopulation pair
- The igraph object is a valid graph with Jaccard similarity-weighted edges derived from the cccd kNN graph over the Rtsne embedding
- Every identified subpopulation (community) carries a cluster annotation consistent between mclust data-structure learning and igraph community detection
- Trajectory paths include intermediate and mature cell states, not only terminal lineages (e.g., erythroid, myeloid, lymphoid paths from HSPCs)
- R packages from the README list ('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd') are installed and the CellRouter class sources with a valid libdir

## Limitations

- CellRouter was tested in Unix-based systems (or Mac) and was not tested in Windows
- Dimensionality reduction choice is flexible but the flow-network formulation is independent of it, so poor embeddings (e.g., bad t-SNE parameters) will propagate into the graph and trajectories
- Application to single-cell chromatin accessibility, DNA methylation, and mass cytometry data was only envisioned as future work, not demonstrated in this study

## Evidence

- [abstract] CellRouter uses flow networks on subpopulation structure: "identifies complex cell-state transition trajectories by using flow networks to explore the subpopulation structure of multi-dimensional, single-cell omics data"
- [methods] Subpopulations from data-structure learning plus community detection: "Subpopulations are then identified by a combination of learning the structure of the data and community detection algorithms"
- [methods] High-resolution multi-state trajectories between any subpopulations: "High-resolution multi-state trajectories are identified from any subpopulation to any other, including intermediate and mature cell types"
- [readme] README lists the required R packages including igraph, Rtsne, cccd, mclust: "list.of.packages <- c('reshape','reshape2','pheatmap','tsne','igraph','ggplot2','mclust','grid','Rtsne','cccd')"
- [intro] CellRouter requires no a priori trajectory structure: "CellRouter requires no a priori knowledge of trajectory structure, such as number of cell fates or branches"
