---
name: dipm-decision-tree-fitting
description: 'Use when a precision-medicine question asks which treatment is optimal
  for which patient subgroups: e.g., a treatment indicator plus covariates (such as
  anorexia pre-weight) and a continuous response like post-weight are available.'
when_to_use_negative:
- The goal is a survival or multivariate structured response rather than a classification-style
  treatment-optimality outcome (use spmtree() or other methods instead of the DIPM
  tree).
- The analysis requires only pruning an already-fitted tree rather than growing one
  (pmprune(tree) alone suffices).
- No treatment variable is present, so subgroup treatment comparisons are not meaningful.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: dipm
  role: Primary function for fitting the DIPM classification tree; R package whose
    code calls a C program
  repo: https://github.com/chenvict/dipm
- name: dipm (R package)
  role: Package dependency providing spmtree, pmprune, and node_dipm companion functions
    for tree fitting, pruning, and node inspection
  repo: https://cran.r-project.org/package=dipm
- name: partykit
  role: Imported dependency for representing/visualizing the fitted tree structure
- name: stats
  role: Imported R package used for base statistical routines during tree fitting
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:1974f4dd7e4b20cc791936b35a08f2a29a950e4f0934dc2f70849ddd464622d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chenvict/dipm
  tools:
  - R
  - dipm
  - dipm (R package)
  - partykit
  - stats
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# dipm-decision-tree-fitting

## Summary

Fit a Depth Importance in Precision Medicine (DIPM) classification tree to identify subgroups where a particular treatment performs especially well or poorly, using the dipm R package (R code calling a C program).

## When to use

Use when a precision-medicine question asks which treatment is optimal for which patient subgroups: e.g., a treatment indicator plus covariates (such as anorexia pre-weight) and a continuous response like post-weight are available.

## When NOT to use

- The goal is a survival or multivariate structured response rather than a classification-style treatment-optimality outcome (use spmtree() or other methods instead of the DIPM tree).
- The analysis requires only pruning an already-fitted tree rather than growing one (pmprune(tree) alone suffices).
- No treatment variable is present, so subgroup treatment comparisons are not meaningful.

## Inputs

- R data.frame with a response variable (e.g., PostWeight), a treatment-group factor (e.g., Cont, CBT, FT in MASS::anorexia), and covariates (e.g., pre-weight)
- R formula specifying response ~ predictors, with predictor types declared via the 'types' argument

## Outputs

- Fitted DIPM tree object (serializable with saveRDS, e.g., anorexia_dipm_tree.rds) listing splitting variables and terminal-node subgroups
- Per-subgroup optimal treatment assignments (e.g., family treatment optimal in both identified subgroups)
- Optional partykit-renderable tree structure via the partykit import

## How to apply

Load the dataset (e.g., MASS::anorexia with 72 observations of treatment group, pre-weight, and post-weight) and call dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune) from the dipm R package, passing a formula with the response (post-weight) and predictors (treatment group, pre-weight); set 'types' to declare which predictors are treatment variables versus covariates. Note that packages listed under Imports (stats, partykit, ggplot2, rlang, utils) are NOT attached by loading dipm, so call library(<dep>) explicitly. The tree search uses a depth importance score to preferentially split on treatment variables, yielding terminal-node subgroups with an optimal treatment assignment in each; companion functions spmtree() and pmprune() fit/prune treatment-specific models and node_dipm() inspects individual nodes. Inspect the fitted tree to extract splitting variables and terminal-node subgroups, verify contrasts (e.g., CBT-vs-control more profound in the higher pre-weight subgroup), and save the result with saveRDS.

## Related tools

- **dipm** (Primary function for fitting the DIPM classification tree; R package whose code calls a C program) — https://github.com/chenvict/dipm
- **dipm (R package)** (Package dependency providing spmtree, pmprune, and node_dipm companion functions for tree fitting, pruning, and node inspection) — https://cran.r-project.org/package=dipm
- **partykit** (Imported dependency for representing/visualizing the fitted tree structure)
- **stats** (Imported R package used for base statistical routines during tree fitting)

## Examples

```
tree <- dipm(PostWeight ~ ., data = anorexia_df, types = c("treatment", "covariate"), dataframe = TRUE); saveRDS(tree, "anorexia_dipm_tree.rds")
```

## Evaluation signals

- The tree object identifies the expected splitting variables (treatment group and pre-weight) and terminal-node subgroups.
- Family treatment (FT) is optimal in both identified subgroups of the anorexia example.
- The CBT-vs-control effect is more profound in the higher pre-weight subgroup, matching the reported finding.
- Tree respects nmin/nmin2 node-size constraints and maxdepth/maxdepth2 depth limits; prune step via pmprune yields the final tree.
- Fitted tree round-trips through saveRDS/readRDS (e.g., anorexia_dipm_tree.rds) and renders under partykit.

## Limitations

- No software version is stated in the article's methods, so reproducibility requires noting the dipm version manually.
- No changelog found in the source repository, so tracking behavioral changes between package versions is difficult.
- Packages under Imports are not attached by loading dipm (only Depends is); users must call library(<dep>) explicitly for stats, partykit, etc.
- The DIPM forest mode requires setting forest-related arguments (ntree, mtry) in the same dipm() call; tree and forest hyperparameters (maxdepth vs maxdepth2) must be set deliberately.

## Evidence

- [methods] DIPM tree fit via dipm(): "dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune)"
- [methods] Package loading caveat: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
- [abstract] Method purpose: "The Depth Importance in Precision Medicine (DIPM) method is a classification tree designed for the identification of subgroups relevant to the precision medicine setting."
- [abstract] Implementation mechanism: "we introduce, dipm, a novel R package that implements the DIPM method using R code that calls a program in C"
- [readme] Package identity: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
