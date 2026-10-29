---
name: terminal-sister-node-merging
description: Use when a fitted DIPM tree (e.g. from dipm() on the MASS::anorexia weight-change
  data) contains terminal sister leaves whose optimal treatment assignments coincide
  and you want a smaller tree that preserves all treatment recommendations.
when_to_use_negative:
- The object is a DIPM forest (multiple trees, ntree > 1) rather than a single fitted
  tree, since pmprune operates on a tree object
- The task requires changing the treatment recommendation or refitting split points
  — pmprune only collapses sister leaves with identical optimal treatments and does
  not alter any recommendation
tools:
- name: dipm (R package)
  role: Provides the pmprune(tree) function that performs the terminal-sister-node
    pruning and the dipm() function that fits the tree being pruned
  repo: https://github.com/chenvict/dipm
- name: dipm
  role: Fitting function for the DIPM classification tree passed to pmprune
  repo: https://cran.r-project.org/package=dipm
- name: partykit
  role: Used to plot and inspect the pruned tree object after pruning
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:3ab2261cc35ccb6e8a3773046c66f7a05556aaa7530a085d5945c96a716c60c0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chenvict/dipm
  tools:
  - R
  - dipm
  - dipm (R package)
  - partykit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# terminal-sister-node-merging

## Summary

Simplifies a fitted DIPM classification tree by removing terminal sister nodes that share the same optimal treatment, via the dipm package's pmprune function, so the tree structure collapses without altering any treatment recommendation.

## When to use

Use when a fitted DIPM tree (e.g. from dipm() on the MASS::anorexia weight-change data) contains terminal sister leaves whose optimal treatment assignments coincide and you want a smaller tree that preserves all treatment recommendations.

## When NOT to use

- The object is a DIPM forest (multiple trees, ntree > 1) rather than a single fitted tree, since pmprune operates on a tree object
- The task requires changing the treatment recommendation or refitting split points — pmprune only collapses sister leaves with identical optimal treatments and does not alter any recommendation

## Inputs

- Fitted DIPM classification tree object produced by dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune), e.g. fitted on the MASS::anorexia weight-change data
- Formula and data.frame inputs defining the treatment-subgroup analysis

## Outputs

- Pruned DIPM tree object with terminal sister nodes sharing the same optimal treatment collapsed
- Serialized pruned tree file (e.g. pruned_tree.rds)
- Optional partykit plot of the pruned tree

## How to apply

Fit the DIPM classification tree with dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune) using default splitting parameters for the treatment-subgroup analysis. Before loading the package, explicitly attach its Imports dependencies (stats, utils, survival, partykit, ggplot2, rlang, grid) with library() calls, because only Depends packages are attached automatically. Then call pmprune(tree), whose mechanism is to remove terminal sister nodes of the tree that share the same optimal treatment, collapsing subtrees whose child leaves agree on the best treatment assignment. Serialize the result (e.g. to pruned_tree.rds) and optionally plot it with partykit for inspection.

## Related tools

- **dipm (R package)** (Provides the pmprune(tree) function that performs the terminal-sister-node pruning and the dipm() function that fits the tree being pruned) — https://github.com/chenvict/dipm
- **dipm** (Fitting function for the DIPM classification tree passed to pmprune) — https://cran.r-project.org/package=dipm
- **partykit** (Used to plot and inspect the pruned tree object after pruning)

## Examples

```
pruned_tree <- pmprune(tree); saveRDS(pruned_tree, "pruned_tree.rds")
```

## Evaluation signals

- pruned_tree.rds exists and can be loaded back into R
- The pruned tree retains the same optimal treatment assignments as the unpruned tree at every retained node
- Every pair of terminal sister nodes in the pruned tree differs in optimal treatment (any that agreed were collapsed)
- Number of terminal nodes in the pruned tree is <= the unpruned tree, with no change to root treatment recommendation
- Pruned tree plots successfully with partykit

## Limitations

- No software version stated for the dipm package in the article's methods section
- No changelog found in the source repository (github.com/chenvict/dipm), so version-specific pruning behavior cannot be pinned down from context
- Packages listed under Imports are not attached when the dipm package is loaded; omitting explicit library() calls for stats, utils, survival, partykit, ggplot2, rlang, grid can cause failures downstream

## Evidence

- [methods] dependency attachment: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
- [intro] package scope: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
- [abstract] implementation: "we introduce, dipm, a novel R package that implements the DIPM method using R code that calls a program in C"
- [readme] readme: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
