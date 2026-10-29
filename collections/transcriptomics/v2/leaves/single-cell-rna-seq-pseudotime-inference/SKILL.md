---
name: single-cell-rna-seq-pseudotime-inference
description: Use when a gene-by-cell scRNA-seq matrix of a differentiating lineage
  needs pseudotime ordering or linear/branched trajectory detection, and development-related
  GO genes plus cell-type labels (e.g. llorens_labels) are available for benchmarking.
when_to_use_negative:
- The goal is integrative, simultaneous modeling of proliferation and differentiation
  coupling rather than visualization; the paper notes integrative models are preferred
  and the underlying mechanism remains obscure.
- Data is already a feature/gene-significance table or the trajectory tools' pseudotime
  results perform poorly — the transition-matrix labeling-free approach 'may fail
  when the pseudotime results perform poorly'.
- The input requires removal of cycling genes to correct cell cycle effects, which
  the paper notes is the traditional approach and redPATH does not rely on.
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: redPATH
  role: 'Main method: cHMT pseudotime inference, GO subsetting, dispersion filtering,
    G0 detection, transition-matrix PCA trajectory detection'
  repo: https://github.com/tinglabs/redPATH
- name: SCORPIUS
  role: Baseline pseudotime method for benchmark comparison on identical filtered
    matrices
- name: TSCAN
  role: Baseline pseudotime method for benchmark comparison on identical filtered
    matrices
- name: Monocle2
  role: Baseline pseudotime method for benchmark comparison on identical filtered
    matrices
- name: DPT
  role: Baseline pseudotime method for benchmark comparison on identical filtered
    matrices
- name: reCAT
  role: Provides mean cell cycle scores used to identify and remove G0-like quiescent
    cells prior to pseudotime inference
- name: ggplot2
  role: Plotting backend for trajectory and 3D visualization utilities
- name: plotly
  role: 3D visualization of the embedded branched trajectory colored by cell type
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.06.014
    title: 'redPATH: Reconstructing the Pseudo Development Time of Cell Lineages in
      Single-Cell RNA-Seq Data and Applications in Cancer'
  source_content_hash: sha256:ef88827b849a0e7374f9ffe0c7c047a66b3fd72acb8e8c374a16b361aa838c17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tinglabs/redPATH
  tools:
  - DPT
  - Monocle2
  - R
  - SCORPIUS
  - TSCAN
  - ggplot2
  - plotly
  - reCAT
  - redPATH
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7058823529411765
n_claims: 17
---

# single-cell-rna-seq-pseudotime-inference

## Summary

Reconstruct pseudo developmental time of cell lineages from single-cell RNA-seq using redPATH, which formulates pseudotime ordering as a consensus asymmetric Hamiltonian path (cHMT) problem and supports linear or branched trajectory detection via transition-matrix PCA.

## When to use

Use when a gene-by-cell scRNA-seq matrix of a differentiating lineage needs pseudotime ordering or linear/branched trajectory detection, and development-related GO genes plus cell-type labels (e.g. llorens_labels) are available for benchmarking.

## When NOT to use

- The goal is integrative, simultaneous modeling of proliferation and differentiation coupling rather than visualization; the paper notes integrative models are preferred and the underlying mechanism remains obscure.
- Data is already a feature/gene-significance table or the trajectory tools' pseudotime results perform poorly — the transition-matrix labeling-free approach 'may fail when the pseudotime results perform poorly'.
- The input requires removal of cycling genes to correct cell cycle effects, which the paper notes is the traditional approach and redPATH does not rely on.

## Inputs

- gene-by-cell raw expression matrix (e.g. llorens_exprs, 145 neural stem cells; MGH107, 252 glioma cells)
- cell type / stage labels (e.g. llorens_labels) for evaluation and plot coloring
- optional reCAT mean cell cycle scores (e.g. llorens_reCAT_sample_data) for G0-like cell detection

## Outputs

- redpath_pseudotime vector covering all cells
- CI, BI, KC (and POS) evaluation scores per method-dataset pair (e.g. figure2b_benchmark_scores.csv, pseudotime_evaluation_metrics.csv)
- transition-matrix PCA trajectory visualization exposing branching (e.g. figure5c_branching_trajectory.png)

## How to apply

Preprocess the m genes x n cells matrix by subsetting to Gene Ontology development genes (get_GO_exp, gene_type='gene_names', organism='mouse', takelog=F) and applying dispersion filtering (filter_exp, dispersion_threshold=10, threads=2). Optionally remove G0-like quiescent cells by kmeans_clustering (k=5) on reCAT mean scores plus statistical_test (threshold=0.001) before filtering. Run redpath(filtered_GO_data, threadnum=4, base_path_range=c(4:8)), where base_path_range spans the estimated number of cell types k0 to k0+4, to obtain pseudotime from consensus asymmetric Hamiltonian paths. For trajectory shape, combine the multiple Hamiltonian path solutions (norm_res_new) into an N-by-N cell transition probability matrix with get_branch_viz_data and visualize by PCA with plot_trajectory. Judge correctness by comparing metrics against baselines (SCORPIUS, TSCAN, Monocle2, DPT) using CI, BI, KC, and POS scores, or against reported values (e.g. NSC-Llorens after G0 removal: redPATH CI 0.868, BI 0.973, KC 0.832 vs SCORPIUS CI 0.849, BI 0.768, KC 0.566).

## Related tools

- **redPATH** (Main method: cHMT pseudotime inference, GO subsetting, dispersion filtering, G0 detection, transition-matrix PCA trajectory detection) — https://github.com/tinglabs/redPATH
- **SCORPIUS** (Baseline pseudotime method for benchmark comparison on identical filtered matrices)
- **TSCAN** (Baseline pseudotime method for benchmark comparison on identical filtered matrices)
- **Monocle2** (Baseline pseudotime method for benchmark comparison on identical filtered matrices)
- **DPT** (Baseline pseudotime method for benchmark comparison on identical filtered matrices)
- **reCAT** (Provides mean cell cycle scores used to identify and remove G0-like quiescent cells prior to pseudotime inference)
- **ggplot2** (Plotting backend for trajectory and 3D visualization utilities)
- **plotly** (3D visualization of the embedded branched trajectory colored by cell type)

## Examples

```
subset_GO_data <- get_GO_exp(llorens_exprs, gene_type="gene_names", organism="mouse", takelog=F); filtered_GO_data <- filter_exp(subset_GO_data, dispersion_threshold=10, threads=2); redpath_pseudotime <- redpath(filtered_GO_data, threadnum=4, base_path_range=c(4:8))
```

## Evaluation signals

- redPATH attains the best CI, BI, KC, POS scores across all method-dataset pairs in the assembled benchmark table (figure2b_benchmark_scores.csv).
- On NSC-Llorens after G0 removal (56 cells), computed redPATH metrics match CI 0.868, BI 0.973, KC 0.832, beating SCORPIUS (CI 0.849, BI 0.768, KC 0.566).
- The returned pseudotime vector covers every cell from the redpath() call (no cells dropped during filtering).
- Transition-matrix PCA plot reproduces the two-branch structure of Figure 5C (file_exists on figure5c_branching_trajectory.png).
- redpath() automatically saves 'norm_res_new' Hamiltonian path solutions to the global environment, confirming the cHMT stage completed.

## Limitations

- The transition-matrix labeling-free trajectory approach may fail when pseudotime results perform poorly.
- Traditional cell-cycle correction via removal of cycling genes is not what redPATH does; coupling between proliferation and differentiation remains obscure and integrative models are preferred for simultaneous analysis.
- base_path_range must be set from an estimate of the number of cell types k0 to k0+4; poor k0 estimates may degrade results.
- On very small datasets the method is reliable only after removing G0-like cells (e.g. 56 cells remained from the 145-cell llorens dataset).

## Evidence

- [abstract] consensus asymmetric Hamiltonian path: "we developed a computational method, called redPATH, to reconstruct the pseudo developmental time of cell lineages using a consensus asymmetric Hamiltonian path algorithm"
- [methods] GO subsetting and dispersion filtering: "subset_GO_data <- get_GO_exp(input_data, gene_type = "gene_names", organism = "mouse", takelog = F)"
- [methods] dispersion threshold: "filtered_GO_data <- filter_exp(subset_GO_data, dispersion_threshold = 10, threads = 2)"
- [readme] redpath invocation with base_path_range: "redpath_pseudotime <- redpath(filtered_GO_data, threadnum = 4, base_path_range = c(4:8))
# base_path_range is set to the estimated number of cell types, k0, and k0 +4."
- [intro] transition matrix to visualize trajectory: "Additionally, we developed an approach to decipher and combine multiple Hamiltonian path solutions into a transition matrix to visualize the trajectory (linear or branched) developmental trend."
- [readme] transition matrix visualization via PCA: [evidence span withheld — anchor status: external_doc]
- [discussion] failure mode of labeling-free approach: "Although the advantage is that cell type labeling is not required here, this approach may fail when the pseudotime results perform poorly."
- [methods] neural stem cell example dataset: "a neural stem cell (145 cells) dataset from 2015 (1) is used as an example analysis"
