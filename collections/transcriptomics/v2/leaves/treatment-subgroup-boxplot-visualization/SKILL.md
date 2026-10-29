---
name: treatment-subgroup-boxplot-visualization
description: Use when a fitted subgroup tree (e.g. dipm on anorexia with a continuous
  outcome, or spmtree on GBSG2 with a survival outcome) needs terminal-node plots
  showing how treatment groups differ within each node.
when_to_use_negative:
- The goal is fitting or pruning a subgroup tree rather than visualizing an already-fitted
  one — use the dipm(), spmtree(), or pmprune() functions instead.
- The tree is not a partykit-compatible object, since node_dipm is designed as a panel
  function for partykit's plot method.
- The outcome is neither continuous nor survival, as node_dipm handles only boxplots
  for continuous outcomes and survival plots for survival outcomes.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: dipm
  role: Provides the node_dipm(obj, ...) terminal-node panel function that visualizes
    stratified treatment groups
  repo: https://github.com/chenvict/dipm
- name: partykit
  role: Plot framework whose plot method accepts node_dipm as the terminal-node panel
- name: ggplot2
  role: Renders treatment-group boxplots for continuous-outcome nodes
- name: survival
  role: Supplies survfit objects used to draw treatment-group survival curves for
    survival-outcome nodes
- name: R
  role: Runtime environment in which the dipm package and plotting pipeline execute
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:214d4464d8fdbaffee9fb159dbe0049f8dfae9aef963d62a2fc2f51f3ef8626c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chenvict/dipm
  tools:
  - R
  - dipm
  - ggplot2
  - grid
  - partykit
  - stats
  - survival
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# treatment-subgroup-boxplot-visualization

## Summary

Visualizes stratified treatment-effect subgroups from fitted DIPM/spmtree trees by attaching the dipm package's node_dipm panel function to partykit's plot method, rendering per-node treatment-group boxplots (continuous outcomes) or survival curves (survival outcomes).

## When to use

Use when a fitted subgroup tree (e.g. dipm on anorexia with a continuous outcome, or spmtree on GBSG2 with a survival outcome) needs terminal-node plots showing how treatment groups differ within each node.

## When NOT to use

- The goal is fitting or pruning a subgroup tree rather than visualizing an already-fitted one — use the dipm(), spmtree(), or pmprune() functions instead.
- The tree is not a partykit-compatible object, since node_dipm is designed as a panel function for partykit's plot method.
- The outcome is neither continuous nor survival, as node_dipm handles only boxplots for continuous outcomes and survival plots for survival outcomes.

## Inputs

- Fitted dipm tree object (e.g. trained on the anorexia dataset with a continuous outcome)
- Fitted spmtree tree object (e.g. trained on the GBSG2 dataset with a survival outcome)
- Terminal-node panel function node_dipm(obj, ...) from the dipm package

## Outputs

- Partykit tree plots with per-node treatment-group boxplots (continuous outcomes)
- Partykit tree plots with per-node treatment-group survival curves (survival outcomes)
- PNG files of the rendered tree plots for the anorexia and GBSG2 trees

## How to apply

Load the fitted dipm or spmtree tree objects — e.g. the anorexia tree (continuous outcome) and the GBSG2 tree (survival outcome) — in R. Implement the exported node_dipm(obj, ...) panel function from the dipm package's API so it is compatible with partykit's plot method for terminal nodes. For continuous-outcome nodes, draw treatment-group distributions as boxplots using ggplot2; for survival-outcome nodes, render treatment-group survival curves from survival-package survfit objects within the partykit plotting framework. Then call partykit::plot on each tree with node_dipm as the terminal-node panel and save each rendered tree plot as a PNG file. The rationale is that the DIPM method identifies subgroups where subjects perform especially well or poorly under a particular treatment, so per-node visualization makes the treatment contrast in each identified subgroup directly inspectable.

## Related tools

- **dipm** (Provides the node_dipm(obj, ...) terminal-node panel function that visualizes stratified treatment groups) — https://github.com/chenvict/dipm
- **partykit** (Plot framework whose plot method accepts node_dipm as the terminal-node panel)
- **ggplot2** (Renders treatment-group boxplots for continuous-outcome nodes)
- **survival** (Supplies survfit objects used to draw treatment-group survival curves for survival-outcome nodes)
- **R** (Runtime environment in which the dipm package and plotting pipeline execute)

## Examples

```
library(dipm); library(partykit); plot(tree, terminal_panel = node_dipm); png filename via plot(tree, terminal_panel = node_dipm) saved with png("anorexia_tree.png")
```

## Evaluation signals

- partykit::plot on the fitted tree completes without error when node_dipm is passed as the terminal-node panel function.
- Continuous-outcome tree plot shows one treatment-group boxplot per terminal node for the anorexia tree.
- Survival-outcome tree plot shows treatment-group survival curves per terminal node for the GBSG2 tree.
- Both rendered tree plots are saved as PNG files that open as valid images.
- The dipm package is attached/available (a package under Imports is not attached by loading dipm, so dependencies must be called explicitly before using their functions).

## Limitations

- Packages listed under Dependencies are NOT attached by loading dipm; only Depends is attached, so partykit, ggplot2, and survival must be loaded explicitly via library() calls.
- node_dipm supports only two outcome types: boxplots for continuous outcomes and survival plots for survival outcomes.
- The article README is minimal and states no software version or changelog, so version-specific behavior of node_dipm must be verified against the installed package.

## Evidence

- [methods] node_dipm visualizes stratified treatment groups: [evidence span withheld — anchor status: not_found]
- [methods] node_dipm signature: "- `node_dipm(obj, ...)`"
- [methods] dependencies not attached: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
- [methods] partykit as dependency: "- `partykit (Imports)`"
- [abstract] DIPM purpose: [evidence span withheld — anchor status: not_found]
- [readme] dipm repo: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
