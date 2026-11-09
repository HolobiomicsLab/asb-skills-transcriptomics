---
name: r-package-reproducible-scripting
description: Use when fitting a precision-medicine subgroup classification tree in
  R where a single split per node (highest split-criteria score from exhaustive split
  search) is preferred over the forest-based DIPM node fitting, e.g. on the anorexia
  patient weight-change dataset.
when_to_use_negative:
- When forest-based node fitting of the DIPM method is desired — use dipm(formula,
  data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe,
  prune) instead of spmtree
- When the analysis goal is not treatment-subgroup identification for precision medicine
  (the DIPM method targets subgroups where subjects perform especially well or poorly
  under a treatment assignment)
- When only nmin/maxdepth structural control is insufficient — spmtree exposes no
  other tuning arguments (no ntree/mtry/ncores), so it cannot mimic forest behavior
edam_operation: http://edamontology.org/operation_2945
tools:
- name: dipm
  role: R package providing the spmtree function whose node-fitting exhaustively evaluates
    all possible splits of all candidate split variables and selects the highest split-criteria
    score; R code calls a program in C
  repo: https://github.com/chenvict/dipm
- name: partykit
  role: Imported dependency of dipm providing the tree object structure; not attached
    automatically, must be loaded explicitly with library()
- name: pmprune
  role: Post-pruning routine applied to the fitted spmtree object when the prune argument
    is set
- name: R
  role: Scripting environment in which the reproducible workflow (formula construction,
    spmtree call, pruning, print/inspect, .rds serialization) is executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:a0f50e2dcf818887db6b24f5d36da1ce951b6fb512ee4c215853a36f42cbed93
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chenvict/dipm
  tools:
  - R
  - dipm
  - partykit
  - pmprune
  edam_operation: http://edamontology.org/operation_2945
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# r-package-reproducible-scripting

## Summary

A reproducible R scripting pattern for running the dipm package's classical tree fitter spmtree, which exhaustively searches all possible splits of all candidate variables at each node instead of fitting a random forest, and persisting the fitted tree object as an .rds file.

## When to use

Use when fitting a precision-medicine subgroup classification tree in R where a single best split per node (highest split-criteria score from exhaustive split search) is preferred over the forest-based DIPM node fitting, e.g. on the anorexia patient weight-change dataset.

## When NOT to use

- When forest-based node fitting of the DIPM method is desired — use dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune) instead of spmtree
- When the analysis goal is not treatment-subgroup identification for precision medicine (the DIPM method targets subgroups where subjects perform especially well or poorly under a treatment assignment)
- When only nmin/maxdepth structural control is insufficient — spmtree exposes no other tuning arguments (no ntree/mtry/ncores), so it cannot mimic forest behavior

## Inputs

- anorexia patient weight-change dataset loaded as an R data frame
- model formula relating outcome to treatment assignment and baseline covariates
- types argument encoding treatment/coding/interacting variable roles
- structural tuning parameters: nmin (minimum node size) and maxdepth (maximum depth)

## Outputs

- fitted spmtree object (partykit-style tree structure with exhaustively searched splits)
- optional pruned tree via pmprune(tree)
- serialized RDS file, e.g. spmtree_anorexia.rds

## How to apply

Load the anorexia patient weight-change data into R and build a model formula relating the outcome to treatment assignment and baseline covariates, passing the types argument to encode treatment/coding/interacting variable roles. Call spmtree(formula, data, types, nmin, maxdepth, print, dataframe, prune) from the dipm package (R code that calls a C program); its node-fitting logic exhaustively evaluates every possible split of every candidate variable at each node and keeps the split with the highest split-criteria score, in contrast to dipm()'s forest-based node fitting. Constrain tree growth with only the structural tuning arguments the function exposes: nmin (minimum node size) and maxdepth (maximum depth). If prune is set, apply the pmprune(tree) post-pruning routine to simplify the fitted tree. Note that packages listed under Dependencies are NOT attached by loading dipm — call library(<dep>) explicitly before using their functions (e.g. partykit for tree handling). Finally, print/inspect the tree structure and save the fitted object as spmtree_anorexia.rds.

## Related tools

- **dipm** (R package providing the spmtree function whose node-fitting exhaustively evaluates all possible splits of all candidate split variables and selects the highest split-criteria score; R code calls a program in C) — https://github.com/chenvict/dipm
- **partykit** (Imported dependency of dipm providing the tree object structure; not attached automatically, must be loaded explicitly with library())
- **pmprune** (Post-pruning routine applied to the fitted spmtree object when the prune argument is set)
- **R** (Scripting environment in which the reproducible workflow (formula construction, spmtree call, pruning, print/inspect, .rds serialization) is executed)

## Examples

```
R> tree <- spmtree(formula, data, types, nmin, maxdepth, print, dataframe, prune); pmprune(tree)
```

## Evaluation signals

- The saved spmtree object (e.g. spmtree_anorexia.rds) exists and can be reloaded with readRDS and printed
- Each internal node corresponds to the exhaustively searched split with the highest split-criteria score rather than a random-forest-based split
- Tree growth respects the declared nmin (minimum node size) and maxdepth (maximum depth) constraints
- When prune was set, the returned tree reflects the pmprune(tree) post-pruning output
- Explicit library() calls cover the Imports dependencies (partykit, ggplot2, rlang, stats, utils) since only Depends is attached by loading dipm

## Limitations

- spmtree exposes only nmin and maxdepth as structural tuning arguments; it has no ntree, mtry, ncores, nmin2, or maxdepth2 parameters
- spmtree is the more classical approach (exhaustive split search), not the random-forest-based DIPM node fitting, so results differ from dipm()
- No software version stated, no dataset names or accessions given in the methods section, and no changelog found in the source repository
- Packages listed under Dependencies are NOT attached by loading the package (only Depends is)

## Evidence

- [methods] spmtree signature: "- `spmtree(formula, data, types, nmin, maxdepth, print, dataframe, prune)`"
- [readme] dipm package purpose: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and Forest-based method"
- [abstract] R + C implementation: "we introduce, dipm, a novel R package that implements the DIPM method using R code that calls a program in C"
- [methods] dependency attachment behavior: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
