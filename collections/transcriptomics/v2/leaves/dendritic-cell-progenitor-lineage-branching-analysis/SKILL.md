---
name: dendritic-cell-progenitor-lineage-branching-analysis
description: Use when a single-cell RNA-seq dataset of dendritic cell progenitors
  (CDP through preDC stages) must be checked for a branching event separating cDC1-
  versus cDC2-committed preDC subsets, and when benchmarking which trajectory method
  recovers that branch.
when_to_use_negative:
- Development processes starting from multiple independent progenitor cell types —
  Mpath does not yet support these (planned future extension)
- Highly heterogeneous single-cell datasets in which specific subset information is
  not known — Mpath depends on signature genes of FACS-sorted terminal subsets for
  fate prediction
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Mpath
  role: Reference trajectory-inference method whose neighborhood-based multi-branching
    trajectory resolves the preDC_c (cDC1) / preDC_d (cDC2) branch
  repo: JinmiaoChenLab/Mpath
- name: SCUBA
  role: Benchmarked baseline; checked for whether its trajectory contains a branching
    event separating cDC1- and cDC2-committed preDC subsets
- name: R
  role: Environment for running all methods and writing trajectory_method_comparison.csv;
    Mpath ships as an R package
  repo: JinmiaoChenLab/Mpath
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:75614d17333b7f4a1ad6997af03ad3cc0c241e89a84570de1d9877c3b71ee5b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  - SCUBA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# dendritic-cell-progenitor-lineage-branching-analysis

## Summary

Benchmark of trajectory-inference methods (Mpath, SCUBA, monocle, diffusion map, PCA, conventional MST) on the mouse DC progenitor single-cell RNA-seq dataset to test which detects the preDC-stage branching into cDC1- and cDC2-committed subsets.

## When to use

Use when a single-cell RNA-seq dataset of dendritic cell progenitors (CDP through preDC stages) must be checked for a branching event separating cDC1- versus cDC2-committed preDC subsets, and when benchmarking which trajectory method recovers that branch.

## When NOT to use

- Development processes starting from multiple independent progenitor cell types — Mpath does not yet support these (planned future extension)
- Highly heterogeneous single-cell datasets in which specific subset information is not known — Mpath depends on signature genes of FACS-sorted terminal subsets for fate prediction

## Inputs

- QC-passed mouse DC progenitor single-cell expression matrix (CDP through preDC stages)
- Differentially expressed gene signature lists for mature cDC1 and cDC2 (Supplementary Data 1)

## Outputs

- Per-method branch-detection outcomes for SCUBA, monocle, diffusion map, PCA, MST and Mpath
- trajectory_method_comparison.csv tabulating branch detected (yes/no) and placement of preDC_c (cDC1) and preDC_d (cDC2)

## How to apply

Load the QC-passed DC progenitor expression matrix and the mature cDC1/cDC2 DEG signature lists (Supplementary Data 1) into R. Run each comparison method in turn: SCUBA (check whether its inferred trajectory contains a branch separating cDC1- and cDC2-committed preDCs), monocle (check whether its single-path pseudotime ordering captures or misses the cDC1/cDC2 preDC branch), a diffusion-map and a PCA embedding (check visually for cDC1/cDC2 preDC branching), and a conventional minimum spanning tree on Euclidean distances (record branch placement of preDC_c and preDC_d). Then run Mpath on the identical input as the reference method — it derives multi-branching developmental trajectories using neighborhood-based cell state transitions and predicts cell fate via signature genes of the terminal cDC1/cDC2 subsets, without requiring a priori temporal ordering. Tabulate per-method outcomes (branch detected yes/no, placement of preDC_c and preDC_d) and write trajectory_method_comparison.csv.

## Related tools

- **Mpath** (Reference trajectory-inference method whose neighborhood-based multi-branching trajectory resolves the preDC_c (cDC1) / preDC_d (cDC2) branch) — JinmiaoChenLab/Mpath
- **SCUBA** (Benchmarked baseline; checked for whether its trajectory contains a branching event separating cDC1- and cDC2-committed preDC subsets)
- **R** (Environment for running all methods and writing trajectory_method_comparison.csv; Mpath ships as an R package) — JinmiaoChenLab/Mpath

## Evaluation signals

- Mpath, and only Mpath, resolves the branching of cDC1- and cDC2-committed preDCs on the same input
- Conventional MST is expected to place preDC_c and preDC_d on the wrong branch — this known failure is a positive control for the benchmark
- monocle's single-path pseudotime should miss (not capture) the cDC1/cDC2 preDC branch
- Neither the diffusion map nor the PCA embedding should visually separate cDC1- and cDC2-committed preDCs
- trajectory_method_comparison.csv exists and contains a row per method with branch-detected yes/no and preDC_c/preDC_d placement columns

## Limitations

- Conventional MST on Euclidean distances misplaces cDC1- and cDC2-committed preDCs, so tree-based baselines cannot be trusted for this branch
- Demonstrated utility is limited to two datasets: mouse bone marrow-derived DC progenitors and primary human myoblasts
- Mpath does not yet support development processes starting from multiple independent progenitor cell types
- Mpath requires signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction

## Evidence

- [abstract] Mpath-generated trajectories detect a branching event at the preDC stage: "The Mpath-generated trajectories detect a branching event at the preDC stage revealing preDC subsets that are exclusively committed to cDC1 or cDC2 lineages."
- [abstract] Mpath algorithm derives multi-branching trajectories via neighborhood-based transitions: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [discussion] Branching led to two preDC subsets exclusively committed to cDC1 or cDC2: "The branching event led to two preDC subsets that were exclusively committed to cDC1 or cDC2 lineages."
- [discussion] Mpath fate prediction uses signature genes of differentiated cell types: "Mpath is able to predict individual cell's fate using signature genes of differentiated cell types."
- [readme] Mpath README: algorithm for constructing multi-branching cell lineages: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
