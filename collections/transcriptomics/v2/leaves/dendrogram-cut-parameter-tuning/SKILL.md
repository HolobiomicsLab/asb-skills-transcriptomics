---
name: dendrogram-cut-parameter-tuning
description: Use when a clustering-based trajectory method (e.g. Mpath) requires choosing
  a dendrogram cut depth and a diversity/aggregation cut, and you must confirm that
  downstream landmarks, branches, or networks are robust to those parameter choices.
when_to_use_negative:
- The pipeline's conclusions depend on a parameterization that changes qualitatively
  across cuts — sensitivity analysis alone will not resolve this and the clustering
  definition must be re-specified.
- The method does not involve hierarchical clustering or a cut-parameter at all (e.g.
  data already summarized to fixed cell states) — there is nothing to tune.
- Development processes start from multiple independent progenitor cell types or subsets
  are unknown in highly heterogeneous data — Mpath does not yet support these settings,
  so tuning its cut parameters is moot.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Mpath
  role: algorithm whose landmark clusters and neighborhood-based state transition
    networks are re-derived across dendrogram cut depths (11-13 clusters) and diversity
    cuts (0.5-0.7) to test robustness
  repo: https://github.com/JinmiaoChenLab/Mpath
- name: R
  role: environment in which the Mpath R package is run and parameter-sweep results
    are tabulated
  repo: https://github.com/JinmiaoChenLab/Mpath
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/ncomms11988
    title: Mpath maps multi-branching single-cell trajectories revealing progenitor
      cell progression during development
  source_content_hash: sha256:03a4324cfdb325250ed8e5f7b8f1dd326c99aae7454858244a32e1916e263840
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JinmiaoChenLab/Mpath
  tools:
  - Mpath
  - R
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# dendrogram-cut-parameter-tuning

## Summary

A sensitivity-analysis skill for single-cell clustering pipelines: re-run the clustering/trajectory workflow across multiple dendrogram cut depths and diversity cut thresholds (e.g. 11-13 clusters and diversity cuts 0.5-0.7 in Mpath) to verify that derived landmarks and state transition networks are invariant to these arbitrary parameters.

## When to use

Use when a clustering-based trajectory method (e.g. Mpath) requires choosing a dendrogram cut depth and a diversity/aggregation cut, and you must confirm that downstream landmarks, branches, or networks are robust to those parameter choices.

## When NOT to use

- The pipeline's conclusions depend on a parameterization that changes qualitatively across cuts — sensitivity analysis alone will not resolve this and the clustering definition must be re-specified.
- The method does not involve hierarchical clustering or a cut-parameter at all (e.g. data already summarized to fixed cell states) — there is nothing to tune.
- Development processes start from multiple independent progenitor cell types or subsets are unknown in highly heterogeneous data — Mpath does not yet support these settings, so tuning its cut parameters is moot.

## Inputs

- single-cell RNA-seq expression matrix (mouse DC progenitor data set)
- end-stage signature gene sets of differentiated cell subsets
- initial cluster dendrogram supporting cuts at multiple depths (11-13 clusters)
- Mpath R package (JinmiaoChenLab/Mpath)

## Outputs

- landmark cell sets per parameter combination (diversity cut x dendrogram depth)
- state transition networks per parameter combination
- sensitivity table of landmark counts, identities and topology-agreement status (mpath_sensitivity_results.csv)

## How to apply

Load the expression matrix and end-stage signature gene sets into R, then run the Mpath pipeline at diversity cut 0.7 with dendrogram cuts yielding 11, 12, and 13 initial clusters, recording landmark cells and the neighborhood-based state transition network for each setting. Repeat the runs at diversity cuts 0.6 and 0.5 across the same 11-13 cluster dendrogram depths. Compare landmark sets across all parameter combinations (the paper found 9 landmarks at cut 0.7 and 7 at cut 0.5) and check that the network topology — including the MDP-CDP-preDC cascade and the preDC-stage split into cDC1- and cDC2-committed branches — is preserved. Tabulate landmark counts, identities and topology-agreement status per combination and write results (e.g. mpath_sensitivity_results.csv); accept the run only if landmark sets and branching topology are reproducible across settings.

## Related tools

- **Mpath** (algorithm whose landmark clusters and neighborhood-based state transition networks are re-derived across dendrogram cut depths (11-13 clusters) and diversity cuts (0.5-0.7) to test robustness) — https://github.com/JinmiaoChenLab/Mpath
- **R** (environment in which the Mpath R package is run and parameter-sweep results are tabulated) — https://github.com/JinmiaoChenLab/Mpath

## Examples

```
Rscript mpath_sensitivity.R --diversity_cuts 0.7,0.6,0.5 --dendrogram_clusters 11,12,13 --out mpath_sensitivity_results.csv
```

## Evaluation signals

- Landmark set sizes match reported values: 9 landmarks at diversity cut 0.7 and 7 landmarks at diversity cut 0.5.
- Landmark identities are reproducible across all diversity cut (0.5-0.7) x dendrogram depth (11-13 cluster) combinations.
- The state transition network preserves the MDP-CDP-preDC cascade at every parameter combination.
- The branching topology — the preDC-stage split into cDC1- and cDC2-committed branches — is preserved across all settings.
- The sensitivity table (mpath_sensitivity_results.csv) is complete, with a topology-agreement status recorded for every parameter combination.

## Limitations

- Mpath is not yet extended to development processes starting from multiple independent progenitor cell types (planned future extension).
- Mpath is not yet extended to highly heterogeneous single-cell data sets in which specific subset information is not known (planned future extension).
- Mpath depends on signature genes of differentiated cell types (end-stage contrasts) and FACS-sorted terminal subsets for fate prediction, so the robustness conclusion is conditional on those inputs.

## Evidence

- [results] Mpath is robust: repeating the analysis across dendrogram cut depths (11-13 clusters) and diversity cuts (0.5-0.7) reproduced similar landmark sets (up to nine landmarks at cut 0.7, seven at cut 0.5) and similar state transition networks preserving the MDP-CDP-preDC cascade and the preDC branching event.: [evidence span withheld — anchor status: not_found]
- [abstract] Mpath derives trajectories via neighborhood-based transitions: "Here we present Mpath, an algorithm that derives multi-branching developmental trajectories using neighborhood-based cell state transitions."
- [abstract] The trajectories detect the branching event at the preDC stage into cDC1/cDC2-committed subsets: "The Mpath-generated trajectories detect a branching event at the preDC stage revealing preDC subsets that are exclusively committed to cDC1 or cDC2 lineages."
- [discussion] Mpath is currently limited to a single progenitor type and known subsets: "In future, we will extend the Mpath algorithm for development processes starting from multiple independent progenitor cell types"
- [other] Repository and R package documentation location: "Mpath reference manual provides detailed documentation of Mpath R package."
