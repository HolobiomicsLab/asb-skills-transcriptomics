---
name: anorexia-treatment-outcome-analysis
description: Use when a treatment-outcome dataset (e.g., anorexia patients with treatment
  group, pre-weight, and post-weight) needs subgroup identification via a DIPM tree
  with a categorical response and mixed covariate types.
when_to_use_negative:
- If the goal is subgroup detection for a continuous response or survival outcome
  — the dipm package also provides spmtree() for that setting, not the DIPM tree workflow
- If the dataset is not a treatment-comparison design with a treatment-assignment
  covariate (e.g., a feature table or omics data without treatment labels), the subgroup-with-optimal-treatment
  interpretation does not apply
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: dipm
  role: primary R function (backed by a C program) that fits the DIPM classification
    tree to the anorexia data
  repo: https://github.com/chenvict/dipm
- name: partykit
  role: dependency used for rendering/inspecting the fitted tree structure after loading
    explicitly
- name: stats
  role: R stats package dependency supporting formula handling and model statistics
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:6f0dacde04f00694a7e5362b8df9893648b9619258deb09b74c6d3f8ac591e23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chenvict/dipm
  tools:
  - R
  - dipm
  - partykit
  - stats
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# anorexia-treatment-outcome-analysis

## Summary

Reproduce the DIPM classification-tree analysis of the MASS::anorexia dataset with post-weight (Postwt) as the response, identifying treatment-assignment subgroups via the dipm R package and verifying that family treatment is optimal in both identified subgroups.

## When to use

Use when a treatment-outcome dataset (e.g., anorexia patients with treatment group, pre-weight, and post-weight) needs subgroup identification via a DIPM tree with a categorical response and mixed covariate types.

## When NOT to use

- If the goal is subgroup detection for a continuous response or survival outcome — the dipm package also provides spmtree() for that setting, not the DIPM tree workflow
- If the dataset is not a treatment-comparison design with a treatment-assignment covariate (e.g., a feature table or omics data without treatment labels), the subgroup-with-optimal-treatment interpretation does not apply

## Inputs

- MASS::anorexia data frame (72 rows: treatment group, pre-weight, post-weight)
- model formula with a categorical post-weight response and treatment/pre-weight predictors
- covariate type specification (e.g., 'types' argument distinguishing treatment vs numeric variables)

## Outputs

- fitted DIPM tree object with identified splitting variables and terminal-node subgroups
- anorexia_dipm_tree.rds file saved via saveRDS
- per-subgroup optimal-treatment assignments

## How to apply

Load MASS::anorexia (72 observations with treatment group, pre-weight, and post-weight variables) and fit a DIPM tree with dipm(Postwt ~ Treat + Prewt, data = MASS::anorexia), passing the response as post-weight and treatment group and pre-weight as predictors; the dipm() call accepts the signature dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune). Because packages listed under Dependencies are not attached when dipm is loaded, call library(<dep>) explicitly (e.g., partykit for tree visualization) before using their functions. Inspect the fitted tree object to extract the splitting variables and terminal-node subgroups, then save it with saveRDS to anorexia_dipm_tree.rds. The rationale is that DIPM is a classification tree designed for the precision-medicine setting, ranking covariate splits by depth importance to find subgroups where a particular treatment performs especially well or poorly.

## Related tools

- **dipm** (primary R function (backed by a C program) that fits the DIPM classification tree to the anorexia data) — https://github.com/chenvict/dipm
- **partykit** (dependency used for rendering/inspecting the fitted tree structure after loading explicitly)
- **stats** (R stats package dependency supporting formula handling and model statistics)

## Examples

```
library(dipm); tree <- dipm(Postwt ~ Treat + Prewt, data = MASS::anorexia); saveRDS(tree, "anorexia_dipm_tree.rds")
```

## Evaluation signals

- The fitted tree identifies subgroups in which family treatment is the optimal treatment in both terminal nodes
- The CBT-vs-control contrast is more profound in the higher pre-weight subgroup than in the lower pre-weight subgroup
- The tree object exposes identifiable splitting variables and terminal-node subgroups when inspected
- The fitted tree round-trips through saveRDS to anorexia_dipm_tree.rds and reloads correctly

## Limitations

- Packages listed under Dependencies are NOT attached by loading the dipm package (only Depends is); they must be attached explicitly with library(<dep>) before use
- No software version is stated in the article's usage documentation, and no changelog was found in the source repository
- The dipm() function has many tunables (nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune) whose values are not specified in the reproduced workflow, so defaults must be relied upon

## Evidence

- [abstract] DIPM identifies treatment-optimal subgroups: [evidence span withheld — anchor status: not_found]
- [methods] dipm function signature: "dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune)"
- [methods] dependency attachment behavior: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
- [readme] package description from README: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
- [abstract] implementation mechanism: "we introduce, dipm, a novel R package that implements the DIPM method using R code that calls a program in C"
