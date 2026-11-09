---
name: treatment-effect-split-structure-verification
description: Use when a fitted DIPM tree (e.g. on GBSG2 with Surv(tyears, status))
  must be checked against a published subgroup finding, confirming the split variables
  and thresholds match the reported treatment-effect pattern.
when_to_use_negative:
- When the goal is forest-based variable importance or subgroup detection itself,
  rather than verifying an already-published tree split structure
- When the input is not a treatment-assignment survival dataset (no Surv outcome with
  treatment as a predictor), since the DIPM classification tree targets treatment-effect
  subgroups
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: dipm
  role: R package implementing the DIPM tree; provides dipm(formula, data, types,
    nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune)
    for fitting and pmprune() for pruning
  repo: https://github.com/chenvict/dipm
- name: survival
  role: Provides Surv(tyears, status) for the survival outcome formula; must be attached
    explicitly since only Depends is attached by loading dipm
- name: partykit
  role: Imported by dipm; used to convert/visualize the fitted tree for split inspection
- name: R
  role: Runtime environment in which the dipm package (R code calling a C program)
    is executed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:728eacfd1199eecfd926803a486dec5f42e6ff197ab0992a1b42044808b41437
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# treatment-effect-split-structure-verification

## Summary

Verifies that a fitted DIPM survival tree's split structure reproduces the reported treatment-effect subgroups, using the dipm R package's dipm() call and pmprune()/partykit inspection, as demonstrated on the GBSG2 breast cancer data.

## When to use

Use when a fitted DIPM tree (e.g. on GBSG2 with Surv(tyears, status)) must be checked against a published subgroup finding, confirming the split variables and thresholds match the reported treatment-effect pattern.

## When NOT to use

- When the goal is forest-based variable importance or subgroup detection itself, rather than verifying an already-published tree split structure
- When the input is not a treatment-assignment survival dataset (no Surv outcome with treatment as a predictor), since the DIPM classification tree targets treatment-effect subgroups

## Inputs

- GBSG2 dataset from the TH.data R package (German Breast Cancer Study data)
- Model formula with Surv(tyears, status) outcome and treatment + covariate predictors
- types vector declaring variable types for dipm(formula, data, types, ...)

## Outputs

- Fitted DIPM tree object (optionally pruned with pmprune(tree))
- Extracted sequence of tree splits (split variables and thresholds)
- Tree structure text file dipm_gbsg2_tree_structure.txt
- Optional partykit visualization of the tree

## How to apply

Attach the survival package explicitly (dipm lists it under Imports, and only Depends entries are attached when loading dipm). Fit the tree with dipm(formula, data, types, nmin, maxdepth, ...), using Surv(tyears, status) as the outcome and treatment plus covariates (progesterone receptor, tumor grade, tumor size) as predictors, specifying variable types and node-size/depth parameters. Optionally prune with pmprune(tree) or convert via partykit for visualization, then extract the sequence of splits from the fitted tree object. Write the split structure to dipm_gbsg2_tree_structure.txt and check that progesterone receptor, tumor grade, and tumor size appear as splitting variables and that the tree reproduces the reported subgrouping: hormonal therapy is most effective except for progesterone receptor < 74 fmol with tumor grade I, or progesterone receptor > 74 fmol with tumor size > 26 mm.

## Related tools

- **dipm** (R package implementing the DIPM tree; provides dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune) for fitting and pmprune() for pruning) — https://github.com/chenvict/dipm
- **survival** (Provides Surv(tyears, status) for the survival outcome formula; must be attached explicitly since only Depends is attached by loading dipm)
- **partykit** (Imported by dipm; used to convert/visualize the fitted tree for split inspection)
- **R** (Runtime environment in which the dipm package (R code calling a C program) is executed)

## Examples

```
Rscript -e 'library(dipm); library(survival); tree <- dipm(Surv(tyears, status) ~ treatment + progreceptor + tumorgade + tumorsize, data = GBSG2, types = c("t","n","n","n","n"), nmin = 50, maxdepth = 2); pmprune(tree)'
```

## Evaluation signals

- dipm_gbsg2_tree_structure.txt exists and is non-empty
- Progesterone receptor, tumor grade, and tumor size all appear as splitting variables in the output tree structure
- The tree's split structure matches the reported subgroup pattern (progesterone receptor < 74 fmol with tumor grade I, or progesterone receptor > 74 fmol with tumor size > 26 mm, where hormonal therapy is not most effective)
- pmprune(tree) returns a valid pruned tree and the pruned structure retains the reported splits
- survival is confirmed attached (library(survival) called explicitly) before formula construction with Surv(tyears, status)

## Limitations

- Packages listed under Dependencies are NOT attached by loading dipm (only Depends is); failing to explicitly call library(survival) causes the Surv() formula to fail
- No software version stated in the methods documentation and no changelog found in the source repository, so reproducibility against a pinned dipm release is not documented
- Verification is limited to the tree; forest-level DIPM results require the ntree/mtry forest parameters and are not covered by single-tree split-structure checks

## Evidence

- [abstract] DIPM is a classification tree for precision-medicine subgroups: "The Depth Importance in Precision Medicine (DIPM) method is a classification tree designed for the identification of subgroups relevant to the precision medicine setting."
- [abstract] dipm implemented via R calling C: "we introduce, dipm, a novel R package that implements the DIPM method using R code that calls a program in C"
- [methods] dipm() signature with types, nmin, maxdepth: "- `dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune)`"
- [methods] Dependencies not attached automatically: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
- [readme] README description of the package: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
