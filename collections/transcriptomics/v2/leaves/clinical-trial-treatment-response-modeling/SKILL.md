---
name: clinical-trial-treatment-response-modeling
description: Use when modeling treatment response in a clinical or trial-like dataset
  where the goal is subgroup identification (e.
when_to_use_negative:
- When you want the DIPM forest-based method rather than a single classical tree —
  use dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores,
  print, dataframe, prune), which fits a random forest at each node
- When the analysis goal is not subgroup/treatment-response identification (the DIPM
  method targets subgroups where subjects perform especially well or poorly with a
  particular treatment assignment)
- When only the tree-fitting is needed but no pruning is desired — pmprune(tree) is
  a separate post-pruning step, not part of split selection
tools:
- name: dipm
  role: R package providing spmtree(formula, data, types, nmin, maxdepth, print, dataframe,
    prune) for exhaustive split-search tree fitting and pmprune(tree) for post-pruning
  repo: github.com/chenvict/dipm
- name: partykit
  role: Dependency of the dipm package used for tree structure representation/inspection
    of the fitted spmtree object
- name: R
  role: Runtime environment in which the dipm package functions are called (the package
    implements the DIPM method using R code that calls a program in C)
  repo: cran.r-project.org/package=dipm
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:265c0d5677c97642d9230c05d396c75b92a6bf8f7f79713d7aaaf4eb5baffd14
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - dipm
  - partykit
  - pmprune
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# clinical-trial-treatment-response-modeling

## Summary

Fits a classical (exhaustive split-search) precision-medicine classification tree with the spmtree function of the dipm R package, identifying subgroups of subjects who respond especially well or poorly to a treatment assignment, in contrast to the forest-based DIPM node fitting.

## When to use

Use when modeling treatment response in a clinical or trial-like dataset where the goal is subgroup identification (e.g. anorexia patient weight change by treatment and baseline covariates), and a single interpretable tree built by exhaustively searching all splits is preferred over a random-forest-based approach.

## When NOT to use

- When you want the DIPM forest-based method rather than a single classical tree — use dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune), which fits a random forest at each node
- When the analysis goal is not subgroup/treatment-response identification (the DIPM method targets subgroups where subjects perform especially well or poorly with a particular treatment assignment)
- When only the tree-fitting is needed but no pruning is desired — pmprune(tree) is a separate post-pruning step, not part of split selection

## Inputs

- Clinical-trial-style outcome data in an R data.frame (e.g. anorexia patient weight change)
- A model formula relating the outcome to treatment assignment and baseline covariates
- A types argument encoding treatment/coding/interacting variable roles

## Outputs

- A fitted spmtree tree object whose each internal node is the exhaustively searched split with the highest split-criteria score
- Optionally, a post-pruned tree produced by pmprune(tree)
- A serialized tree file such as spmtree_anorexia.rds

## How to apply

Load the outcome data (e.g. anorexia patient weight change) into R and build a formula relating the outcome to treatment assignment and baseline covariates, passing a types argument that encodes treatment/coding/interacting variable roles. Call spmtree(formula, data, types, nmin, maxdepth, print, dataframe, prune) from the dipm package: at each node it does NOT fit a random forest (as dipm() does), but instead considers all possible splits of all candidate split variables and selects the single split with the highest split-criteria score. Constrain tree growth with nmin (minimum node size) and maxdepth (maximum depth), the only structural tuning arguments spmtree exposes. Optionally set prune to apply the pmprune(tree) post-pruning routine to simplify the fitted tree. Print/inspect the resulting tree structure (with partykit) and save the fitted object, e.g. as spmtree_anorexia.rds.

## Related tools

- **dipm** (R package providing spmtree(formula, data, types, nmin, maxdepth, print, dataframe, prune) for exhaustive split-search tree fitting and pmprune(tree) for post-pruning) — github.com/chenvict/dipm
- **partykit** (Dependency of the dipm package used for tree structure representation/inspection of the fitted spmtree object)
- **R** (Runtime environment in which the dipm package functions are called (the package implements the DIPM method using R code that calls a program in C)) — cran.r-project.org/package=dipm

## Examples

```
tree <- spmtree(formula, data, types, nmin, maxdepth, print, dataframe, prune); pruned <- pmprune(tree); print(pruned); saveRDS(pruned, "spmtree_anorexia.rds")
```

## Evaluation signals

- The saved spmtree object (e.g. spmtree_anorexia.rds) exists and can be reloaded and printed
- Each internal node corresponds to the split found by exhaustively searching all possible splits of all candidate variables, i.e. the split with the highest split-criteria score
- Tree growth respects the structural constraints: node sizes meet nmin and depth does not exceed maxdepth
- If prune was set, the pmprune(tree) routine was applied and the simplified tree remains interpretable via print

## Limitations

- spmtree exposes only nmin (minimum node size) and maxdepth (maximum depth) as structural tuning arguments — no ntree/mtry/ncores forest parameters, unlike dipm()
- No software version, changelog, or dataset accession is stated in the article or repository documentation
- The dipm package's Dependencies are not attached by loading the package (only Depends is); dependent packages such as partykit must be attached explicitly with library() before use

## Evidence

- [abstract] DIPM method targets precision-medicine subgroups: "The Depth Importance in Precision Medicine (DIPM) method is a classification tree designed for the identification of subgroups relevant to the precision medicine setting."
- [methods] spmtree function signature: "spmtree(formula, data, types, nmin, maxdepth, print, dataframe, prune)"
- [intro] Package scope: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
- [abstract] Implementation approach: "we introduce, dipm, a novel R package that implements the DIPM method using R code that calls a program in C"
