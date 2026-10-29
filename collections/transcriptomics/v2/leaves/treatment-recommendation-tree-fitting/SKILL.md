---
name: treatment-recommendation-tree-fitting
description: Use when a treatment-subgroup analysis needs a fitted classification
  tree (e.g., on weight-change data like MASS::anorexia) whose leaves give optimal
  treatment assignments per subgroup.
when_to_use_negative:
- The analysis goal is survival-model subgroup trees rather than treatment-subgroup
  classification — the package's separate spmtree(formula, data, types, nmin, maxdepth,
  print, dataframe, prune) targets that setting
- The tree must retain full depth/structure for inference, since pmprune collapses
  terminal sister nodes that agree on optimal treatment
- Input is not a treatment-comparison dataset with a subgroup-identification goal;
  DIPM trees are designed for identification of subgroups relevant to the precision
  medicine setting
tools:
- name: dipm
  role: R package whose dipm() function grows the DIPM classification tree and whose
    pmprune() prunes terminal sister nodes with the same optimal treatment
  repo: https://github.com/chenvict/dipm
- name: partykit
  role: Dependency used to inspect/plot the fitted and pruned tree objects
  repo: https://cran.r-project.org/package=dipm
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:e885760a57496febb7564295643e73f511ba53288e63e5fa8b4f8fabe0b061dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chenvict/dipm
  tools:
  - R
  - dipm
  - partykit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# treatment-recommendation-tree-fitting

## Summary

Fits a Depth Importance in Precision Medicine (DIPM) classification tree with the dipm R package (R code calling a C program) to identify subgroups where subjects perform especially well or poorly under a particular treatment, then simplifies it via pmprune without changing treatment recommendations.

## When to use

Use when a treatment-subgroup analysis needs a fitted classification tree (e.g., on weight-change data like MASS::anorexia) whose leaves give optimal treatment assignments per subgroup.

## When NOT to use

- The analysis goal is survival-model subgroup trees rather than treatment-subgroup classification — the package's separate spmtree(formula, data, types, nmin, maxdepth, print, dataframe, prune) targets that setting
- The tree must retain full depth/structure for inference, since pmprune collapses terminal sister nodes that agree on optimal treatment
- Input is not a treatment-comparison dataset with a subgroup-identification goal; DIPM trees are designed for identification of subgroups relevant to the precision medicine setting

## Inputs

- Formula plus data frame (e.g., formula applied to MASS::anorexia weight-change data) with a treatment-assignment outcome and subject covariates
- dipm() parameters: types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune
- A fitted DIPM tree object (input to pmprune)

## Outputs

- A fitted DIPM classification tree object with optimal treatment assignment at each node
- A pruned tree (via pmprune) with terminal sister nodes sharing the same optimal treatment collapsed
- Serialized tree object (pruned_tree.rds) and optional partykit plot

## How to apply

Load the dipm package and explicitly attach its Dependencies (stats, utils, survival, partykit, ggplot2, rlang, grid) with library() calls, since only Depends packages are attached automatically. Call dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune) with default splitting parameters on the treatment-subgroup data (e.g., the MASS::anorexia weight-change dataset) to grow the DIPM classification tree. Then call pmprune(tree) to remove terminal sister nodes whose optimal treatments coincide, collapsing subtrees whose child leaves agree on the best treatment assignment — this simplifies the tree without altering any treatment recommendation. Serialize the pruned tree to pruned_tree.rds and optionally plot it with partykit for inspection.

## Related tools

- **dipm** (R package whose dipm() function grows the DIPM classification tree and whose pmprune() prunes terminal sister nodes with the same optimal treatment) — https://github.com/chenvict/dipm
- **partykit** (Dependency used to inspect/plot the fitted and pruned tree objects) — https://cran.r-project.org/package=dipm

## Examples

```
library(dipm); tree <- dipm(formula, data = anorexia); pruned_tree <- pmprune(tree); saveRDS(pruned_tree, "pruned_tree.rds")
```

## Evaluation signals

- pruned_tree.rds exists and can be loaded back into R
- Pruned tree retains the same optimal treatment assignments as the unpruned tree at every retained node
- pmprune only removed terminal sister nodes whose optimal treatments coincide — no node with divergent child treatments was collapsed
- All dependency packages (stats, utils, survival, partykit, ggplot2, rlang, grid) attach without error, confirming the Imports-only setup is handled

## Limitations

- Packages listed under Imports are NOT attached when the package is loaded (only Depends is); each must be attached with library() explicitly
- No software version stated in the article's usage/docs; no changelog found in the source repository
- The README alone gives only a one-line package description; concrete parameters (nmin, nmin2, mtry, maxdepth, maxdepth2) must be set or left at defaults by the practitioner

## Evidence

- [abstract] DIPM purpose (abstract): "The Depth Importance in Precision Medicine (DIPM) method is a classification tree designed for the identification of subgroups relevant to the precision medicine setting."
- [abstract] Implementation mechanism (abstract): "we introduce, dipm, a novel R package that implements the DIPM method using R code that calls a program in C"
- [methods] dipm() signature (methods): "dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune)"
- [methods] Dependency attachment rule (methods): "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
- [readme] README description (readme): "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
