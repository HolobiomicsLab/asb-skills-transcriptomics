---
name: partykit-tree-visualization
description: Use when a DIPM tree (e.g. fitted to the MASS::anorexia weight-change
  data) or a pmprune-pruned version of it needs to be rendered for inspection, so
  subgroup splits and optimal treatment assignments at each node can be visually verified.
when_to_use_negative:
- The tree has not yet been fitted or pruned — plot() on a non-tree object from the
  dipm workflow will not render the DIPM node structure
- The goal is treatment-effect estimation or subgroup identification itself rather
  than visual inspection of an already-fitted DIPM tree; use dipm()/spmtree() instead
- partykit cannot be attached because its dependencies were not installed — the dipm
  package does not attach it automatically, so plotting will fail without an explicit
  library(partykit) call
edam_operation: http://edamontology.org/operation_0306
tools:
- name: partykit
  role: Dependency package of dipm used to plot and inspect the fitted/pruned DIPM
    tree object
- name: dipm (R package)
  role: Produces the DIPM tree (via dipm()) and the simplified tree (via pmprune())
    that partykit renders
  repo: https://github.com/chenvict/dipm
- name: R
  role: Runtime environment in which dipm and partykit are loaded and the tree is
    plotted
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:e00d4708e9d3ce8fe41fd32f7a7823dba3e03ace93e8081831aa3186bf626c11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chenvict/dipm
  tools:
  - R
  - dipm (R package)
  - partykit
  edam_operation: http://edamontology.org/operation_0306
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# partykit tree visualization

## Summary

Visual inspection of a fitted (and optionally pmprune-simplified) DIPM classification tree by plotting the tree object with the partykit package, which the dipm package lists as a Dependency. Plotting lets the analyst confirm the treatment-subgroup structure and that pruning collapsed only terminal sister nodes sharing the same optimal treatment.

## When to use

Use when a DIPM tree (e.g. fitted to the MASS::anorexia weight-change data) or a pmprune-pruned version of it needs to be rendered for inspection, so subgroup splits and optimal treatment assignments at each node can be visually verified.

## When NOT to use

- The tree has not yet been fitted or pruned — plot() on a non-tree object from the dipm workflow will not render the DIPM node structure
- The goal is treatment-effect estimation or subgroup identification itself rather than visual inspection of an already-fitted DIPM tree; use dipm()/spmtree() instead
- partykit cannot be attached because its dependencies were not installed — the dipm package does not attach it automatically, so plotting will fail without an explicit library(partykit) call

## Inputs

- Fitted dipm classification tree object (e.g. on MASS::anorexia weight-change data)
- Pruned tree object produced by pmprune(tree), optionally serialized as pruned_tree.rds

## Outputs

- Graphical rendering of the DIPM tree structure with node splits and optimal treatment assignments
- Visual confirmation that the pruned tree retains the same optimal treatment assignments as the unpruned tree at every retained node

## How to apply

First fit the DIPM classification tree with dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune), then simplify it with pmprune(tree), which removes terminal sister nodes whose optimal treatments coincide. Because packages listed under Dependencies (including partykit) are NOT attached automatically — only Depends packages are — explicitly call library(partykit) before plotting. Plot the tree (or the serialized pruned_tree.rds object reloaded with readRDS) with partykit's plotting to inspect the node structure and each node's optimal treatment. The visualization should confirm that pruning collapsed subtrees whose child leaves agree on the best treatment, leaving treatment recommendations unchanged.

## Related tools

- **partykit** (Dependency package of dipm used to plot and inspect the fitted/pruned DIPM tree object)
- **dipm (R package)** (Produces the DIPM tree (via dipm()) and the simplified tree (via pmprune()) that partykit renders) — https://github.com/chenvict/dipm
- **R** (Runtime environment in which dipm and partykit are loaded and the tree is plotted)

## Examples

```
library(dipm); library(partykit); pruned_tree <- pmprune(dipm(Genotype ~ Treatment + weight_change, data = anorexia)); plot(pruned_tree)
```

## Evaluation signals

- pruned_tree.rds exists and can be loaded back with readRDS before plotting
- The plotted pruned tree shows collapsed subtrees only where terminal sister nodes shared the same optimal treatment
- Every retained node in the plotted pruned tree shows the same optimal treatment assignment as the corresponding node in the unpruned tree
- library(partykit) succeeds (dependencies explicitly attached) before the plot call, since only Depends packages load automatically with the dipm package

## Limitations

- Packages listed under Dependencies (partykit, ggplot2, stats, utils, survival, rlang) are NOT attached by loading dipm — only Depends is — so plotting fails unless each is loaded explicitly
- No software version, changelog, or dataset accession details were found in the source repository, so reproducibility of the plotted tree depends on unspecified package versions
- Plotting is purely for inspection; it does not change treatment recommendations or substitute for the pmprune invariant check

## Evidence

- [methods] partykit dependency for plotting: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
- [abstract] package purpose grounding the tree object: "we introduce, dipm, a novel R package that implements the DIPM method using R code that calls a program in C"
- [readme] repository README: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
