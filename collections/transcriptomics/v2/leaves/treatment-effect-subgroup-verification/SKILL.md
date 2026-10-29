---
name: treatment-effect-subgroup-verification
description: Use when a randomized or observational dataset has a treatment assignment
  variable, a response outcome, and covariates, and the goal is to identify subgroups
  with differential treatment effects (precision medicine setting).
when_to_use_negative:
- The goal is single-population treatment effect estimation without subgroup identification
  — DIPM is specifically a classification tree for identifying subgroups relevant
  to the precision medicine setting
- Predictors are continuous-only with no treatment assignment variable, so no treatment-contrast
  comparisons within subgroups can be formed
- A survival-type outcome is the target — use the package's spmtree function rather
  than the tree-based dipm workflow for subgroup search
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: dipm
  role: Primary function for fitting the DIPM classification tree with formula, types,
    nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune
    parameters
  repo: https://github.com/chenvict/dipm
- name: dipm (R package)
  role: R package implementing the DIPM tree and forest method via R code that calls
    a program in C
  repo: https://cran.r-project.org/package=dipm
- name: partykit
  role: Imported dependency of the dipm package used to represent/inspect the fitted
    tree structure
- name: stats
  role: Imported dependency of the dipm package used for underlying statistical computations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:241cb7c7936fdbb539f5907bb27bba10ae8524356ecf1c6763a7535bb6f587ae
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# treatment-effect-subgroup-verification

## Summary

Fits a DIPM classification tree with the dipm R package to identify subgroups where subjects perform especially well or poorly under a particular treatment assignment, then verifies which treatment is optimal in each terminal-node subgroup.

## When to use

Use when a randomized or observational dataset has a treatment assignment variable, a response outcome, and covariates, and the goal is to identify subgroups with differential treatment effects (precision medicine setting).

## When NOT to use

- The goal is single-population treatment effect estimation without subgroup identification — DIPM is specifically a classification tree for identifying subgroups relevant to the precision medicine setting
- Predictors are continuous-only with no treatment assignment variable, so no treatment-contrast comparisons within subgroups can be formed
- A survival-type outcome is the target — use the package's spmtree function rather than the tree-based dipm workflow for subgroup search

## Inputs

- data.frame with a response outcome, a treatment assignment variable, and covariates (e.g. MASS::anorexia: treatment group, pre-weight, post-weight)
- model formula specifying response and predictors
- types vector declaring predictor variable types (e.g. 'c' for categorical, 'n' for numeric)

## Outputs

- Fitted DIPM tree object with splitting variables and terminal-node subgroups (serializable via saveRDS, e.g. anorexia_dipm_tree.rds)
- Per-subgroup optimal treatment assignments and cross-subgroup treatment-contrast comparisons
- partykit-convertable tree structure for visualization

## How to apply

Load the dataset (e.g. MASS::anorexia, 72 observations with treatment group, pre-weight, and post-weight variables) and fit a DIPM tree with dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune), passing the response (e.g. PostWeight) and treatment group and pre-weight as predictors via types. Set maxdepth/maxdepth2 and nmin/nmin2 to control tree depth and minimum node sizes; ntree and mtry matter for the forest variant. Inspect the fitted tree object (or convert it via partykit for visualization) to extract splitting variables and terminal-node subgroups. Verify the structure by comparing treatment contrasts within each subgroup — e.g. in the anorexia analysis, family treatment should be optimal in both identified subgroups and the CBT-vs-control contrast should be more profound in the higher pre-weight subgroup. Save the fitted tree with saveRDS for reproducibility.

## Related tools

- **dipm** (Primary function for fitting the DIPM classification tree with formula, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune parameters) — https://github.com/chenvict/dipm
- **dipm (R package)** (R package implementing the DIPM tree and forest method via R code that calls a program in C) — https://cran.r-project.org/package=dipm
- **partykit** (Imported dependency of the dipm package used to represent/inspect the fitted tree structure)
- **stats** (Imported dependency of the dipm package used for underlying statistical computations)

## Examples

```
tree <- dipm(PostWeight ~ Treat + PreWeight, data = anorexia, types = c('c','n')); saveRDS(tree, 'anorexia_dipm_tree.rds')
```

## Evaluation signals

- Tree object contains the expected splitting variables (e.g. treatment group and pre-weight) and the expected number of terminal-node subgroups
- Treatment contrasts computed within each terminal node match reported findings (family treatment optimal in both anorexia subgroups; CBT-vs-control contrast more profound in the higher pre-weight subgroup)
- Node sizes respect the nmin/nmin2 minimum node size settings and depth respects maxdepth/maxdepth2
- Fitted tree can be converted/visualized through partykit without error
- saveRDS-serialized tree (e.g. anorexia_dipm_tree.rds) reloads and reproduces identical subgroup structure

## Limitations

- The package is listed under Dependencies that are NOT attached by loading it; call library(<dep>) explicitly before using dependency functions
- No software version is stated in the source documentation
- No changelog found in the source repository, so version-to-version behavioral changes are not traceable from the repo

## Evidence

- [abstract] DIPM method purpose: [evidence span withheld — anchor status: not_found]
- [abstract] dipm implementation via C: "we introduce, dipm, a novel R package that implements the DIPM method using R code that calls a program in C"
- [methods] dipm function signature: "dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune)"
- [methods] dependency loading caveat: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
- [abstract] repository location: "It is continuously being developed at https://github.com/chenvict/dipm"
- [intro] package scope: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
