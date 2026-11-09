---
name: node-panel-function-reconstruction
description: Use when a fitted dipm or spmtree tree object needs per-node subgroup
  visualization via partykit::plot, and the tree's outcome is continuous (e.g. anorexia)
  or survival (e.g. GBSG2).
when_to_use_negative:
- When the goal is fitting or pruning a DIPM tree (use dipm or spmtree with pmprune)
  rather than visualizing an already-fitted tree's terminal nodes
- When the tree object is not of a partykit-compatible class (e.g. a raw rpart or
  random-forest result without a party/partykit representation), since node_dipm only
  works as a panel inside partykit::plot
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_2269
tools:
- name: dipm
  role: Provides the exported node_dipm(obj, ...) panel function for per-node subgroup
    visualization
  repo: https://github.com/chenvict/dipm
- name: partykit
  role: Plot framework into which node_dipm is passed as the terminal-node panel
- name: ggplot2
  role: Renders treatment-group boxplots for continuous-outcome nodes
- name: survival
  role: Supplies survfit objects for drawing treatment-group survival curves in survival-outcome
    nodes
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbac041
    title: 'dipm: an R package implementing the Depth Importance in Precision Medicine
      (DIPM) tree and Forest-based method'
  source_content_hash: sha256:24cbdbf908d9318611d4ac2c839095e132d1756fcd41dd1b656a8751c43b7c82
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
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# node-panel-function-reconstruction

## Summary

Implements a custom terminal-node panel function (dipm's node_dipm) so fitted dipm/spmtree trees can be visualized per node through partykit's plot method, rendering treatment-group boxplots for continuous outcomes and survival curves for survival outcomes.

## When to use

Use when a fitted dipm or spmtree tree object needs per-node subgroup visualization via partykit::plot, and the tree's outcome is continuous (e.g. anorexia) or survival (e.g. GBSG2).

## When NOT to use

- When the goal is fitting or pruning a DIPM tree (use dipm or spmtree with pmprune) rather than visualizing an already-fitted tree's terminal nodes
- When the tree object is not of a partykit-compatible class (e.g. a raw rpart or random-forest result without a party/partykit representation), since node_dipm only works as a panel inside partykit::plot

## Inputs

- Fitted dipm tree object (e.g. on the anorexia continuous-outcome dataset)
- Fitted spmtree object (e.g. on the GBSG2 survival-outcome dataset)
- Node panel function node_dipm(obj, ...) from the dipm package's exported API

## Outputs

- Tree plot with per-node treatment-group boxplots (continuous outcome), saved as a PNG file
- Tree plot with per-node treatment-group survival curves (survival outcome), saved as a PNG file

## How to apply

Load the fitted dipm/spmtree tree objects — the article's workflow uses the anorexia (continuous outcome) and GBSG2 (survival outcome) datasets. Implement the exported node_dipm(obj, ...) panel function from the dipm package API so it draws a per-node visualization compatible with partykit's plot method. For continuous-outcome nodes, render treatment-group distributions as boxplots using ggplot2; for survival-outcome nodes, render treatment-group survival curves using survival package survfit objects within the partykit plotting framework. Finally call partykit::plot on the anorexia and GBSG2 trees with node_dipm as the terminal-node panel, saving each rendered tree plot as a PNG. Note that dipm's package dependencies (ggplot2, survival, partykit, stats) are listed under Imports, not Depends, so each must be attached explicitly with library() before use.

## Related tools

- **dipm** (Provides the exported node_dipm(obj, ...) panel function for per-node subgroup visualization) — https://github.com/chenvict/dipm
- **partykit** (Plot framework into which node_dipm is passed as the terminal-node panel)
- **ggplot2** (Renders treatment-group boxplots for continuous-outcome nodes)
- **survival** (Supplies survfit objects for drawing treatment-group survival curves in survival-outcome nodes)

## Examples

```
R> plot(tree, terminal_panel = node_dipm)
```

## Evaluation signals

- Calling partykit::plot(tree, terminal_panel = node_dipm) executes without error for both the anorexia (continuous) and GBSG2 (survival) trees
- Continuous-outcome nodes display ggplot2-based boxplots stratified by treatment group; survival-outcome nodes display survival-package-based curves stratified by treatment group
- Both rendered tree plots are written successfully as PNG files
- The panel function is the exported node_dipm(obj, ...) from dipm's API, invoked with the partykit plot method's expected signature (obj, ...)

## Limitations

- Packages listed under Dependencies are NOT attached by loading dipm (only Depends is); ggplot2, survival, and partykit must be attached explicitly before their functions are used
- node_dipm depends on dipm's tree objects being partykit-compatible (dipm and partykit are both Imports of the package); trees from other fitting engines may not render
- No software version is stated for dipm in the source, and the GitHub repository README contains no changelog, so version-specific behavior of node_dipm cannot be verified from the provided context

## Evidence

- [methods] workflow step 2: [evidence span withheld — anchor status: not_found]
- [methods] workflow finding: [evidence span withheld — anchor status: not_found]
- [methods] workflow step 5: [evidence span withheld — anchor status: not_found]
- [methods] dependency attachment: "Packages listed under Dependencies are NOT attached by loading this package (only Depends is); call library(<dep>) explicitly before using their functions."
- [readme] package scope: "R package for implementing the Depth Importance in Precision Medicine (DIPM) tree and forest based method"
