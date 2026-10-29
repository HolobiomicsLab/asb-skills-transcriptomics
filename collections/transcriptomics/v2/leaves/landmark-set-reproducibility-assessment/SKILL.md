---
name: landmark-set-reproducibility-assessment
description: Use when an Mpath trajectory analysis on single-cell RNA-seq data (e.g.
  mouse DC progenitors) needs its landmark clusters and branching topology checked
  for robustness to clustering/cut parameters before downstream interpretation.
when_to_use_negative:
- Trajectories expected to start from multiple independent progenitor cell types —
  Mpath does not yet support development processes starting from multiple independent
  progenitor cell types
- Highly heterogeneous single-cell data sets where specific subset information is
  unknown — Mpath is not yet extended to such data
- Analyses lacking signature genes of differentiated cell types and FACS-sorted terminal
  subsets — Mpath depends on end-stage contrasts for fate prediction
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3170
tools:
- name: Mpath
  role: Algorithm whose landmark designation and neighborhood-based state transition
    network are re-run across the diversity-cut and dendrogram-cut parameter grid
  repo: JinmiaoChenLab/Mpath
- name: R
  role: Language in which the Mpath pipeline is executed (Mpath R package) and in
    which the sensitivity comparison table is generated
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:a697657b19330cb3a56e195c0db9023eb8938bab6ef6a4b82fd8f33bcf951407
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# landmark-set-reproducibility-assessment

## Summary

A sensitivity-analysis skill for the Mpath trajectory pipeline that re-runs landmark-cluster designation and neighborhood-based state transition network construction across diversity cuts (0.5, 0.6, 0.7) and dendrogram cut depths (11-13 initial clusters) to verify the reconstructed DC progenitor cascade is parameter-invariant.

## When to use

Use when an Mpath trajectory analysis on single-cell RNA-seq data (e.g. mouse DC progenitors) needs its landmark clusters and branching topology checked for robustness to clustering/cut parameters before downstream interpretation.

## When NOT to use

- Trajectories expected to start from multiple independent progenitor cell types — Mpath does not yet support development processes starting from multiple independent progenitor cell types
- Highly heterogeneous single-cell data sets where specific subset information is unknown — Mpath is not yet extended to such data
- Analyses lacking signature genes of differentiated cell types and FACS-sorted terminal subsets — Mpath depends on end-stage contrasts for fate prediction

## Inputs

- Mouse DC progenitor single-cell RNA-seq expression matrix
- End-stage signature gene sets of differentiated cell types (used for fate prediction by travelling back in time from terminal subsets)
- Mpath parameter grid: diversity cuts (0.5, 0.6, 0.7) and dendrogram cut depths yielding 11-13 initial clusters

## Outputs

- Per-parameter-setting landmark sets with counts (9 landmarks at diversity cut 0.7; 7 at diversity cut 0.5)
- Neighborhood-based state transition networks for each parameter combination
- Tabulated sensitivity results (landmark counts, landmark identities, topology-agreement status, e.g. mpath_sensitivity_results.csv)

## How to apply

Load the mouse DC progenitor single-cell RNA-seq expression matrix and end-stage signature gene sets into R, then run Mpath at diversity cut 0.7 with dendrogram cuts producing 11, 12 and 13 clusters, recording the landmark cells and the neighborhood-based state transition network for each setting. Repeat the runs at diversity cuts 0.6 and 0.5 across the same 11-13 cluster dendrogram depths. Compare landmark sets across all parameter combinations, confirming nine landmarks at diversity cut 0.7 and seven at diversity cut 0.5. Then compare the state transition networks to verify that the branching topology — the MDP-CDP-preDC cascade and the preDC-stage split into cDC1- and cDC2-committed branches — is preserved. Tabulate landmark counts, landmark identities and topology-agreement status per parameter combination (e.g. write to mpath_sensitivity_results.csv). The rationale is that Mpath derives trajectories from neighborhood-based cell state transitions, so the reported topology should be invariant to reasonable cut choices.

## Related tools

- **Mpath** (Algorithm whose landmark designation and neighborhood-based state transition network are re-run across the diversity-cut and dendrogram-cut parameter grid) — JinmiaoChenLab/Mpath
- **R** (Language in which the Mpath pipeline is executed (Mpath R package) and in which the sensitivity comparison table is generated)

## Examples

```
Rscript mpath_sensitivity.R --diversity-cuts 0.5,0.6,0.7 --dendrogram-cuts 11,12,13 --output mpath_sensitivity_results.csv
```

## Evaluation signals

- Landmark count invariance: exactly 9 landmarks at diversity cut 0.7 and 7 landmarks at diversity cut 0.5 across all 11-13 cluster dendrogram cuts
- Reproducible landmark identities across all diversity-cut / dendrogram-cut combinations
- Preserved branching topology: the MDP-CDP-preDC cascade and the preDC-stage split into cDC1- and cDC2-committed branches appear in every state transition network
- All nine parameter combinations (3 diversity cuts x 3 dendrogram depths) complete and are recorded in the output table
- Topology-agreement status per parameter combination supports the reported invariance claim in mpath_sensitivity_results.csv

## Limitations

- Demonstrated utility limited to two data sets: mouse bone marrow-derived DC progenitors and primary human myoblasts
- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction
- Mpath does not yet support development processes starting from multiple independent progenitor cell types (planned future extension)
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown (planned future extension)

## Evidence

- [other] repeating the analysis across dendrogram cut depths (11-13 clusters) and diversity cuts (0.5-0.7) reproduced similar landmark sets: [evidence span withheld — anchor status: not_found]
- [abstract] Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions.: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [discussion] predict individual cell's fate using signature genes of differentiated cell types.: "Mpath is able to predict individual cell's fate using signature genes of differentiated cell types."
- [supplementary] Mpath reference manual provides detailed documentation of Mpath R package.: "Mpath reference manual provides detailed documentation of Mpath R package."
