---
name: single-cell-trajectory-inference
description: Use when scRNA-seq data from a differentiating progenitor population
  must be ordered into a multi-branching trajectory with landmark states, without
  a priori temporal ordering of subsets, to detect branching fate decisions.
when_to_use_negative:
- Developmental processes start from multiple independent progenitor cell types —
  Mpath does not yet support this (planned future extension)
- Highly heterogeneous datasets in which specific subset information is not known
  — Mpath depends on signature genes of differentiated cell types and FACS-sorted
  terminal subsets for fate prediction
- The input is already a trajectory or pseudotime ordering and the goal is only gene-wave
  detection rather than trajectory reconstruction
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Mpath
  role: R package that derives multi-branching developmental trajectories via neighborhood-based
    cell state transitions, landmark designation, aggregation-tree construction, and
    cell fate prediction
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: R
  role: Runtime environment in which Mpath runs and in which DEG derivation (ANOVA,
    Tukey's test, Benjamini-Hochberg adjustment) and trajectory comparisons are scripted
- name: SCUBA
  role: Benchmark trajectory-inference method compared against Mpath on the DC progenitor
    dataset for detection of the cDC1/cDC2 preDC branching event
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_008
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:daca755f26d1561d3e33467d848af1c138e1076551e0c033f17f296e97890323
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  - SCALA
  - SCUBA
  - nichenetr
  - phateR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# single-cell-trajectory-inference

## Summary

Constructs multi-branching single-cell developmental trajectories using neighborhood-based cell state transitions, as implemented in the Mpath R package, to reveal progenitor progression and fate commitment from scRNA-seq data.

## When to use

Use when scRNA-seq data from a differentiating progenitor population must be ordered into a multi-branching trajectory with landmark states, without a priori temporal ordering of subsets, to detect branching fate decisions.

## When NOT to use

- Developmental processes start from multiple independent progenitor cell types — Mpath does not yet support this (planned future extension)
- Highly heterogeneous datasets in which specific subset information is not known — Mpath depends on signature genes of differentiated cell types and FACS-sorted terminal subsets for fate prediction
- The input is already a trajectory or pseudotime ordering and the goal is only gene-wave detection rather than trajectory reconstruction

## Inputs

- QC-passed scRNA-seq expression matrix (e.g. GEO GSE60781 mouse bone marrow DC progenitors; GSE52529 human myoblasts)
- Signature gene sets of mature/terminal cell types from end-stage contrasts (e.g. Supplementary Data 1: cDC1/cDC2 DEGs; Supplementary Data 4: muscle vs non-muscle DEGs)
- FACS-sorted landmark subset labels for designating landmark clusters

## Outputs

- Multi-branching state transition network as an edge list (landmark_a, landmark_b, transition-cell weight), e.g. transition_network.csv
- Designated landmark states (e.g. 11 clusters / 8 landmark clusters for DC progenitors; 8 landmark states for myoblasts)
- Per-cell branch/fate assignment (e.g. cDC1- vs cDC2-committed preDC branches; muscle vs non-muscle paths)
- Ordered cells along the trajectory revealing sequential waves of gene regulation
- Method/sensitivity comparison tables (e.g. trajectory_method_comparison.csv, mpath_sensitivity_results.csv)

## How to apply

Download the scRNA-seq expression matrix (e.g. mouse bone marrow DC progenitors from GSE60781, or primary human myoblasts from GSE52529) and derive signature genes of the mature/terminal subsets by ANOVA followed by post-hoc Tukey's test with Benjamini-Hochberg P-value adjustment, using end-stage contrasts only. Run Mpath's neighborhood-based cell state transition mapping: partition cells into initial clusters (e.g. 11 clusters with 8 FACS-derived landmark clusters for the DC data), assign each cell to its two nearest landmarks to define candidate edges, and weight each edge by the count of transitional cells. Build the aggregation/minimum-spanning-tree over edge distances -w(e), trimming low-support edges so all landmarks remain connected while retaining the maximum number of cells on edges, yielding a multi-branching state transition network (e.g. MDP to CDP to preDC with a branching event at the early preDC stage). Map individual cell fates by travelling back in time from differentiated subsets using signature genes (e.g. cDC1- vs cDC2-specific genes; MYOG/MYOD1/MEF2C for muscle vs PDGFRA for non-muscle), then reorder cells along the trajectory to detect sequential waves of gene regulation. Robustness should be checked by varying dendrogram cut depth (11-13 clusters) and diversity cut (0.5, 0.6, 0.7) and confirming the branching topology is preserved.

## Related tools

- **Mpath** (R package that derives multi-branching developmental trajectories via neighborhood-based cell state transitions, landmark designation, aggregation-tree construction, and cell fate prediction) — https://github.com/JinmiaoChenLab/Mpath
- **R** (Runtime environment in which Mpath runs and in which DEG derivation (ANOVA, Tukey's test, Benjamini-Hochberg adjustment) and trajectory comparisons are scripted)
- **SCUBA** (Benchmark trajectory-inference method compared against Mpath on the DC progenitor dataset for detection of the cDC1/cDC2 preDC branching event)

## Examples

```
library(Mpath); traj <- Mpath(expr.matrix, landmarks = dc_landmarks, diversity.cut = 0.7); write.csv(traj$edges, "transition_network.csv")
```

## Evaluation signals

- The trimmed transition network forms a connected spanning structure over all landmarks, with each retained edge supported by at least one transitional cell
- The reconstructed trajectory recapitulates the expected MDP-to-CDP-to-preDC cascade with a branching event at the early preDC stage (preDC_c committed to cDC1, preDC_d to cDC2), with the 11-cluster / 8-landmark structure recovered
- Robustness check across diversity cuts 0.5-0.7 and dendrogram cuts of 11-13 clusters yields reproducible landmark sets (9 landmarks at cut 0.7, 7 at cut 0.5) and preserved branching topology
- Fate prediction places cDC1/cDC2 signature-gene-committed cells exclusively on the correct branches, unlike conventional MST which misplaced cDC1- and cDC2-committed preDCs
- Reordering cells along the myoblast trajectory reproduces 8 landmark states, a two-branch muscle/non-muscle split, and 7 sequential waves of transcriptional changes

## Limitations

- Mpath does not yet support development processes starting from multiple independent progenitor cell types (planned future extension)
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown (planned future extension)
- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction
- Demonstrated utility limited to two data sets: mouse bone marrow-derived DC progenitors and primary human myoblasts

## Evidence

- [abstract] Mpath derives multi-branching trajectories using neighborhood-based cell state transitions: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [discussion] Mpath uses count of neighbouring cells to estimate state transitions: "Mpath uses count of neighbouring cells to estimate potential state transition. Via neighborhood-based cell state transition mapping"
- [discussion] Trajectory without a priori temporal ordering knowledge: "without requiring a priori knowledge of the temporal ordering of developmental subsets, and utilizing genes only from end-stage contrasts."
- [supplementary] DEG identification with ANOVA and Tukey's test: "The DEGs were identified using ANOVA followed by post-hoc Tukey's test."
- [readme] README confirms Mpath purpose: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
