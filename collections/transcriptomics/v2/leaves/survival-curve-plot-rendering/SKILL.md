---
name: survival-curve-plot-rendering
description: Use when a fitted dipm or spmtree tree (e.g., anorexia continuous-outcome
  or GBSG2 survival-outcome trees) needs per-node treatment-group plots rendered via
  partykit::plot.
when_to_use_negative:
- The tree object is not from the dipm or spmtree fitting functions, since node_dipm
  expects partykit-compatible terminal-node panels for DIPM stratified treatment groups
- The goal is to fit or prune the tree (use dipm, spmtree, or pmprune) rather than
  visualize the already-fitted tree structure
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: dipm (R package)
  role: Provides the node_dipm(obj, ...) panel function that draws per-node visualizations
    of stratified treatment groups
  repo: https://github.com/chenvict/dipm
- name: partykit
  role: Provides the plot method that node_dipm must be compatible with as a terminal-node
    panel
- name: ggplot2
  role: Used to render treatment-group distributions as boxplots for continuous-outcome
    nodes
- name: survival
  role: Provides survfit objects used to render treatment-group survival curves for
    survival-outcome nodes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:37d0e4b0de5510f100ef2d34c9808a3085575bc6486fb67610cdd38ba0275adb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chenvict/dipm
  tools:
  - R
  - dipm (R package)
  - ggplot2
  - grid
  - partykit
  - stats
  - survival
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# survival-curve-plot-rendering

## Summary

Renders per-node subgroup visualizations for fitted DIPM trees by supplying the dipm package's node_dipm panel function to partykit's plot method, drawing treatment-group boxplots for continuous outcomes and survival curves for survival outcomes.

## When to use

Use when a fitted dipm or spmtree tree (e.g., anorexia continuous-outcome or GBSG2 survival-outcome trees) needs per-node treatment-group plots rendered via partykit::plot.

## When NOT to use

- The tree object is not from the dipm or spmtree fitting functions, since node_dipm expects partykit-compatible terminal-node panels for DIPM stratified treatment groups
- The goal is to fit or prune the tree (use dipm, spmtree, or pmprune) rather than visualize the already-fitted tree structure

## Inputs

- Fitted dipm tree object (e.g., for the anorexia dataset with a continuous outcome)
- Fitted spmtree tree object (e.g., for the GBSG2 dataset with a survival outcome)
- node_dipm panel function from the dipm package's exported API

## Outputs

- Tree plot PNG with per-node treatment-group boxplots (continuous outcome)
- Tree plot PNG with per-node treatment-group survival curves (survival outcome)

## How to apply

Load the fitted dipm/spmtree tree objects for a continuous-outcome dataset (anorexia) and/or a survival-outcome dataset (GBSG2) in R. Implement the node_dipm(obj, ...) panel function from the dipm package's exported API, which is designed to be compatible with the plot method defined in the partykit package. For continuous-outcome terminal nodes, node_dipm renders treatment-group distributions as boxplots using ggplot2; for survival-outcome nodes, it renders treatment-group survival curves using survfit objects from the survival package. Call partykit::plot on each tree with node_dipm as the terminal-node panel and save the rendered tree plots as PNG files.

## Related tools

- **dipm (R package)** (Provides the node_dipm(obj, ...) panel function that draws per-node visualizations of stratified treatment groups) — https://github.com/chenvict/dipm
- **partykit** (Provides the plot method that node_dipm must be compatible with as a terminal-node panel)
- **ggplot2** (Used to render treatment-group distributions as boxplots for continuous-outcome nodes)
- **survival** (Provides survfit objects used to render treatment-group survival curves for survival-outcome nodes)

## Examples

```
library(dipm); library(partykit); partykit::plot(anorexia_tree, terminal_panel = node_dipm)
```

## Evaluation signals

- The rendered PNG shows the tree's terminal nodes each populated with a treatment-group boxplot (continuous outcome) or survival curve (survival outcome), not partykit's default panel
- The partykit::plot call completes without error when node_dipm is passed as the terminal-node panel, confirming API compatibility with partykit's plot method
- For the GBSG2 survival tree, each node panel displays survival curves computed from survfit objects (survival package), distinguishable per treatment group
- For the anorexia continuous tree, each node panel displays ggplot2 boxplots stratified by treatment group
- Both rendered tree plots are saved as PNG files

## Limitations

- Packages listed under Imports in the dipm package are NOT attached by loading the package; call library(<dep>) explicitly before using functions from dependencies such as partykit, ggplot2, or survival
- No software version stated and no changelog found in the source repository, so panel-function behavior may vary across versions

## Evidence

- [other] node_dipm draws boxplots for continuous outcomes and survival plots for survival outcomes: [evidence span withheld — anchor status: not_found]
- [methods] partykit is imported by the dipm package: "`partykit (Imports)`"
- [readme] README confirms the package scope: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
- [methods] Dependency attachment behavior caveat: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
