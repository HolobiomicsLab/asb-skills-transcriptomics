---
name: tsne-visualization-of-simulated-counts
description: Use when you have run SymSim's SimulateTrueCounts for discrete or continuous
  populations on a phyla tree (e.g. Phyla5()) and need to verify the simulated cell
  organization reflects the tree, or to inspect how Sigma changes population heterogeneity.
when_to_use_negative:
- Input is observed counts from a real single-cell dataset rather than SymSim true
  counts — PlotTsne as used here operates on simulated log2(true_counts+1) with known
  population labels
- The analysis requires trajectory inference on real data — use dynverse tools (dynwrap/dynmethods)
  instead of simulated-tree validation
- Only one homogeneous population with no tree structure or population labels exists,
  so there is nothing to compare against tree leaves
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3523
tools:
- name: SymSim
  role: Generates the simulated true counts and cell metadata via SimulateTrueCounts,
    and renders the t-SNE via PlotTsne; tree supplied by Phyla5()
  repo: https://github.com/YosefLab/SymSim
- name: PCA
  role: Dimensionality reduction upstream of t-SNE (n_pc=20 principal components of
    log2 true counts feed PlotTsne)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:8eeb6e8b333b7cc781468253c9af38cbc330c7ebb354dc9dbb55a52a9d70e4d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - PCA
  - SymSim
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# tSNE visualization of simulated counts

## Summary

Visualize SymSim-simulated true counts with t-SNE (via PlotTsne on log2-transformed counts) to check that discrete and continuous cell populations from an input tree structure separate or connect as expected. This validates that the simulator's EVF-based variability parameter Sigma produces heterogeneity patterns consistent with the generative tree.

## When to use

Use when you have run SymSim's SimulateTrueCounts for discrete or continuous populations on a phyla tree (e.g. Phyla5()) and need to verify the simulated cell organization reflects the tree, or to inspect how Sigma changes population heterogeneity.

## When NOT to use

- Input is observed counts from a real single-cell dataset rather than SymSim true counts — PlotTsne as used here operates on simulated log2(true_counts+1) with known population labels
- The analysis requires trajectory inference on real data — use dynverse tools (dynwrap/dynmethods) instead of simulated-tree validation
- Only one homogeneous population with no tree structure or population labels exists, so there is nothing to compare against tree leaves

## Inputs

- Five-leaf phyla tree (extdata/Newick_ABCDE.txt read with ape::read.tree, or Phyla5() output)
- SimulateTrueCounts result: true counts matrix and cell metadata with EVFs (true_counts_res[[1]] and true_counts_res[[3]])
- Simulation parameters: ncells_total, ngenes, nevf, evf_type ('discrete' or 'continuous'), n_de_evf, Sigma, randseed

## Outputs

- t-SNE scatter plots colored by population (discrete) or tree edge membership (continuous)
- Persisted true counts matrix and cell metadata (RDS/CSV) with EVF values

## How to apply

Load the tree with ape::read.tree on the package's extdata/Newick_ABCDE.txt or generate it with Phyla5(). Run SimulateTrueCounts — for discrete populations use ncells_total=300, min_popsize=50, i_minpop=2, ngenes=500, nevf=10, evf_type='discrete', n_de_evf=9, vary='s', Sigma=0.4; for continuous populations use ncells_total=500, ngenes=500, nevf=20, evf_type='continuous', n_de_evf=12, Sigma=0.4. Call PlotTsne on log2(true_counts+1) with n_pc=20, coloring cells by population for the discrete case and by tree edge membership for the continuous case. Vary Sigma to examine how within-population variability modulates heterogeneity; increasing Sigma scatters continuous cells around the main trajectory paths. Persist the true-counts matrix and cell metadata (including EVFs) as RDS/CSV and export the t-SNE figures to disk.

## Related tools

- **SymSim** (Generates the simulated true counts and cell metadata via SimulateTrueCounts, and renders the t-SNE via PlotTsne; tree supplied by Phyla5()) — https://github.com/YosefLab/SymSim
- **PCA** (Dimensionality reduction upstream of t-SNE (n_pc=20 principal components of log2 true counts feed PlotTsne))

## Examples

```
library(SymSim); phyla <- Phyla5(); res <- SimulateTrueCounts(ncells_total=300, min_popsize=50, i_minpop=2, ngenes=500, nevf=10, evf_type='discrete', n_de_evf=9, vary='s', Sigma=0.4, phyla=phyla, randseed=0); tsne <- PlotTsne(meta=res[[3]], data=log2(res[[1]]+1), n_pc=20, label='pop', evf_type='discrete')
```

## Evaluation signals

- The discrete-simulation t-SNE figure shows exactly five cell groups whose separation mirrors the five leaves of the input tree
- The continuous-simulation t-SNE shows a connected trajectory structure whose paths follow the tree edges
- Increasing Sigma visibly increases within-population scatter in the t-SNE while preserving the overall tree-driven layout
- PlotTsne was called on log2(true_counts+1) with n_pc=20, not on raw counts
- Saved artifacts include both the t-SNE figures and the SimulateTrueCounts outputs (true counts matrix and cell meta with EVFs) in RDS/CSV form

## Limitations

- Results depend on Sigma: high within-population variability blurs population separation in the t-SNE, so the check is parameter-dependent
- The simulated pipeline omits several steps including reverse transcription and library cleanup, so t-SNE patterns reflect true counts rather than full wet-lab distortion
- t-SNE on simulated data validates the simulator against the input tree; it is not evidence that a real dataset follows the same tree

## Evidence

- [methods] SimulateTrueCounts generates true transcript counts for the given number of genes and cells: "SimulateTrueCounts generates true transcript counts for the given number of genes and cells"
- [methods] PlotTsne usage on log2 true counts: "tsne_true_counts <- PlotTsne(meta=true_counts_res[[3]], data=log2(true_counts_res[[1]]+1), evf_type="one.population""
- [methods] Tree generation: "A tree with five leaves (five populations) can be generated as follows"
- [readme] SymSim installation from GitHub: "devtools::install_github('YosefLab/SymSim')"
- [abstract] SymSim purpose: "we present SymSim, a simulator that explicitly models the processes that give rise to data observed in single cell RNA-Seq experiments."
