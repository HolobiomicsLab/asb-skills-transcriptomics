---
name: survival-tree-fitting
description: Use when analyzing censored survival outcomes (Surv time-to-event data)
  from a clinical study with a treatment arm, to find covariate-defined subgroups
  where one treatment underperforms.
when_to_use_negative:
- The outcome is a binary or continuous response without censoring — use the classification/regression
  variant (spmtree) rather than the survival tree interface
- The goal is forest-based importance over many trees (ntree, mtry, maxdepth2 parameters)
  rather than a single interpretable subgroup tree
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: dipm
  role: Core package implementing the DIPM tree/forest method; dipm() fits the survival
    tree, pmprune() prunes it
  repo: https://github.com/chenvict/dipm
- name: survival
  role: Provides Surv() to construct the censored time-to-event outcome; must be attached
    explicitly since only Depends is loaded with dipm
- name: partykit
  role: Imports dependency used to convert/visualize the fitted tree structure
- name: stats
  role: Imports dependency used by the package's model-fitting code
- name: utils
  role: Imports dependency used by the package (e.g. for writing outputs)
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:381eb931ab0c0120795ec0badf943a387d6b079831118d6427bf782085ddcf95
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chenvict/dipm
  tools:
  - R
  - dipm
  - partykit
  - stats
  - survival
  - utils
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# survival-tree-fitting

## Summary

Fits Depth Importance in Precision Medicine (DIPM) survival trees with the dipm R package (R code calling a C program) to identify treatment-ineffective subgroups in censored clinical data, e.g. GBSG2 breast cancer patients where hormonal therapy is not most effective.

## When to use

Use when analyzing censored survival outcomes (Surv time-to-event data) from a clinical study with a treatment arm, to find covariate-defined subgroups where one treatment underperforms.

## When NOT to use

- The outcome is a binary or continuous response without censoring — use the classification/regression variant (spmtree) rather than the survival tree interface
- The goal is forest-based importance over many trees (ntree, mtry, maxdepth2 parameters) rather than a single interpretable subgroup tree

## Inputs

- R data frame of censored survival data (e.g. GBSG2 from TH.data) with a Surv(tyears, status) outcome, a treatment indicator, and covariates such as progesterone receptor, tumor grade, tumor size
- Character vector of variable types passed to dipm(..., types=)
- Model formula with Surv() outcome, e.g. Surv(tyears, status) ~ horTh + prog + tgrade + tsize

## Outputs

- Fitted DIPM tree object with a split structure naming the splitting variables (progesterone receptor, tumor grade, tumor size)
- Pruned tree via pmprune(tree), optionally converted to a partykit object for visualization
- Tree-structure text file, e.g. dipm_gbsg2_tree_structure.txt, listing split variables and subgroup paths

## How to apply

Load the survival and dipm packages explicitly — survival is only under Imports, so attaching dipm does not attach it. Build a formula with Surv(tyears, status) as the outcome and treatment plus covariates (e.g. progesterone receptor, tumor grade, tumor size in GBSG2) as predictors, matching the dipm(formula, data, types, ...) interface. Call dipm(), specifying variable types and tree-shape parameters nmin, nmin2, maxdepth (set ntree/mtry only for the forest variant), and inspect the fitted tree; optionally prune with pmprune(tree) or convert via partykit for visualization. Extract the split sequence and verify the progesterone-receptor/tumor-grade/tumor-size subgroup structure against the expected pattern, writing the split variables to a structure file.

## Related tools

- **dipm** (Core package implementing the DIPM tree/forest method; dipm() fits the survival tree, pmprune() prunes it) — https://github.com/chenvict/dipm
- **survival** (Provides Surv() to construct the censored time-to-event outcome; must be attached explicitly since only Depends is loaded with dipm)
- **partykit** (Imports dependency used to convert/visualize the fitted tree structure)
- **stats** (Imports dependency used by the package's model-fitting code)
- **utils** (Imports dependency used by the package (e.g. for writing outputs))

## Examples

```
library(survival); library(dipm); tree <- dipm(Surv(tyears, status) ~ horTh + prog + tgrade + tsize, data = GBSG2, types = types, nmin = nmin, maxdepth = maxdepth); pmprune(tree)
```

## Evaluation signals

- The output file (e.g. dipm_gbsg2_tree_structure.txt) exists and names progesterone receptor, tumor grade, and tumor size as splitting variables
- The tree's subgroup structure matches the reported pattern: hormonal therapy least effective for progesterone receptor < 74 fmol with tumor grade I, or progesterone receptor > 74 fmol with tumor size > 26 mm
- Trees contain no node smaller than the nmin/nmin2 settings and do not exceed maxdepth
- The survival package is explicitly attached before calling dipm() (loading dipm attaches only Depends, not Imports)

## Limitations

- No software version is stated in the article's methods
- Only Depends packages are attached by loading dipm; survival, partykit, ggplot2, rlang, stats, utils must be attached explicitly
- Single-tree results can be unstable; the package also offers a forest variant (ntree, mtry, maxdepth2) for that setting, which this skill's tree-fitting flow does not cover

## Evidence

- [abstract] The Depth Importance in Precision Medicine (DIPM) method is a classification tree for identifying subgroups in which subjects perform especially well or poorly with a particular treatment assignment.: [evidence span withheld — anchor status: not_found]
- [methods] dipm signature: "- `dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune)`"
- [methods] packages under Imports not attached: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
- [intro] R package implementing DIPM: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
