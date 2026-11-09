---
name: pca-embedding-of-trajectory-data
description: Use when pseudotime has been inferred with redPATH's consensus Hamiltonian
  path algorithm and you need to detect and visualize whether the developmental trajectory
  is linear or branched.
when_to_use_negative:
- When pseudotime results perform poorly — the article notes this labeling-free approach
  'may fail when the pseudotime results perform poorly'
- When the goal is simultaneous integrative analysis of proliferation and differentiation
  rather than trajectory visualization — the article states integrative models are
  preferred and the current visualization is limited
- When cell type labels are required as input — this approach was designed so that
  'cell type labeling is not required here'
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: redPATH
  role: Provides redpath() to generate multiple Hamiltonian path solutions, get_branch_viz_data()
    to build the transition matrix, and plot_trajectory() for PCA-based trajectory
    visualization
  repo: https://github.com/tinglabs/redPATH
- name: ggplot2
  role: Rendering of the 2-panel trajectory plots produced by plot_trajectory()
- name: plotly
  role: 3D visualization of the embedded trajectory (plot_diff_3d / plot_cycle_diff_3d)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:167e79b1694bb0b71b715773d721740b124a7a505452d9308a68dec8dffcebee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - ggplot2
  - plotly
  - redPATH
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# pca-embedding-of-trajectory-data

## Summary

redPATH's trajectory visualization approach: combine multiple asymmetric Hamiltonian path solutions from the redpath() call into an N-by-N cell transition probability matrix, then apply PCA to embed cells in a low-dimensional space that exposes linear or branched developmental trends.

## When to use

Use when pseudotime has been inferred with redPATH's consensus Hamiltonian path algorithm and you need to detect and visualize whether the developmental trajectory is linear or branched.

## When NOT to use

- When pseudotime results perform poorly — the article notes this labeling-free approach 'may fail when the pseudotime results perform poorly'
- When the goal is simultaneous integrative analysis of proliferation and differentiation rather than trajectory visualization — the article states integrative models are preferred and the current visualization is limited
- When cell type labels are required as input — this approach was designed so that 'cell type labeling is not required here'

## Inputs

- norm_res_new: multiple Hamiltonian path solutions automatically saved to the global environment by the redpath() function
- redpath_pseudotime: pseudotime vector returned by redpath() covering all cells
- Optional cell type label vector (e.g., llorens_labels or hHSC type labels)
- Pre-filtered gene-by-cell expression matrix (GO-subset, dispersion_threshold = 10)

## Outputs

- trajectory_visualization_data: transition probability matrix embedded via PCA
- trajectory_plots: 2-panel ggplot2 visualization colored by pseudotime and cell type showing linear/branched structure
- 3D trajectory/proliferation-differentiation plots via plotly-based utilities (plot_diff_3d)

## How to apply

After running redpath(filtered_GO_data, threadnum = 4, base_path_range = c(4:8)), retrieve the norm_res_new object automatically saved to the global environment, which holds the multiple Hamiltonian path solutions. Pass these solutions together with the redpath_pseudotime vector to get_branch_viz_data(), optionally supplying cell type labels, to sum transition probabilities between cells and build the transition matrix. Apply PCA to this matrix (via plot_trajectory()) so cells embed in a low-dimensional space exposing the branched or linear developmental trend. Visualize the two panels (pseudotime-colored and label-colored) with grid.arrange; for the hHSC dataset this reproduces the binary cell fate branching of Figure 5C.

## Related tools

- **redPATH** (Provides redpath() to generate multiple Hamiltonian path solutions, get_branch_viz_data() to build the transition matrix, and plot_trajectory() for PCA-based trajectory visualization) — https://github.com/tinglabs/redPATH
- **ggplot2** (Rendering of the 2-panel trajectory plots produced by plot_trajectory())
- **plotly** (3D visualization of the embedded trajectory (plot_diff_3d / plot_cycle_diff_3d))

## Examples

```
hamiltonian_path_solutions <- norm_res_new; trajectory_visualization_data <- get_branch_viz_data(hamiltonian_path_solutions, redpath_pseudotime, labels = NULL); trajectory_plots <- plot_trajectory(trajectory_visualization_data, redpath_pseudotime)
```

## Evaluation signals

- The output plot reproduces the two-branch structure of Figure 5C when applied to the hHSC dataset (script_runs and file_exists on figure5c_branching_trajectory.png)
- The returned pseudotime vector from redpath() covers all cells in the input matrix
- norm_res_new is present in the global environment after the redpath() call and can be passed to get_branch_viz_data() without error
- get_branch_viz_data() returns an N-by-N transition-based embedding covering all cells, and plot_trajectory() returns exactly 2 panels arrangeable with matrix(c(1:2), 1, 2)

## Limitations

- This approach may fail when the pseudotime results perform poorly, even though cell type labeling is not required
- The transition-matrix PCA is a visualization aid; integrative models are preferred to analyze proliferation and differentiation simultaneously, and the underlying coupling mechanism remains obscure
- 'norm_res_new' is overwritten every time the redpath() function is performed, so solutions must be captured before re-running

## Evidence

- [intro] Additionally, we developed an approach to decipher and combine multiple Hamiltonian path solutions into a transition matrix to visualize the trajectory (linear or branched) developmental trend.: "Additionally, we developed an approach to decipher and combine multiple Hamiltonian path solutions into a transition matrix to visualize the trajectory (linear or branched) developmental trend."
- [readme] We sum the probabilities of transitions between cells and visualize the transition matrix by PCA.: [evidence span withheld — anchor status: external_doc]
- [readme] Note that, 'norm_res_new' is automatically saved to the global environment every time the 'redpath' function is performed. It will be overwritten.: [evidence span withheld — anchor status: external_doc]
- [discussion] Although the advantage is that cell type labeling is not required here, this approach may fail when the pseudotime results perform poorly.: "Although the advantage is that cell type labeling is not required here, this approach may fail when the pseudotime results perform poorly."
- [readme] trajectory_visualization_data <- get_branch_viz_data(hamiltonian_path_solutions, redpath_pseudotime, labels = NULL): [evidence span withheld — anchor status: external_doc]
