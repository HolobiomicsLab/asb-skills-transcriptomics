---
name: lineage-fate-prediction-from-signature-genes
description: Use when you have scRNA-seq of progenitor populations plus FACS-sorted
  mature/differentiated subsets from which end-stage signature genes can be derived,
  and the goal is to assign individual progenitor cells to lineage-committed branches
  of a branching trajectory.
when_to_use_negative:
- The developmental process starts from multiple independent progenitor cell types
  — Mpath does not yet support this (planned future extension).
- The single-cell data set is highly heterogeneous with no known subset information,
  or no signature genes from FACS-sorted terminal subsets are available — Mpath depends
  on signature genes of differentiated cell types and end-stage contrasts for fate
  prediction.
- The goal is a linear, non-branching ordering — other tools (e.g. SCUBA-based clustering,
  used in this article's supplementary analyses) may be more appropriate.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Mpath
  role: 'Core algorithm: aggregates cells via neighborhood-based cell state transitions,
    builds the multi-branching trajectory, and predicts individual cell fate by travelling
    back in time from differentiated subsets using signature genes.'
  repo: JinmiaoChenLab/Mpath
- name: R
  role: Language in which the Mpath package is run; used for loading the GSE60781
    expression matrix, deriving signature genes via ANOVA/Tukey, and executing the
    trajectory/fate-prediction workflow.
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:eba53624ebc3a6c1cdf8947a30d5535ca89005195be4d4b15c9179c982601b66
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# lineage-fate-prediction-from-signature-genes

## Summary

Mpath-based prediction of individual cell fate along a multi-branching single-cell trajectory: the algorithm travels back in time from differentiated cell subsets (e.g. mature cDC1 and cDC2) using their signature genes to map each progenitor cell onto a lineage-committed branch. This reveals branching events such as the preDC-stage split into cDC1- versus cDC2-exclusive preDC subsets.

## When to use

Use when you have scRNA-seq of progenitor populations plus FACS-sorted mature/differentiated subsets from which end-stage signature genes can be derived, and the goal is to assign individual progenitor cells to lineage-committed branches of a branching trajectory.

## When NOT to use

- The developmental process starts from multiple independent progenitor cell types — Mpath does not yet support this (planned future extension).
- The single-cell data set is highly heterogeneous with no known subset information, or no signature genes from FACS-sorted terminal subsets are available — Mpath depends on signature genes of differentiated cell types and end-stage contrasts for fate prediction.
- The goal is a linear, non-branching ordering — other tools (e.g. SCUBA-based clustering, used in this article's supplementary analyses) may be more appropriate.

## Inputs

- scRNA-seq expression matrix of progenitor populations (e.g. mouse bone marrow MDP, CDP, preDC stages from GEO GSE60781) loaded in R
- Expression matrix of FACS-sorted terminal/differentiated subsets used to derive signature genes (e.g. mature cDC1 and cDC2 from GSE60782)
- Set of signature genes per mature cell type, derived from end-stage contrasts (ANOVA + post-hoc Tukey's test, Benjamini-Hochberg adjustment)

## Outputs

- Multi-branching aggregation tree spanning progenitor to preDC (e.g. MDP → CDP → preDC with clusters/landmarks designated)
- Per-cell fate predictions assigning individual cells to lineage branches (cDC1- vs cDC2-committed)
- Identification of a branching event and lineage-exclusive preDC subsets (e.g. preDC_a–preDC_c branch exclusively cDC2, preDC_a–preDC_d branch exclusively cDC1)

## How to apply

First derive signature genes of the mature cell types by identifying differentially expressed genes from end-stage contrasts of the FACS-sorted terminal subsets (e.g. mature cDC1 vs cDC2 from GSE60782), using ANOVA followed by post-hoc Tukey's test with Benjamini-Hochberg P-value adjustment. Then run Mpath in R: aggregate cells via neighborhood-based cell state transitions (using count of neighbouring cells to estimate potential state transitions), partition cells into clusters (e.g. 11 clusters with 8 landmark clusters for MDP, CDP and preDC stages), and build the aggregation tree without imposing a priori temporal ordering of developmental subsets. Finally, travel back in time from the differentiated cDC1 and cDC2 subsets using the derived signature genes to predict individual cell fate, tracing the developmental path from known terminal subsets back to progenitors. Correct application yields a multi-branching trajectory — for mouse bone marrow DC progenitors, an MDP-to-CDP-to-preDC progression with a branching event at the early preDC stage giving two preDC subsets exclusively committed to cDC1 or cDC2.

## Related tools

- **Mpath** (Core algorithm: aggregates cells via neighborhood-based cell state transitions, builds the multi-branching trajectory, and predicts individual cell fate by travelling back in time from differentiated subsets using signature genes.) — JinmiaoChenLab/Mpath
- **R** (Language in which the Mpath package is run; used for loading the GSE60781 expression matrix, deriving signature genes via ANOVA/Tukey, and executing the trajectory/fate-prediction workflow.)

## Examples

```
library(Mpath); expr <- read.table('GSE60781_expression.txt'); sig <- deriveSignature(expr_mature, method='anova_tukey_bh'); mpath_obj <- Mpath(expr, sig, ncluster=11, landmarks=8)
```

## Evaluation signals

- The trajectory recapitulates the expected biological progression, e.g. MDP → CDP → preDC for mouse DC progenitors, without imposing a priori temporal ordering.
- A branching event is detected at the expected stage (early preDC), with branches exclusively committed to one lineage each (preDC_a/preDC_b/preDC_c → cDC2; preDC_a/preDC_d → cDC1).
- Cluster structure is recovered as expected: e.g. 11 clusters with 8 landmark clusters corresponding to the FACS-sorted MDP, CDP and preDC populations.
- Cell ordering reveals biologically consistent dynamics, such as sequential waves of gene regulation and a switch from proliferation to differentiation during DC maturation.
- The clustering/state-transition mapping is stable when derived from end-stage contrast signature genes only (ANOVA + Tukey's test, Benjamini-Hochberg adjustment).

## Limitations

- Mpath does not yet support developmental processes starting from multiple independent progenitor cell types (planned future extension).
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown (planned future extension).
- Fate prediction depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets; demonstrated utility is limited to mouse bone marrow-derived DC progenitors and primary human myoblasts.

## Evidence

- [discussion] Mpath uses count of neighbouring cells to estimate potential state transition.: "Mpath uses count of neighbouring cells to estimate potential state transition. Via neighborhood-based cell state transition mapping"
- [discussion] The algorithm travels back in time from differentiated cell subsets and maps the developmental path from known terminal differentiated cell subsets back to its progenitors.: "The algorithm travels back in time from differentiated cell subsets and maps the developmental path from known terminal differentiated cell subsets back to its progenitors."
- [discussion] without requiring a priori knowledge of the temporal ordering of developmental subsets, and utilizing genes only from end-stage contrasts.: "without requiring a priori knowledge of the temporal ordering of developmental subsets, and utilizing genes only from end-stage contrasts."
- [abstract] The Mpath-generated trajectories detect a branching event at the preDC stage revealing preDC subsets that are exclusively committed to cDC1 or cDC2 lineages.: "The Mpath-generated trajectories detect a branching event at the preDC stage revealing preDC subsets that are exclusively committed to cDC1 or cDC2 lineages."
- [discussion] Mpath is able to predict individual cell's fate using signature genes of differentiated cell types.: "Mpath is able to predict individual cell's fate using signature genes of differentiated cell types."
- [discussion] The branching event led to two preDC subsets that were exclusively committed to cDC1 or cDC2 lineages.: "The branching event led to two preDC subsets that were exclusively committed to cDC1 or cDC2 lineages."
