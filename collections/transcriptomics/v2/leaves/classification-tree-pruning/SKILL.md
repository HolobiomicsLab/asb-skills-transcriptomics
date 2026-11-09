---
name: classification-tree-pruning
description: Use when a dipm() or spmtree() tree was fitted (possibly with default
  pruning disabled) and the resulting node structure needs simplification into a parsimonious
  subgroup tree before printing, plotting, or saving the fitted object.
when_to_use_negative:
- The tree has not yet been fitted — pmprune operates on a fitted dipm/spmtree object,
  not on raw data or a model formula
- The analysis goal is forest-based subgroup screening with dipm()'s ntree/mtry aggregation
  rather than a single interpretable pruned tree (e.g. spmtree's exhaustive per-node
  split search)
- Structural controls (nmin, maxdepth) already suffice and no branch simplification
  is needed
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: dipm
  role: R package providing dipm(), spmtree(), and pmprune() for fitting and post-pruning
    DIPM treatment-subgroup classification trees
  repo: https://github.com/chenvict/dipm
- name: partykit
  role: Dependency of the dipm package used for representing/inspecting the party-style
    tree object produced by spmtree()
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:6249714d94835b3de684e884fcbb3522c60d87944513f87fa2b060acb5e0c856
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chenvict/dipm
  tools:
  - R
  - dipm
  - partykit
  - pmprune
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# classification-tree-pruning

## Summary

Post-prune a fitted DIPM classification tree (from dipm() or spmtree() in the dipm R package) with the pmprune(tree) routine so that overgrown branches grown under nmin/maxdepth constraints are simplified before interpretation or subgroup reporting.

## When to use

Use when a dipm() or spmtree() tree was fitted (possibly with default pruning disabled) and the resulting node structure needs simplification into a parsimonious subgroup tree before printing, plotting, or saving the fitted object.

## When NOT to use

- The tree has not yet been fitted — pmprune operates on a fitted dipm/spmtree object, not on raw data or a model formula
- The analysis goal is forest-based subgroup screening with dipm()'s ntree/mtry aggregation rather than a single interpretable pruned tree (e.g. spmtree's exhaustive per-node split search)
- Structural controls (nmin, maxdepth) already suffice and no branch simplification is needed

## Inputs

- A fitted dipm tree or spmtree object (classification tree for treatment-subgroup identification)
- Model formula relating the outcome to treatment assignment and baseline covariates
- types argument encoding treatment/coding/interacting variable roles
- Structural tuning settings: nmin (minimum node size) and maxdepth (maximum depth)

## Outputs

- A pruned tree object with simplified node structure
- Printed tree structure for inspection
- Saved fitted object (e.g. spmtree_anorexia.rds)

## How to apply

Fit the tree first with dipm(formula, data, types, nmin, nmin2, ntree, mtry, maxdepth, maxdepth2, ncores, print, dataframe, prune) or the simpler spmtree(formula, data, types, nmin, maxdepth, print, dataframe, prune), passing the types argument to encode treatment/coding/interacting variable roles and constraining growth via nmin and maxdepth (spmtree exposes only these two structural tuning arguments). If prune is set, the package applies pmprune(tree) as the post-pruning step to simplify the fitted tree; otherwise call pmprune on the returned tree object explicitly. Pruning rationale: nodes grown to maxdepth may contain splits selected from an exhaustively searched candidate set (spmtree) or forest-based splits (dipm), and trimming weak branches yields a more stable, interpretable subgroup tree. After pruning, print/inspect the tree structure and save the fitted object (e.g. as an .rds file) for validation.

## Related tools

- **dipm** (R package providing dipm(), spmtree(), and pmprune() for fitting and post-pruning DIPM treatment-subgroup classification trees) — https://github.com/chenvict/dipm
- **partykit** (Dependency of the dipm package used for representing/inspecting the party-style tree object produced by spmtree())

## Examples

```
tree <- spmtree(formula, data = anorexia, types = types, nmin = nmin, maxdepth = maxdepth, prune = TRUE); tree <- pmprune(tree); saveRDS(tree, 'spmtree_anorexia.rds')
```

## Evaluation signals

- The pruned tree object can be reloaded (e.g. from spmtree_anorexia.rds) and printed without error
- Every retained internal node corresponds to the split with the highest split-criteria score (for spmtree, the exhaustive split search; for dipm, the forest-based node fitting)
- Pruned tree depth does not exceed the maxdepth constraint and node sizes respect the nmin minimum
- Comparing pre- and post-pruning structure shows removal of weak branches while the top split-criteria nodes are retained
- The types argument correctly encoded treatment/coding/interacting variable roles so pruning decisions respect covariate roles

## Limitations

- spmtree exposes only nmin and maxdepth as structural tuning arguments, limiting control over tree shape before pruning
- The dipm package lists packages under Dependencies as NOT attached by loading the package — partykit and other dependencies must be attached explicitly with library() before use
- No software version, changelog, or dataset accession details were documented in the source context

## Evidence

- [abstract] dipm implements the DIPM method: "we introduce, dipm, a novel R package that implements the DIPM method using R code that calls a program in C"
- [methods] spmtree signature with prune argument: "- `spmtree(formula, data, types, nmin, maxdepth, print, dataframe, prune)`"
- [full_text] spmtree split selection contrast: [evidence span withheld — anchor status: not_found]
- [readme] package scope: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
