---
name: branch-assignment-in-differentiation-trajectories
description: Use when an Mpath trajectory contains a branching event and you must
  isolate which cells commit to each distinct differentiation path, such as muscle
  versus non-muscle fates in myoblast differentiation.
when_to_use_negative:
- Development processes starting from multiple independent progenitor cell types —
  Mpath does not yet support this (planned future extension)
- Highly heterogeneous single-cell datasets in which specific subset information is
  not known
- Analyses lacking signature genes of differentiated cell types or FACS-sorted terminal
  subsets, which Mpath depends on for fate prediction
tools:
- name: Mpath
  role: Neighborhood-based cell state transition mapping that constructs the multi-branching
    trajectory and assigns individual cells to branches using signature genes of differentiated
    cell types
  repo: JinmiaoChenLab/Mpath
- name: R
  role: Implementation environment for Mpath (R package) and for the ANOVA/Tukey DEG
    identification that feeds trajectory construction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:3d9bcdd58121006e022130eddb3a46f3ddd82812212df64335fe52723b136e39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# Branch assignment in differentiation trajectories

## Summary

Assigns individual cells to specific branches of an Mpath-constructed multi-branching trajectory (e.g. separating the muscle differentiation path marked by MYOG, MYOD1, MEF2C from the non-muscle branch marked by PDGFRA in primary human myoblast scRNA-seq), enabling fate prediction after a branching event.

## When to use

Use when an Mpath trajectory contains a branching event and you must isolate which cells commit to each distinct differentiation path, such as muscle versus non-muscle fates in myoblast differentiation.

## When NOT to use

- Development processes starting from multiple independent progenitor cell types — Mpath does not yet support this (planned future extension)
- Highly heterogeneous single-cell datasets in which specific subset information is not known
- Analyses lacking signature genes of differentiated cell types or FACS-sorted terminal subsets, which Mpath depends on for fate prediction

## Inputs

- Mpath-constructed multi-branching trajectory with designated landmark states (from primary human myoblast scRNA-seq, GEO GSE52529)
- ANOVA-derived DEGs across 0/24/48/72 h differentiation time points (Benjamini-Hochberg adjusted)
- Signature genes of differentiated cell types (end-stage contrasts), e.g. MYOG/MYOD1/MEF2C for muscle, PDGFRA for non-muscle

## Outputs

- Per-cell branch assignment labels separating muscle and non-muscle differentiation paths
- Trajectory CSV containing 8 designated landmark states and a two-branch split between muscle and non-muscle cells
- Branch-specific gene regulation waves (7 ordered temporal waves in the human myoblast analysis)

## How to apply

First run Mpath's neighborhood-based cell state transition mapping on ANOVA-derived DEGs (from the 0/24/48/72 h time points with Benjamini-Hochberg correction) to construct the multi-branching trajectory and designate landmark states (8 landmark states in the human myoblast analysis). Mpath estimates potential state transitions via counts of neighboring cells and can predict individual cell fate by travelling back in time from differentiated cell subsets using their signature genes. At the branching event, assign each cell to a branch and label branches by their marker expression: the muscle path upregulates MYOG, MYOD1 and MEF2C, while the non-muscle (interstitial mesenchymal) path upregulates PDGFRA. Validate branch assignment by confirming the expected two-way split and by ordering cells along each branch to detect sequential temporal waves of gene regulation.

## Related tools

- **Mpath** (Neighborhood-based cell state transition mapping that constructs the multi-branching trajectory and assigns individual cells to branches using signature genes of differentiated cell types) — JinmiaoChenLab/Mpath
- **R** (Implementation environment for Mpath (R package) and for the ANOVA/Tukey DEG identification that feeds trajectory construction)

## Examples

```
Rscript myoblast_mpath_branch_assignment.R  # loads GSE52529, ANOVA DEGs, runs Mpath neighborhood mapping, assigns cells to muscle (MYOG/MYOD1/MEF2C) vs non-muscle (PDGFRA) branches
```

## Evaluation signals

- Trajectory CSV contains exactly 8 designated landmark states, matching the reported human myoblast analysis
- A two-branch split at the branching event cleanly separates muscle cells (upregulated MYOG, MYOD1, MEF2C) from non-muscle/interstitial mesenchymal cells (upregulated PDGFRA)
- Reordering cells along each branch recovers 7 ordered temporal waves of transcriptional changes
- All cells from the GSE52529 time course are assigned to a branch with no unassigned cells after fate prediction
- Supplementary Data 4/5-style differential expression between the muscle and non-muscle paths yields the reported branch-specific marker genes

## Limitations

- Mpath does not yet support development processes starting from multiple independent progenitor cell types (planned future extension)
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown (planned future extension)
- Fate prediction depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets
- Demonstrated utility limited to two data sets: mouse bone marrow-derived DC progenitors and primary human myoblasts

## Evidence

- [abstract] Applied to human myoblasts, Mpath recapitulates the time course of myoblast differentiation and isolates a branch of non-muscle cells: "Applied to human myoblasts, Mpath recapitulates the time course of myoblast differentiation and isolates a branch of non-muscle cells involved in the differentiation."
- [discussion] Mpath uses count of neighbouring cells to estimate potential state transition.: "Mpath uses count of neighbouring cells to estimate potential state transition. Via neighborhood-based cell state transition mapping"
- [discussion] Mpath is able to predict individual cell's fate using signature genes of differentiated cell types.: "Mpath is able to predict individual cell's fate using signature genes of differentiated cell types."
- [discussion] multiple cell fates stemming from a single progenitor cell type: [evidence span withheld — anchor status: not_found]
- [readme] Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
