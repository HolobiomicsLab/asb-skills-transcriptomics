---
name: exhaustive-split-search-tree-construction
description: Use when you need a classical, deterministic single--split classification
  tree (e.g. on anorexia weight-change data with treatment/coding/interacting variable
  types) rather than the DIPM random-forest-based node fitting.
when_to_use_negative:
- When you want the DIPM method itself, which fits a random forest at each node—use
  dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores,
  print, dataframe, prune) instead
- When you need forest-based tuning parameters such as ntree, mtry, or ncores—spmtree
  does not fit a random forest and exposes only nmin and maxdepth
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: dipm
  role: R package providing the spmtree function that performs exhaustive split-search
    node fitting for subgroup-identification trees
  repo: https://github.com/chenvict/dipm
- name: R
  role: runtime environment; dipm implements the DIPM method using R code that calls
    a program in C
- name: partykit
  role: dependency of dipm supporting the tree object structure/printing of the fitted
    spmtree
- name: pmprune
  role: post-pruning routine applied to the fitted spmtree when prune is set
  repo: https://github.com/chenvict/dipm
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:c1110aa8313120cdfa23a839a39cf1b9a6c514193291d3ccebbbb997d3dc387a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chenvict/dipm
  tools:
  - R
  - dipm
  - partykit
  - pmprune
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# exhaustive-split-search-tree-construction

## Summary

Fits a subgroup-identification classification tree with the dipm package's spmtree function, which—unlike the DIPM forest-based node fitting—exhaustively evaluates every possible split of every candidate split variable at each node and keeps the single split with the highest split-criteria score.

## When to use

Use when you need a classical, deterministic single-best-split classification tree (e.g. on anorexia weight-change data with treatment/coding/interacting variable types) rather than the DIPM random-forest-based node fitting.

## When NOT to use

- When you want the DIPM method itself, which fits a random forest at each node—use dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune) instead
- When you need forest-based tuning parameters such as ntree, mtry, or ncores—spmtree does not fit a random forest and exposes only nmin and maxdepth

## Inputs

- R data frame of outcome, treatment assignment, and baseline covariates (e.g. anorexia patient weight-change data)
- model formula relating outcome to treatment and covariates
- types argument encoding treatment/coding/interacting variable roles
- structural tuning arguments nmin (minimum node size) and maxdepth (maximum depth)
- optional prune flag triggering pmprune post-pruning

## Outputs

- fitted spmtree classification tree object (e.g. spmtree_anorexia.rds) whose internal nodes each hold the exhaustively searched highest-scoring split
- printed tree structure for subgroup inspection

## How to apply

Load the outcome/treatment data (e.g. anorexia patient weight change) into R and build a model formula relating the outcome to treatment and baseline covariates, supplying the types argument to encode treatment/coding/interacting variable roles. Call spmtree(formula, data, types, nmin, maxdepth, print, dataframe, prune): its node-fitting logic considers all possible splits of all candidate split variables and selects the split with the highest split-criteria score, instead of fitting a random forest at each node as dipm() does. Constrain tree growth with nmin (minimum node size) and maxdepth (maximum depth), which are the only structural tuning arguments spmtree exposes—note there is no ntree/mtry/ncores since no forest is grown. If prune is set, apply the pmprune(tree) post-pruning routine to simplify the fitted tree. Print/inspect the resulting tree structure (partykit is an import of the package) and save the fitted object, e.g. as spmtree_anorexia.rds.

## Related tools

- **dipm** (R package providing the spmtree function that performs exhaustive split-search node fitting for subgroup-identification trees) — https://github.com/chenvict/dipm
- **R** (runtime environment; dipm implements the DIPM method using R code that calls a program in C)
- **partykit** (dependency of dipm supporting the tree object structure/printing of the fitted spmtree)
- **pmprune** (post-pruning routine applied to the fitted spmtree when prune is set) — https://github.com/chenvict/dipm

## Examples

```
tree <- spmtree(formula, data = anorexia, types = types, nmin = 30, maxdepth = 3, prune = TRUE)
```

## Evaluation signals

- The saved spmtree object (e.g. spmtree_anorexia.rds) exists and can be reloaded and printed
- Each internal node corresponds to the exhaustively searched split with the highest split-criteria score rather than a random-forest-based split
- No node is smaller than nmin and tree depth does not exceed maxdepth
- If prune was set, pmprune(tree) ran and the simplified tree still contains the highest-scoring splits at retained nodes

## Limitations

- spmtree exposes only nmin and maxdepth for structural tuning; no ntree, mtry, or ncores because it does not fit a random forest at each node
- Dependencies listed under Imports (partykit, ggplot2, rlang, stats, utils, survival) are NOT attached by loading dipm; call library(<dep>) explicitly before using their functions

## Evidence

- [methods] spmtree exhaustive split search: "`spmtree(formula, data, types, nmin, maxdepth, print, dataframe, prune)`"
- [full_text] spmtree vs DIPM split selection: [evidence span withheld — anchor status: not_found]
- [intro] dipm package purpose: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
- [readme] package purpose from README: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
