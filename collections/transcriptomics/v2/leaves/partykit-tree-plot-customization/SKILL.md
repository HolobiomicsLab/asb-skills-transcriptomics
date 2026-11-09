---
name: partykit-tree-plot-customization
description: Use when a fitted subgroup tree (dipm/spmtree for continuous or survival
  outcomes) needs publication-ready visualization, and the default partykit terminal
  panels are insufficient because treatment-stratified distributions or survival curves
  must be shown per node.
when_to_use_negative:
- The tree is a plain rpart or party object from a non-dipm workflow — node_dipm is
  designed for dipm/spmtree objects returned by this package's fitting functions
- Only the DIPM depth importance scores or pruned subgroup structure are needed (use
  pmprune(tree) or the printed tree object instead of rendering plots)
- The analysis goal is fitting or pruning the tree rather than visualizing it — use
  dipm(), spmtree(), or pmprune() directly
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: dipm
  role: Supplies the node_dipm(obj, ...) terminal-node panel function that draws boxplots
    (continuous) or survival plots (survival) compatible with partykit's plot method
  repo: https://github.com/chenvict/dipm
- name: dipm (R package)
  role: R package implementing the DIPM tree and forest method; its exported API includes
    node_dipm for visualization
  repo: https://cran.r-project.org/package=dipm
- name: partykit
  role: Provides the plot method into which node_dipm is registered as the terminal-node
    panel
- name: ggplot2
  role: Used by node_dipm to render treatment-group boxplots for continuous-outcome
    nodes
- name: survival
  role: Provides survfit objects underlying the treatment-group survival curves rendered
    for survival-outcome nodes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:f637bf1ef6af9d2e529110820846e85dc957d0642213cb75e9733014eb4911c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/chenvict/dipm
  tools:
  - R
  - dipm
  - dipm (R package)
  - ggplot2
  - grid
  - partykit
  - stats
  - survival
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# partykit-tree-plot-customization

## Summary

This skill renders fitted precision-medicine trees (dipm or spmtree objects) as tree diagrams with custom terminal-node panels, using node_dipm from the dipm package as the panel function for partykit::plot. It produces per-node boxplots for continuous outcomes (e.g. anorexia) and survival curves for survival outcomes (e.g. GBSG2).

## When to use

Use when a fitted subgroup tree (dipm/spmtree for continuous or survival outcomes) needs publication-ready visualization, and the default partykit terminal panels are insufficient because treatment-stratified distributions or survival curves must be shown per node.

## When NOT to use

- The tree is a plain rpart or party object from a non-dipm workflow — node_dipm is designed for dipm/spmtree objects returned by this package's fitting functions
- Only the DIPM depth importance scores or pruned subgroup structure are needed (use pmprune(tree) or the printed tree object instead of rendering plots)
- The analysis goal is fitting or pruning the tree rather than visualizing it — use dipm(), spmtree(), or pmprune() directly

## Inputs

- Fitted dipm tree object (continuous-outcome subgroup tree, e.g. anorexia data)
- Fitted spmtree tree object (survival-outcome subgroup tree, e.g. GBSG2 data)
- node_dipm panel function exported by the dipm package

## Outputs

- PNG files of full tree diagrams with per-node treatment-stratified visualizations (boxplots for continuous outcomes, survival plots for survival outcomes)

## How to apply

Load the fitted dipm/spmtree tree objects — for example the anorexia (continuous outcome) and GBSG2 (survival outcome) trees built with dipm(formula, data, types, nmin, nmin2, ...) or spmtree(formula, data, types, nmin, maxdepth, ...) — then pass node_dipm(obj, ...) from the dipm package's exported API as the terminal_panel argument to partykit::plot. For continuous-outcome nodes, node_dipm draws treatment-group distributions as boxplots via ggplot2; for survival-outcome nodes it draws treatment-group survival curves using survfit objects from the survival package, all within partykit's plotting framework (grid graphics). Save each rendered tree plot as a PNG file. Note that dipm lists partykit, ggplot2, survival, rlang, stats, and utils under Imports: these are not attached automatically, so call library(<dep>) explicitly before plotting.

## Related tools

- **dipm** (Supplies the node_dipm(obj, ...) terminal-node panel function that draws boxplots (continuous) or survival plots (survival) compatible with partykit's plot method) — https://github.com/chenvict/dipm
- **dipm (R package)** (R package implementing the DIPM tree and forest method; its exported API includes node_dipm for visualization) — https://cran.r-project.org/package=dipm
- **partykit** (Provides the plot method into which node_dipm is registered as the terminal-node panel)
- **ggplot2** (Used by node_dipm to render treatment-group boxplots for continuous-outcome nodes)
- **survival** (Provides survfit objects underlying the treatment-group survival curves rendered for survival-outcome nodes)

## Examples

```
library(dipm); library(partykit); partykit::plot(spmtree_fit, terminal_panel = node_dipm)
```

## Evaluation signals

- The plot call completes only if node_dipm is passed as the terminal panel to partykit::plot; failure to attach partykit/ggplot2/survival explicitly (they are Imports, not Depends) should raise 'could not find function' errors that signal incorrect setup
- Continuous-outcome tree PNGs must show per-node boxplots stratified by treatment group; survival-outcome tree PNGs must show per-node survival curves by treatment group
- The rendered tree structure (splits, node counts) must match the fitted dipm/spmtree object, including any pruning applied via pmprune
- Each rendered tree plot is saved as a PNG file for the anorexia and GBSG2 trees as specified in the workflow

## Limitations

- Packages listed under Imports are NOT attached by loading dipm (only Depends is); partykit, ggplot2, and survival must be attached explicitly before plotting
- node_dipm is documented only as node_dipm(obj, ...); no software version, changelog, or dataset accessions are stated in the article or repository README

## Evidence

- [methods] node_dipm panel function documented: "- `node_dipm(obj, ...)`"
- [methods] partykit dependency for plotting: "- `partykit (Imports)`"
- [methods] Dependencies not attached automatically: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
- [readme] package scope from README: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
- [abstract] DIPM subgroup identification purpose: "The Depth Importance in Precision Medicine (DIPM) method is a classification tree designed for the identification of subgroups relevant to the precision medicine setting."
