---
name: temporal-gene-expression-wave-detection
description: Use when a single-cell RNA-seq time course (e.g. 0/24/48/72 h myoblast
  differentiation) already has a trajectory/branch structure and you need to order
  cells to identify sequential temporal waves of gene regulation along the path.
when_to_use_negative:
- Mpath does not yet support development processes starting from multiple independent
  progenitor cell types, so do not apply it to data with several unrelated progenitor
  populations
- Mpath is not yet extended to highly heterogeneous single-cell data sets in which
  specific subset information is not known, so avoid unannotated, deeply heterogeneous
  data
- Trajectory construction requires signature genes of differentiated cell types (end-stage
  contrasts) and FACS-sorted terminal subsets for fate prediction; without terminal
  subset information, cell ordering/fate prediction will not be reliable
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Mpath
  role: R package performing neighborhood-based cell state transition mapping, landmark
    designation, branch assignment, and reordering of single cells along the reconstructed
    trajectory
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: R
  role: Runtime environment; Mpath is distributed and documented as an R package (Mpath
    reference manual)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:e4f89a0d66b1fae5fc189b4172566dcd0d965d886f72d278c677315ed04a9985
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# temporal-gene-expression-wave-detection

## Summary

Detects sequential, phase-dependent waves of gene regulation by ordering single cells along an Mpath-reconstructed multi-branching trajectory, revealing temporal coupling between biological processes (e.g. cell cycle and differentiation) and temporal waves of marker up- or down-regulation.

## When to use

Use when a single-cell RNA-seq time course (e.g. 0/24/48/72 h myoblast differentiation) already has a trajectory/branch structure and you need to order cells to identify sequential temporal waves of gene regulation along the path.

## When NOT to use

- Mpath does not yet support development processes starting from multiple independent progenitor cell types, so do not apply it to data with several unrelated progenitor populations
- Mpath is not yet extended to highly heterogeneous single-cell data sets in which specific subset information is not known, so avoid unannotated, deeply heterogeneous data
- Trajectory construction requires signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction; without terminal subset information, cell ordering/fate prediction will not be reliable

## Inputs

- Single-cell RNA-seq expression matrix (primary human myoblast scRNA-seq, GEO GSE52529, sampled across 0/24/48/72 h differentiation)
- ANOVA-derived DEG list across the 0/24/48/72 h time points (Benjamini-Hochberg adjusted P values)
- Mpath multi-branching trajectory with designated landmark states and branch assignments

## Outputs

- Ordered (pseudotemporally sorted) cells along each trajectory branch
- Sequential temporal waves of gene regulation (7 ordered waves in the human myoblast analysis)
- Trajectory CSV containing 8 landmark states and the two-branch muscle/non-muscle split with cells placed in order

## How to apply

First run Mpath's neighborhood-based cell state transition mapping on ANOVA-derived DEGs (ANOVA plus post-hoc Tukey's test, P values adjusted with Benjamini-Hochberg) to construct the multi-branching trajectory and designate landmark states. Assign individual cells to branches, separating e.g. the muscle path (MYOG, MYOD1, MEF2C) from the non-muscle path (PDGFRA) at the branching event. Then reorder individual cells along the reconstructed trajectory so that expression changes can be examined as ordered sequences rather than cluster averages. This ordering reveals phase-dependent waves of gene regulation on the scale of the differentiation continuum, and in the DC progenitor example exposed temporal coupling between cell cycle and differentiation (a switch from proliferation to differentiation during maturation). Validate by confirming the expected number of ordered waves (7 in the human myoblast case) and that the ordering recapitulates the 0/24/48/72 h time course.

## Related tools

- **Mpath** (R package performing neighborhood-based cell state transition mapping, landmark designation, branch assignment, and reordering of single cells along the reconstructed trajectory) — https://github.com/JinmiaoChenLab/Mpath
- **R** (Runtime environment; Mpath is distributed and documented as an R package (Mpath reference manual))

## Examples

```
Rscript -e 'library(Mpath); # run Mpath trajectory on ANOVA-derived DEGs, assign cells to branches, then reorder cells along the trajectory to detect temporal waves'
```

## Evaluation signals

- The output trajectory CSV contains the 8 designated landmark states reported for the human myoblast analysis
- Cells split into two branches at the branching event: muscle path upregulating MYOG, MYOD1, MEF2C and non-muscle path upregulating PDGFRA
- The ordering yields 7 sequential temporal waves of gene regulation, matching the reported count for the myoblast data set
- Ordered cell positions recapitulate the 0/24/48/72 h differentiation time course
- DEGs used to build the trajectory derive from ANOVA plus post-hoc Tukey's test with Benjamini-Hochberg multi-test correction

## Limitations

- Mpath cannot yet start from multiple independent progenitor cell types (planned future extension)
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown (planned future extension)
- Demonstrated utility is limited to two data sets: mouse bone marrow-derived DC progenitors and primary human myoblasts
- Fate prediction depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets

## Evidence

- [abstract] Reordering cells along cDC development reveals sequential waves of gene regulation: "Reordering cells along cDC development reveals sequential waves of gene regulation and temporal coupling between cell cycle and cDC differentiation."
- [discussion] Phase-dependent waves across the differentiation continuum: "its ordering of single cells discovered phase-dependent waves of gene regulation on the scale of DC development and subset differentiation continuum."
- [discussion] Switch from proliferation to differentiation: "it placed individual cells in order along the developmental trajectories, revealing a switch from proliferation to differentiation during DC maturation."
- [supplementary] DEG identification method: "The DEGs were identified using ANOVA followed by post-hoc Tukey's test."
- [supplementary] Mpath is an R package: "Mpath reference manual provides detailed documentation of Mpath R package."
- [readme] Mpath purpose: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
