---
name: precision-medicine-subgroup-tree-fitting
description: 'Use when the goal is a treatment-subgroup analysis: a treatment/coding/interacting
  variable structure must be encoded via the types argument of dipm(), and a tree
  over baseline covariates (e.g. MASS::anorexia weight-change data) must be grown,
  split by depth-importance, and optionally pruned.'
when_to_use_negative:
- The goal is to inspect per-variable depth importance alone without growing a treatment-subgroup
  tree — compute node statistics only via node_dipm(obj)
- A forest-based node fit is not desired and an exhaustive classical split search
  is preferred — use spmtree() instead of dipm(), noting it exposes only nmin and
  maxdepth as structural tuning
- Pruning must alter optimal treatment assignments — pmprune only collapses terminal
  sister nodes with the same optimal treatment, so it is the wrong tool for cost/complexity
  pruning that changes recommendations
edam_operation: http://edamontology.org/operation_2945
tools:
- name: dipm
  role: Primary function fitting the DIPM tree/forest via formula/data/types interface
    with nmin, maxdepth and related tuning arguments
  repo: https://github.com/chenvict/dipm
- name: spmtree
  role: Alternative split selection that exhaustively evaluates all possible splits
    of all candidate variables at each node instead of the forest-based node fitting
  repo: https://cran.r-project.org/package=dipm
- name: pmprune
  role: Post-pruning routine that removes terminal sister nodes sharing the same optimal
    treatment to simplify the fitted tree
  repo: https://cran.r-project.org/package=dipm
- name: node_dipm
  role: Accessor via the partykit representation used to enumerate internal nodes
    t, splitting variable j, depth L(t), and split effect statistic Gt
  repo: https://cran.r-project.org/package=dipm
- name: partykit
  role: Dependency required for tree representation, inspection, and plotting of the
    fitted DIPM tree
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:2c56afe51ee4dde32bba75382fe6d1326bb18c62f75dac7a2fce614811699bc8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chenvict/dipm
  tools:
  - R
  - dipm
  - node_dipm
  - partykit
  - pmprune
  - spmtree
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# precision-medicine-subgroup-tree-fitting

## Summary

Fits Depth Importance in Precision Medicine (DIPM) classification trees using the dipm R package, which implements the tree/forest method through R code that calls a C program, to identify subgroups where subjects perform especially well or poorly under a particular treatment.

## When to use

Use when the goal is a treatment-subgroup analysis: a treatment/coding/interacting variable structure must be encoded via the types argument of dipm(), and a tree over baseline covariates (e.g. MASS::anorexia weight-change data) must be grown, split by depth-importance, and optionally pruned.

## When NOT to use

- The goal is to inspect per-variable depth importance alone without growing a treatment-subgroup tree — compute node statistics only via node_dipm(obj)
- A forest-based node fit is not desired and an exhaustive classical split search is preferred — use spmtree() instead of dipm(), noting it exposes only nmin and maxdepth as structural tuning
- Pruning must alter optimal treatment assignments — pmprune only collapses terminal sister nodes with the same optimal treatment, so it is the wrong tool for cost/complexity pruning that changes recommendations

## Inputs

- Treatment-subgroup dataset such as MASS::anorexia (patient weight change with treatment assignment and baseline covariates)
- R model formula relating the outcome to treatment and covariates
- types argument encoding treatment/coding/interacting variable roles
- Tuning parameters: nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores (dipm); nmin, maxdepth (spmtree)

## Outputs

- Fitted DIPM classification tree (partykit-representable object) suitable for serialization as .rds (e.g. pruned_tree.rds, spmtree_anorexia.rds)
- Per-variable depth importance table score(T,j) = sum 2^{-L(t)} * Gt, exportable as dipm_depth_importance.csv
- Pruned tree from pmprune(tree) with terminal sister nodes of identical optimal treatment collapsed

## How to apply

Load the anorexia dataset and attach dipm's dependencies explicitly (stats, utils, survival, partykit, ggplot2, rlang, grid) since only Depends is attached automatically. Fit a tree with dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune), supplying predictor roles through the formula/data interface and types argument; structural growth is constrained by nmin (minimum node size) and maxdepth. At each node, the forest-based splitting uses the depth variable importance score score(T,j) = sum over nodes t in Tj of 2^{-L(t)} * Gt, so shallower splits contribute exponentially more weight. Extract the fitted structure with node_dipm(obj) to enumerate internal nodes, splitting variables, depths L(t), and split effect statistics Gt. Simplify the result with pmprune(tree), which removes terminal sister nodes sharing the same optimal treatment without changing any treatment recommendation; alternatively call spmtree(formula, data, types, nmin, maxdepth, print, dataframe, prune), which exhaustively evaluates all possible splits of all candidate variables at each node and picks the split with the highest split-criteria score instead of fitting a random forest.

## Related tools

- **dipm** (Primary function fitting the DIPM tree/forest via formula/data/types interface with nmin, maxdepth and related tuning arguments) — https://github.com/chenvict/dipm
- **spmtree** (Alternative split selection that exhaustively evaluates all possible splits of all candidate variables at each node instead of the forest-based node fitting) — https://cran.r-project.org/package=dipm
- **pmprune** (Post-pruning routine that removes terminal sister nodes sharing the same optimal treatment to simplify the fitted tree) — https://cran.r-project.org/package=dipm
- **node_dipm** (Accessor via the partykit representation used to enumerate internal nodes t, splitting variable j, depth L(t), and split effect statistic Gt) — https://cran.r-project.org/package=dipm
- **partykit** (Dependency required for tree representation, inspection, and plotting of the fitted DIPM tree)

## Examples

```
tree <- dipm(formula, data=anorexia, types=types, nmin=nmin, maxdepth=maxdepth, prune=TRUE); pruned <- pmprune(tree); saveRDS(pruned, 'pruned_tree.rds')
```

## Evaluation signals

- depth-importance invariant: score(T,j) = sum over nodes t in Tj of 2^{-L(t)} * Gt, with shallower splits contributing exponentially more
- pmprune validation: the pruned tree retains the same optimal treatment assignments as the unpruned tree at every retained node
- spmtree validation: each internal node corresponds to the exhaustively searched split with the highest split-criteria score, not a random-forest-based split
- saved .rds objects (pruned_tree.rds, spmtree_anorexia.rds, dipm_depth_importance.csv) exist, reload, and print via partykit
- all Imports dependencies (stats, utils, survival, partykit, ggplot2, rlang, grid) attached explicitly before calling functions

## Limitations

- Packages listed under Imports are NOT attached by loading the dipm package (only Depends is); call library(<dep>) explicitly before using their functions
- spmtree exposes only nmin and maxdepth as structural tuning arguments
- No software version, changelog, or dataset accessions are stated in the article's methods/discussion sections

## Evidence

- [abstract] depth importance score: "we introduce, dipm, a novel R package that implements the DIPM method using R code that calls a program in C"
- [methods] growth parameters: "- `dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune)`"
- [methods] node accessor: "- `node_dipm(obj, ...)`"
- [methods] dependency note: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
- [intro] scope: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
- [readme] repo: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
