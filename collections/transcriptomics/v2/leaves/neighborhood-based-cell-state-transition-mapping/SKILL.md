---
name: neighborhood-based-cell-state-transition-mapping
description: Use when reconstructing multi-branching developmental trajectories from
  scRNA-seq (e.g. FACS-sorted progenitor populations like MDP/CDP/preDC or time-course
  myoblast data) where branching points and cell fates must be inferred without known
  temporal ordering.
when_to_use_negative:
- The developmental process starts from multiple independent progenitor cell types
  - Mpath does not yet support this (planned future extension).
- The dataset is highly heterogeneous with unknown specific subset information - Mpath
  currently relies on signature genes of differentiated cell types and FACS-sorted
  terminal subsets for fate prediction.
- A priori temporal ordering of developmental subsets is already known and a linear
  ordering method suffices - Mpath's value is resolving nonlinear multi-branching
  trajectories without temporal prior knowledge.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0085
tools:
- name: Mpath
  role: R package that performs neighborhood-based cell state transition estimation,
    aggregation tree construction, landmark designation, and cell fate prediction
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: R
  role: Environment for running the Mpath R package pipeline on expression matrices
    and signature gene sets
provenance:
  source_task_ids:
  - task_001
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:b1dd1f3ad72a1c8db78cbe2c4c3a6b50f5efe95106d5c476fcecd5349fb01522
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# neighborhood-based-cell-state-transition-mapping

## Summary

Mpath's core operation: aggregate single-cell RNA-seq cells by counting neighbouring cells to estimate potential state transitions, building a multi-branching aggregation tree that maps progenitor progression and branching events without a priori temporal ordering of developmental subsets.

## When to use

Use when reconstructing multi-branching developmental trajectories from scRNA-seq (e.g. FACS-sorted progenitor populations like MDP/CDP/preDC or time-course myoblast data) where branching points and cell fates must be inferred without known temporal ordering.

## When NOT to use

- The developmental process starts from multiple independent progenitor cell types - Mpath does not yet support this (planned future extension).
- The dataset is highly heterogeneous with unknown specific subset information - Mpath currently relies on signature genes of differentiated cell types and FACS-sorted terminal subsets for fate prediction.
- A priori temporal ordering of developmental subsets is already known and a linear ordering method suffices - Mpath's value is resolving nonlinear multi-branching trajectories without temporal prior knowledge.

## Inputs

- scRNA-seq expression matrix (e.g. GEO GSE60781 mouse bone marrow DC progenitors; GSE52529 primary human myoblasts)
- Signature gene sets from end-stage contrasts (ANOVA + post-hoc Tukey's test, Benjamini-Hochberg adjusted P values)
- Landmark cluster designations from FACS-sorted populations (e.g. MDP, CDP, preDC_a-d)

## Outputs

- Aggregation tree / state transition network over cell clusters
- Landmark cluster assignments (e.g. 8 landmark clusters within an 11-cluster partition)
- Branch-level cell fate assignments (e.g. cDC1- vs cDC2-committed preDC branches; muscle vs non-muscle paths)
- Temporally ordered cells revealing waves of gene regulation

## How to apply

Load the scRNA-seq expression matrix (e.g. GSE60781 mouse bone marrow DC progenitors or GSE52529 human myoblasts) into R and reduce the feature space to signature genes derived from end-stage contrasts via ANOVA followed by post-hoc Tukey's test with Benjamini-Hochberg P-value adjustment. Run Mpath to partition cells into initial clusters (e.g. a dendrogram cut yielding 11 clusters), then designate landmark clusters corresponding to known FACS-sorted populations (8 landmark clusters for the DC data). Estimate potential state transitions using the count of neighbouring cells between clusters, and build the aggregation tree to map the multi-branching trajectory (e.g. MDP -> CDP -> preDC with a preDC-stage branch). Predict individual cell fates by travelling back in time from differentiated subsets using their signature genes, then reorder cells along the trajectory to detect sequential temporal waves of gene regulation. Assess robustness by repeating across diversity cuts (0.5-0.7) and dendrogram cut depths (11-13 clusters) and confirming the landmark sets and branching topology are preserved.

## Related tools

- **Mpath** (R package that performs neighborhood-based cell state transition estimation, aggregation tree construction, landmark designation, and cell fate prediction) — https://github.com/JinmiaoChenLab/Mpath
- **R** (Environment for running the Mpath R package pipeline on expression matrices and signature gene sets)

## Examples

```
Rscript mpath_dc_progenitor_pipeline.R  # load GSE60781 matrix, run Mpath at diversity cut 0.7, 11-cluster dendrogram cut
```

## Evaluation signals

- The trajectory recapitulates the expected MDP-to-CDP-to-preDC progression with a branching event at the early preDC stage, and the designated cluster structure (11 clusters, 8 landmark clusters) is recovered.
- The preDC branch spanning preDC_a, preDC_b, preDC_c is exclusively committed to cDC2, and the branch spanning preDC_a and preDC_d is exclusively committed to cDC1.
- Robustness check: repeating at diversity cuts 0.5-0.7 and dendrogram cuts of 11-13 clusters yields reproducible landmark sets (9 landmarks at cut 0.7, 7 at cut 0.5) and preserved branching topology.
- For the myoblast application, the muscle path upregulates MYOG, MYOD1 and MEF2C while the non-muscle path upregulates PDGFRA, and 8 landmark states with 7 ordered temporal waves are recovered.
- Output tables (e.g. mpath_sensitivity_results.csv) tabulate landmark counts, landmark identities and topology-agreement status per parameter combination.

## Limitations

- Mpath does not yet support developmental processes starting from multiple independent progenitor cell types (planned future extension).
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown (planned future extension).
- Fate prediction depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets.
- Demonstrated utility in the study is limited to two data sets: mouse bone marrow-derived DC progenitors and primary human myoblasts.

## Evidence

- [abstract] Mpath derives multi-branching trajectories from neighborhood transitions: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [discussion] Neighbour counts estimate state transitions: "Mpath uses count of neighbouring cells to estimate potential state transition."
- [discussion] End-stage contrast genes without temporal prior: "without requiring a priori knowledge of the temporal ordering of developmental subsets, and utilizing genes only from end-stage contrasts."
- [supplementary] DEG identification method: "The DEGs were identified using ANOVA followed by post-hoc Tukey's test."
- [readme] Mpath purpose from repository README: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
