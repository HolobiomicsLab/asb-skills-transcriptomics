---
name: depth-importance-score-computation
description: Use when you need to rank or select candidate splitting variables at
  nodes of a DIPM classification tree or forest for precision-medicine subgroup identification,
  from fitted dipm tree structures over a covariate dataset.
when_to_use_negative:
- When the goal is pruning an already-fit DIPM tree rather than ranking splitting
  variables — use pmprune(tree) instead
- When the analysis is survival-model subgrouping rather than DIPM classification-tree
  splitting — spmtree is the sibling method in the package for other tree types
- When the input is raw unlabeled data with no treatment-assignment outcome to partition
  — DIPM requires a classification outcome to grow the tree
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: dipm
  role: Primary R package implementing the DIPM tree/forest method and the dipm()
    fitting function whose tree structure the depth importance score is computed from;
    R code calling a C program
  repo: https://github.com/chenvict/dipm
- name: partykit
  role: Provides the party/node representation used by node_dipm(obj) to enumerate
    internal nodes, splitting variables, depths, and split effect statistics
- name: R
  role: Execution environment; dependencies (partykit, stats, utils, survival, ggplot2,
    rlang, grid) must be attached explicitly before use
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:d6a8d16a6fbc1dab557bcd2209aa8de769a4f3731fd8ed60b796b671e412dd4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chenvict/dipm
  tools:
  - R
  - dipm
  - partykit
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# depth-importance-score-computation

## Summary

Computes the DIPM depth variable importance score (Equation 1), score(T,j) = sum over nodes t split by variable j of 2^{-L(t)} * Gt, which weights each split by the exponentially decaying depth factor 2^{-L(t)} times the split effect statistic Gt, so shallower splits contribute more to variable ranking.

## When to use

Use when you need to rank or select candidate splitting variables at nodes of a DIPM classification tree or forest for precision-medicine subgroup identification, from fitted dipm tree structures over a covariate dataset.

## When NOT to use

- When the goal is pruning an already-fit DIPM tree rather than ranking splitting variables — use pmprune(tree) instead
- When the analysis is survival-model subgrouping rather than DIPM classification-tree splitting — spmtree is the sibling method in the package for other tree types
- When the input is raw unlabeled data with no treatment-assignment outcome to partition — DIPM requires a classification outcome to grow the tree

## Inputs

- Clinical/covariate dataset (e.g. the anorexia dataset) as a data.frame with treatment assignment and predictor variables
- Fitted DIPM tree or forest object (from dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune))
- Formula/data interface specification of predictor variables and their types

## Outputs

- Per-variable depth importance scores score(T,j) for each covariate j
- Sorted table of variable importance scores (e.g. dipm_depth_importance.csv)
- Ranked list of covariates identifying the best split variable at each tree node

## How to apply

Load the anorexia dataset and attach the dipm package's dependencies (partykit, stats, utils, survival, ggplot2, rlang, grid) explicitly, since only Depends is attached automatically. Grow a DIPM tree with dipm(formula, data, types, nmin, maxdepth, ...), passing predictors via the formula/data interface. Extract the fitted tree structure with node_dipm(obj) to enumerate every internal node t, its splitting variable j, its depth L(t), and its split effect statistic Gt. For each tree T in the random forest fit at a node, compute score(T,j) = sum over nodes t in Tj of 2^{-L(t)} * Gt, where Tj is the set of nodes split by variable j; shallower splits contribute exponentially more. Assemble per-variable scores into a table sorted by score (e.g. written to dipm_depth_importance.csv).

## Related tools

- **dipm** (Primary R package implementing the DIPM tree/forest method and the dipm() fitting function whose tree structure the depth importance score is computed from; R code calling a C program) — https://github.com/chenvict/dipm
- **partykit** (Provides the party/node representation used by node_dipm(obj) to enumerate internal nodes, splitting variables, depths, and split effect statistics)
- **R** (Execution environment; dependencies (partykit, stats, utils, survival, ggplot2, rlang, grid) must be attached explicitly before use)

## Examples

```
library(dipm); library(partykit); fit <- dipm(y ~ ., data=anorexia, types=..., nmin=..., maxdepth=...); nd <- node_dipm(fit)
```

## Evaluation signals

- Verify each enumerated node t from node_dipm(obj) contributes 2^{-L(t)} * Gt to the score of its splitting variable j — shallower nodes must weight more
- Cross-check that the summed score set matches the set of variables appearing as splitting variables in Tj (variables never used in a split must score 0)
- Confirm the output table is sorted by score and written to dipm_depth_importance.csv without missing-variable rows
- Recompute scores for a single tree T and confirm consistency when the same computation is repeated over all trees in the forest fit at a node
- Check that dependencies (partykit, stats, utils, survival, ggplot2, rlang, grid) were explicitly attached — loading dipm alone does not attach Imports

## Limitations

- Packages listed under Imports are NOT attached by loading the dipm package (only Depends is); functions from partykit, stats, utils, survival, ggplot2, rlang, grid must be called after explicit library() attachment
- No software version is stated in the article methods, so reproduction requires pinning from the GitHub/CRAN source
- The score is defined per tree T in the forest fit at a node; pooling across trees must respect that node-local context rather than summing globally without the tree structure

## Evidence

- [methods] Depth variable importance formula: [evidence span withheld — anchor status: not_found]
- [abstract] Package implementation: "we introduce, dipm, a novel R package that implements the DIPM method using R code that calls a program in C"
- [methods] Dependency attachment caveat: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
- [methods] dipm signature: "dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune)"
- [readme] Repository availability: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
