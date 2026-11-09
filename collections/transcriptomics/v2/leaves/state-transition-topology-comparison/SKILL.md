---
name: state-transition-topology-comparison
description: Use when a reconstructed cell-state transition network (e.g. Mpath's
  MDP-CDP-preDC cascade) must be shown robust to clustering or graph-pruning parameter
  changes before downstream biological interpretation.
when_to_use_negative:
- Development processes that start from multiple independent progenitor cell types
  — Mpath does not yet support these (planned future extension).
- Highly heterogeneous single-cell data sets in which specific subset information
  is unknown — Mpath is not yet extended to such data.
- Analyses lacking signature genes of differentiated cell types or FACS-sorted terminal
  subsets, since Mpath's fate prediction depends on end-stage contrasts.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Mpath
  role: Runs the landmark-cluster derivation and neighborhood-based state transition
    network reconstruction that is compared across diversity cuts and dendrogram cut
    depths.
  repo: JinmiaoChenLab/Mpath
- name: R
  role: Environment for executing the Mpath R package runs and writing the mpath_sensitivity_results.csv
    comparison table.
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:de265313bbbf8a6e43a376dab1d2816473af14a65e3dd9a7710ee6392c88350c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# state-transition-topology-comparison

## Summary

A sensitivity-analysis skill that tests whether Mpath's landmark clusters and neighborhood-based state transition network remain stable across parameter settings — diversity cuts (0.5, 0.6, 0.7) and dendrogram cut depths yielding 11–13 initial clusters — on the mouse DC progenitor single-cell RNA-seq data set.

## When to use

Use when a reconstructed cell-state transition network (e.g. Mpath's MDP-CDP-preDC cascade) must be shown robust to clustering or graph-pruning parameter changes before downstream biological interpretation.

## When NOT to use

- Development processes that start from multiple independent progenitor cell types — Mpath does not yet support these (planned future extension).
- Highly heterogeneous single-cell data sets in which specific subset information is unknown — Mpath is not yet extended to such data.
- Analyses lacking signature genes of differentiated cell types or FACS-sorted terminal subsets, since Mpath's fate prediction depends on end-stage contrasts.

## Inputs

- mouse DC progenitor single-cell RNA-seq expression matrix
- end-stage signature gene sets of differentiated cell types (used by Mpath to predict individual cell fate)
- Mpath parameter grid: diversity cuts 0.5/0.6/0.7 and dendrogram cuts yielding 11/12/13 clusters

## Outputs

- landmark sets (counts and identities) per parameter combination
- state transition network per parameter combination
- topology-agreement status per parameter combination
- mpath_sensitivity_results.csv tabulating landmark counts, identities and topology agreement

## How to apply

Load the mouse DC progenitor expression matrix and end-stage signature gene sets into R and run the Mpath pipeline, which derives trajectories using neighborhood-based cell state transitions. Sweep the diversity cut parameter over 0.5, 0.6 and 0.7, and at each cut repeat with dendrogram cuts producing 11, 12 and 13 clusters, recording landmark cells and the state transition network per setting. Compare landmark sets across settings, confirming the reported invariance of 9 landmarks at diversity cut 0.7 and 7 landmarks at diversity cut 0.5. Compare the networks across settings to verify the branching topology — including the preDC-stage split into cDC1- and cDC2-committed branches — is preserved. Tabulate landmark counts, landmark identities and topology-agreement status per parameter combination and write to mpath_sensitivity_results.csv.

## Related tools

- **Mpath** (Runs the landmark-cluster derivation and neighborhood-based state transition network reconstruction that is compared across diversity cuts and dendrogram cut depths.) — JinmiaoChenLab/Mpath
- **R** (Environment for executing the Mpath R package runs and writing the mpath_sensitivity_results.csv comparison table.)

## Examples

```
Rscript mpath_sensitivity.R --diversity_cuts 0.5,0.6,0.7 --dendrogram_clusters 11,12,13 --output mpath_sensitivity_results.csv
```

## Evaluation signals

- Landmark count at diversity cut 0.7 equals 9 across all 11–13 cluster dendrogram cuts.
- Landmark count at diversity cut 0.5 equals 7 across all 11–13 cluster dendrogram cuts.
- Landmark identities are reproducible (similar landmark sets) across all parameter combinations.
- The MDP-CDP-preDC cascade and the preDC branching event (split into cDC1- and cDC2-committed branches) are preserved in every state transition network.
- mpath_sensitivity_results.csv contains one row per parameter combination with a topology-agreement status column, and all rows pass the invariance check.

## Limitations

- Mpath does not yet support development processes starting from multiple independent progenitor cell types (planned future extension).
- Mpath is not yet extended to highly heterogeneous single-cell data sets where specific subset information is unknown.
- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction.
- Demonstrated utility is limited to two data sets: mouse bone marrow-derived DC progenitors and primary human myoblasts.

## Evidence

- [abstract] Mpath derives trajectories from neighborhood-based transitions: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [abstract] PreDC branching event detected by Mpath trajectories: "The Mpath-generated trajectories detect a branching event at the preDC stage revealing preDC subsets that are exclusively committed to cDC1 or cDC2 lineages."
- [discussion] Subset commitment and branching timing in DC progenitor study: "Our study revealed early subset-commitment emerged from late CDP stage onwards and a branching event occurred at the preDC stage."
- [discussion] Fate prediction depends on signature genes of terminal subsets: "Mpath is able to predict individual cell's fate using signature genes of differentiated cell types."
- [readme] Mpath R package availability: "Mpath: an algorithm for constructing multi-branching cell lineages from single-cell data"
