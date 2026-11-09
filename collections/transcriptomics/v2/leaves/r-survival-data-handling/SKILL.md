---
name: r-survival-data-handling
description: Use when analyzing right-censored time-to-event data in R (e.g. GBSG2
  with Surv(tyears, status)) and fitting a DIPM survival tree, since survival and
  other Imports must be attached explicitly after loading dipm.
when_to_use_negative:
- The outcome is uncensored classification data with no time-to-event component —
  use the standard DIPM classification tree setup rather than the survival-tree path
- You only need the dipm forest (ntree, mtry) rather than a single interpretable tree
  with its split sequence
tools:
- name: dipm
  role: R package that fits the DIPM survival tree via dipm(formula, data, types,
    ...) and prunes it via pmprune(tree)
  repo: https://github.com/chenvict/dipm
- name: survival
  role: Import of dipm providing Surv(tyears, status) for the censored-outcome formula;
    must be attached explicitly
- name: partykit
  role: Import of dipm used to convert the fitted tree for visualization
- name: stats
  role: Import of dipm supporting the model-fitting code path
- name: utils
  role: Import of dipm supporting I/O such as writing the tree structure file
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:6fab8a5c57a081f4a282640ea47839b0c19fd5312c0935cb0c11ff919033adbe
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# r-survival-data-handling

## Summary

Skill for handling right-censored survival data in R within the dipm package workflow: loading the GBSG2 breast cancer dataset, building a Surv(tyears, status) formula, and explicitly attaching dipm's Imports (survival, partykit) because only Depends is attached by loading the package.

## When to use

Use when analyzing right-censored time-to-event data in R (e.g. GBSG2 with Surv(tyears, status)) and fitting a DIPM survival tree, since survival and other Imports must be attached explicitly after loading dipm.

## When NOT to use

- The outcome is uncensored classification data with no time-to-event component — use the standard DIPM classification tree setup rather than the survival-tree path
- You only need the dipm forest (ntree, mtry) rather than a single interpretable tree with its split sequence

## Inputs

- GBSG2 German Breast Cancer Study data frame (loaded from TH.data) with time (tyears), event status, and covariates progesterone receptor, tumor grade, tumor size
- Model formula of the form Surv(tyears, status) ~ treatment + covariates
- Types vector specifying variable types for the dipm(formula, data, types, ...) interface

## Outputs

- Fitted DIPM survival tree object returned by dipm()
- Pruned tree object from pmprune(tree)
- partykit-convertible tree for visualization
- Tree structure text file (e.g. dipm_gbsg2_tree_structure.txt) listing split variables and subgroup pattern

## How to apply

Load the GBSG2 dataset from TH.data and attach the packages needed by dipm — since only Depends is attached when the dipm package loads, call library(survival) and library(partykit) explicitly. Construct a model formula with Surv(tyears, status) as the outcome and treatment plus covariates (progesterone receptor, tumor grade, tumor size) as predictors, matching the dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune) signature. Fit the DIPM survival tree via dipm(), setting variable types and node-size/depth parameters (nmin, maxdepth) per the function signature, then inspect or prune the tree object with pmprune(tree) and optionally convert it via partykit for visualization. Extract the sequence of splits to identify treatment-effect subgroups and write the tree structure to a text file (e.g. dipm_gbsg2_tree_structure.txt) for validation.

## Related tools

- **dipm** (R package that fits the DIPM survival tree via dipm(formula, data, types, ...) and prunes it via pmprune(tree)) — https://github.com/chenvict/dipm
- **survival** (Import of dipm providing Surv(tyears, status) for the censored-outcome formula; must be attached explicitly)
- **partykit** (Import of dipm used to convert the fitted tree for visualization)
- **stats** (Import of dipm supporting the model-fitting code path)
- **utils** (Import of dipm supporting I/O such as writing the tree structure file)

## Examples

```
library(dipm); library(survival); data(GBSG2, package='TH.data'); tree <- dipm(Surv(tyears, status) ~ horm + progrecep + tumgrade + tumsize, data=GBSG2, types, nmin, maxdepth)
```

## Evaluation signals

- The output file dipm_gbsg2_tree_structure.txt exists after the run
- The tree structure contains progesterone receptor, tumor grade, and tumor size as splitting variables
- The recovered split sequence matches the reported subgroup pattern (hormonal therapy most effective except progesterone receptor < 74 fmol with tumor grade I, or progesterone receptor > 74 fmol with tumor size > 26 mm)
- Attaching the environment confirms survival is attached — loading dipm alone does NOT attach Imports (only Depends)

## Limitations

- Packages listed under Imports of the dipm package are NOT attached by loading it (only Depends is), so library(survival) and library(partykit) must be called explicitly before use
- No software version stated in the article's methods
- No dataset names or accessions given in the article's methods section — the GBSG2 dataset source must be inferred from the task workflow

## Evidence

- [methods] DIPM survival tree fitting interface: "dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune)"
- [methods] Dependency attachment caveat: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
- [abstract] Method scope: "The Depth Importance in Precision Medicine (DIPM) method is a classification tree designed for the identification of subgroups relevant to the precision medicine setting."
- [readme] Package description: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
