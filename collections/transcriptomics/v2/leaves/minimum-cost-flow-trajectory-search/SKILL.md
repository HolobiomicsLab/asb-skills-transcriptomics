---
name: minimum-cost-flow-trajectory-search
description: Use when you need transition-centered trajectories between any two chosen
  subpopulations (e.g., HSPC to erythroid/myeloid/lymphoid fates) from single-cell
  RNA-seq, without knowing the number of branches a priori.
when_to_use_negative:
- When a simple linear pseudotime ordering suffices and no source-target subpopulation
  question is posed (Waterfall/Monocle 1-style analyses)
- When the input is not single-cell omics data with detectable subpopulation structure
  — CellRouter requires learning subpopulations before flow search
- When working in Windows without a Unix-like environment, since CellRouter is untested
  on Windows
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellRouter
  role: Implements the subpopulation-aware minimum cost flow network search for multi-state
    trajectory reconstruction
  repo: https://github.com/edroaldo/cellrouter
- name: igraph
  role: Stores the Jaccard-weighted kNN graph and provides community detection for
    subpopulation annotation
- name: cccd
  role: Constructs the k-nearest-neighbor graph over cells in the reduced embedding
- name: mclust
  role: Model-based clustering used to learn the structure of the data when defining
    subpopulations
- name: Rtsne
  role: Produces the t-SNE embedding over which the kNN graph is built
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-018-03214-y
    title: Reconstruction of complex single-cell trajectories using CellRouter
  source_content_hash: sha256:88b4ab5903cf0b67d10df692aa6bccc88b09fec5fe8c6692d4e40f95a51b6282
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
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# minimum-cost-flow-trajectory-search

## Summary

CellRouter's core trajectory algorithm: convert single-cell data into a subpopulation-annotated weighted kNN graph and solve a multi-source/multi-target minimum cost flow problem to rank high-resolution cell-state transition trajectories between any two subpopulations, including rare or intermediate states.

## When to use

Use when you need transition-centered trajectories between any two chosen subpopulations (e.g., HSPC to erythroid/myeloid/lymphoid fates) from single-cell RNA-seq, without knowing the number of branches a priori.

## When NOT to use

- When a simple linear pseudotime ordering suffices and no source-target subpopulation question is posed (Waterfall/Monocle 1-style analyses)
- When the input is not single-cell omics data with detectable subpopulation structure — CellRouter requires learning subpopulations before flow search
- When working in Windows without a Unix-like environment, since CellRouter is untested on Windows

## Inputs

- Single-cell RNA-seq expression matrix (raw count/UMI data supported per updated tutorial)
- Dimensionality-reduced cell embedding (e.g., t-SNE coordinates from Rtsne)
- kNN graph over cells (cccd) converted to Jaccard-weighted igraph graph object
- Subpopulation/community annotations from mclust clustering plus igraph community detection
- User-specified source and target subpopulation pairs

## Outputs

- Ranked trajectory list with per-trajectory subpopulation paths and flow scores (trajectories.tsv)
- Multi-state trajectories showing divergence of single-cell transcriptomes along each trajectory progression

## How to apply

Start from a dimensionality-reduced embedding of the scRNA-seq matrix (e.g., t-SNE via Rtsne) and build a k-nearest-neighbor graph over cells with the cccd package. Re-weight edges using Jaccard similarity and store them in an igraph graph object. Identify subpopulations by combining data-structure learning (mclust model-based clustering) with igraph community detection, then run CellRouter's flow network search (Java libraries loaded via the 'libdir' variable in the CellRouter class) to compute minimum cost flow paths from each chosen source to each target subpopulation. The flow formulation relaxes the need to identify branching points and works independently of the dimensionality reduction technique. Rationale: unlike Monocle 2 or DPT, the minimum cost flow search is subpopulation-aware, so trajectories can pass through or terminate at rare and intermediate/transitory cell states.

## Related tools

- **CellRouter** (Implements the subpopulation-aware minimum cost flow network search for multi-state trajectory reconstruction) — https://github.com/edroaldo/cellrouter
- **igraph** (Stores the Jaccard-weighted kNN graph and provides community detection for subpopulation annotation)
- **cccd** (Constructs the k-nearest-neighbor graph over cells in the reduced embedding)
- **mclust** (Model-based clustering used to learn the structure of the data when defining subpopulations)
- **Rtsne** (Produces the t-SNE embedding over which the kNN graph is built)

## Examples

```
source('path/to/CellRouter_Class.R'); libdir <- 'path/to/CellRouter/'
```

## Evaluation signals

- trajectories.tsv exists, parses as a well-formed TSV, and contains at least one ranked trajectory path for each requested source-target subpopulation pair
- Each trajectory path visits a coherent sequence of annotated subpopulations connecting the chosen source to the target
- Trajectories include expected intermediate/transitory cell states (e.g., lineage-committed progenitors in HSPC differentiation to erythroid, myeloid and lymphoid lineages)
- Flow scores are present for every returned trajectory, enabling ranking and comparison across source-target pairs
- The igraph graph object is correctly Jaccard-weighted and every node carries a community/subpopulation label before flow search

## Limitations

- Thorough functional characterization of genes identified by CellRouter for generating engraftable HSC-like cells is deferred to future work and beyond the scope of the manuscript
- Application to single-cell chromatin accessibility, DNA methylation, and mass cytometry data is only envisioned as future work, not demonstrated
- Requires Java libraries correctly located via 'libdir'; CellRouter was tested only on Unix-based systems and Mac, not Windows

## Evidence

- [abstract] CellRouter, a multifaceted single-cell analysis platform that identifies complex cell-state transition trajectories by using flow networks: "Here, we present CellRouter, a multifaceted single-cell analysis platform that identifies complex cell-state transition trajectories by using flow networks"
- [methods] High-resolution multi-state trajectories identified between any subpopulations including intermediate and mature cell types: "High-resolution multi-state trajectories are identified from any subpopulation to any other, including intermediate and mature cell types"
- [methods] Subpopulations identified by combining data-structure learning with community detection: "Subpopulations are then identified by a combination of learning the structure of the data and community detection algorithms"
- [intro] CellRouter implements a flow network algorithm to reconstruct multi-state transition trajectories: "CellRouter relaxes the requirement of identifying branching points during cell-fate transitions and implements a flow network algorithm to flexibly reconstruct multi-state transition trajectories"
- [readme] README: source the CellRouter class and set libdir for Java libraries: "source the CellRouter class and make sure you provide the correct directory for the Java libraries inside the folder "CellRouter", as specified by the variable 'libdir'"
- [intro] Monocle 2 not designed for trajectories between any two chosen cell states including rare types: "Monocle 2 addresses many of these challenges but is not designed to reconstruct trajectories between any two chosen cell states, which might include transitions from or to rare cell types"
