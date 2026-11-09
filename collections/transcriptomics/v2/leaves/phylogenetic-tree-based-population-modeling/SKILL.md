---
name: phylogenetic-tree-based-population-modeling
description: Use when simulating single-cell RNA-seq data for multiple discrete populations
  or continuous trajectories whose relationships should follow a lineage/phylogenetic
  tree, and you need to validate that the simulated heterogeneity (Sigma) preserves
  tree structure in low-dimensional embeddings.
when_to_use_negative:
- Simulating a single homogeneous population — SymSim handles this with one-population
  EVF type and no tree input is needed
- You need observed counts with library-preparation and sequencing effects rather
  than true transcript counts — True2ObservedCounts must then be applied on top, which
  this tree-modeling skill does not cover
- The populations are unrelated / no lineage or tree relationship should be imposed
  on the cell groups
edam_operation: http://edamontology.org/operation_0244
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SymSim
  role: R package providing SimulateTrueCounts (tree-based EVF simulation), Phyla5
    (five-leaf tree), and PlotTsne (visualization of simulated populations)
  repo: https://github.com/YosefLab/SymSim
- name: PCA
  role: Dimensionality reduction used alongside tSNE to inspect clustering of cells
    simulated from multiple discrete populations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:50dc4803556137801fafa75fb0faa39550973c65480389bfe51bf9c43fef355a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - PCA
  - SymSim
  edam_operation: http://edamontology.org/operation_0244
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# phylogenetic-tree-based-population-modeling

## Summary

Simulate discrete or continuous cell populations whose relationship is defined by a phylogenetic tree using SymSim's SimulateTrueCounts with extrinsic variability factors (EVFs) evolving along the tree (e.g. Brownian motion), then verify that tSNE plots of log2(true_counts+1) reflect the input tree structure as the within-population variability parameter Sigma varies.

## When to use

Use when simulating single-cell RNA-seq data for multiple discrete populations or continuous trajectories whose relationships should follow a lineage/phylogenetic tree, and you need to validate that the simulated heterogeneity (Sigma) preserves tree structure in low-dimensional embeddings.

## When NOT to use

- Simulating a single homogeneous population — SymSim handles this with one-population EVF type and no tree input is needed
- You need observed counts with library-preparation and sequencing effects rather than true transcript counts — True2ObservedCounts must then be applied on top, which this tree-modeling skill does not cover
- The populations are unrelated / no lineage or tree relationship should be imposed on the cell groups

## Inputs

- Phylogenetic tree (Newick file such as extdata/Newick_ABCDE.txt read with ape::read.tree, or an R tree object from Phyla5())
- Simulation parameter set: ncells_total, min_popsize, i_minpop, ngenes, nevf, evf_type ('discrete' or 'continuous'), n_de_evf, vary='s', Sigma, randseed
- log2-transformed true counts matrix (log2(true_counts+1)) for tSNE plotting

## Outputs

- SimulateTrueCounts result object containing the true counts matrix, cell meta information, and EVFs per cell
- tSNE plot colored by population label (discrete) or tree edge membership (continuous)
- Persisted RDS/CSV files of counts and cell metadata plus saved t-SNE figures

## How to apply

Install SymSim with devtools::install_github('YosefLab/SymSim') and load a tree, either via ape::read.tree on extdata/Newick_ABCDE.txt or by calling Phyla5(). For discrete populations run SimulateTrueCounts with evf_type='discrete' (e.g. ncells_total=300, min_popsize=50, i_minpop=2, ngenes=500, nevf=10, n_de_evf=9, vary='s', Sigma=0.4); for continuous populations use evf_type='continuous' with more EVFs (e.g. nevf=20, n_de_evf=12) so EVFs evolve continuously along tree edges. Vary the Sigma parameter to control within-population heterogeneity: larger Sigma scatters cells around the main paths that follow the tree. Call PlotTsne on log2(true_counts+1) with n_pc=20 and label='pop', coloring by population for discrete data or by tree edge membership for continuous data. Judgment: the tSNE must show five groups for the discrete case whose separation mirrors the five leaves of the input tree, and a connected trajectory structure for the continuous case.

## Related tools

- **SymSim** (R package providing SimulateTrueCounts (tree-based EVF simulation), Phyla5 (five-leaf tree), and PlotTsne (visualization of simulated populations)) — https://github.com/YosefLab/SymSim
- **PCA** (Dimensionality reduction used alongside tSNE to inspect clustering of cells simulated from multiple discrete populations)

## Examples

```
true_counts <- SimulateTrueCounts(ncells_total=300, min_popsize=50, i_minpop=2, ngenes=500, nevf=10, evf_type="discrete", n_de_evf=9, vary="s", Sigma=0.4, phyla=Phyla5(), randseed=0)
```

## Evaluation signals

- The tSNE figure for the discrete simulation shows five distinct cell groups whose separation mirrors the five leaves of the input tree
- For continuous populations, increasing Sigma causes cells to scatter around main paths that still follow the tree structure in the tSNE plot
- Different Sigma values yield visibly different within-population heterogeneity across replicate runs
- The true counts matrix dimensions match the requested ngenes x ncells_total and cell metadata includes EVF assignments
- All SimulateTrueCounts outputs and t-SNE figures are persisted to disk (RDS/CSV and image files) and are reproducible with the fixed randseed

## Limitations

- SymSim's pipeline omits several library-preparation steps, including reverse transcription and library clean-up, so true counts are not fully realistic observations
- Consequences of Sigma choices depend on EVF type: only continuous EVFs produce the scattered-around-trajectory behavior; discrete EVFs yield separate population groups
- Results depend on random seed; without fixing randseed the tree-vs-tSNE correspondence check is not reproducible

## Evidence

- [methods] tree requirement for multi-population simulation: "When there are multiple populations, users need to provide a tree."
- [methods] core simulation functions: "SimulateTrueCounts( ) and True2ObservedCounts( ) are the two major functions to generate datasets."
- [methods] tSNE visualization of simulated counts: "tsne_true_counts <- PlotTsne(meta=true_counts_res[[3]], data=log2(true_counts_res[[1]]+1), evf_type="one.population""
- [readme] SymSim package purpose: "SymSim (Synthetic model of multiple variability factors for Simulation) is an R package for simulation of single cell RNA-Seq data."
- [readme] installation command: "devtools::install_github("YosefLab/SymSim")"
