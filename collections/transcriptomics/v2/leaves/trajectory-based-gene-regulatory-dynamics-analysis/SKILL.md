---
name: trajectory-based-gene-regulatory-dynamics-analysis
description: Use when scRNA-seq differentiation data (pre-processed adata with cluster
  annotations and optionally RNA velocity) plus a TF binding-site matrix must be converted
  into time- and lineage-specific TF activity estimates along cell-fate decision trajectories.
when_to_use_negative:
- The goal is bulk RNA-seq motif-activity inference with no single-cell trajectory
  structure — ISMARA or DOROTHEA2 already cover that regime and FateCompass extends
  rather than replaces them
- Only a static feature/cluster table is available without differentiation potential,
  RNA velocity, or progenitor-to-fate structure from which transition probabilities
  can be estimated
- Regulatory interactions are known to be dominated by distal enhancers, since FateCompass
  considers only TF-promoter binding sites in the binding-site matrix
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0749
tools:
- name: FateCompass
  role: 'Main pipeline: trajectory estimation, linear model of gene regulation for
    TF activities, and differential TF activity analysis; core module in fatecompass.py
    (Python), reproducibility scripts in Matlab'
  repo: https://github.com/sarajimenez/fatecompass
- name: dyngen
  role: In silico simulation engine used to generate ground-truth data for validating
    FateCompass predictions
  repo: https://github.com/dynverse/dyngen
- name: dropEst
  role: Droplet-based scRNA-seq preprocessing pipeline for building count matrices
    used as upstream input
  repo: https://github.com/kharchenkolab/dropEst
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:44b7a0d8acb699fcff638124b730a045829651dc4953528a4ea5defcbf76d86f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  repo_url: https://github.com/sarajimenez/fatecompass
  tools:
  - FateCompass
  - Python
  - dropEst
  - dyngen
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# trajectory-based-gene-regulatory-dynamics-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates time- and fate-specific transcription factor activities along stochastic differentiation trajectories by modeling gene expression as a linear combination of regulatory sites and TF activities, then classifies lineage-specific TFs using Z-score, regulatory-activity-change, and dynamical-correlation criteria (FateCompass).

## When to use

Use when scRNA-seq differentiation data (pre-processed adata with cluster annotations and optionally RNA velocity) plus a TF binding-site matrix must be converted into time- and lineage-specific TF activity estimates along cell-fate decision trajectories.

## When NOT to use

- The goal is bulk RNA-seq motif-activity inference with no single-cell trajectory structure — ISMARA or DOROTHEA2 already cover that regime and FateCompass extends rather than replaces them
- Only a static feature/cluster table is available without differentiation potential, RNA velocity, or progenitor-to-fate structure from which transition probabilities can be estimated
- Regulatory interactions are known to be dominated by distal enhancers, since FateCompass considers only TF-promoter binding sites in the binding-site matrix

## Inputs

- Pre-processed single-cell data object (adata) with 'clusters' annotations and optionally an RNA velocity field (Scanpy/scVelo pre-processing suggested)
- TF binding-site matrix with columns = TFs/motifs and rows = expressed genes
- Stochastic differentiation trajectories from the trajectory-estimation step (e.g., differentiation_trajectories.m output)

## Outputs

- Time- and fate-specific TF activity estimates across the trajectory ensemble
- Bootstrapped variability estimates of TF activities
- differential_tf_activity_results.csv classifying lineage-specific TFs (e.g., 86 TFs: 22 both-fate, 38 beta-specific, 25 alpha-specific; Bcl11a Z = 4.1192)

## How to apply

First build stochastic differentiation trajectories: combine a probabilistic framework with RNA velocities or a differentiation-potential gradient to estimate transition probabilities, running a discrete Markov process on a nearest-neighbor network. Next run the tf_activities step to model observed gene expression as a linear combination of TF-promoter binding sites and TF activities, estimating time- and fate-specific activities. Quantify variability of the estimated activities across the trajectory ensemble with bootstrapping. Finally apply the differential TF activity analysis with three criteria: (i) Z-score significance of the TF for explaining variability of the linear model, (ii) change of regulatory activity throughout the cell-fate decision, and (iii) dynamical correlation of TF activity with the TF mRNA level. In the mouse E15.5 pancreatic application this classified 86 lineage-specific TFs (22 shared by alpha and beta fates, 38 beta-specific, 25 alpha-specific), with Bcl11a achieving the highest Z score (4.1192).

## Related tools

- **FateCompass** (Main pipeline: trajectory estimation, linear model of gene regulation for TF activities, and differential TF activity analysis; core module in fatecompass.py (Python), reproducibility scripts in Matlab) — https://github.com/sarajimenez/fatecompass
- **dyngen** (In silico simulation engine used to generate ground-truth data for validating FateCompass predictions) — https://github.com/dynverse/dyngen
- **dropEst** (Droplet-based scRNA-seq preprocessing pipeline for building count matrices used as upstream input) — https://github.com/kharchenkolab/dropEst

## Examples

```
matlab -batch "run('tf_activities.m'); run('bootstrapping.m'); run('differential_motif_activity_analysis.m')"  # after differentiation_trajectories.m, per the fatecompass reproducibility scripts
```

## Evaluation signals

- Differential analysis counts match 86 lineage-specific TFs: 22 specific for both alpha and beta fates, 38 beta-specific, and 25 alpha-specific
- Bcl11a's Z score for explaining variance of the linear model of gene regulation equals 4.1192
- Pipeline scripts run in the documented order: differentiation_trajectories.m, tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m
- Binding-site matrix dimensions are consistent: columns equal TFs/motifs, rows equal expressed genes
- Validation against a Dyngen-generated in silico ground-truth system recovers the expected lineage-specific regulators

## Limitations

- TF activities are estimated using binding sites only in promoter regions; long-range and distal regulatory interactions are not considered, biasing the inferred gene-regulatory network structure
- The pipeline depends on input parameters that must be hand tuned, and differentiation-potential mode requires prior biological knowledge that is hard to supply for some systems (e.g., cancer)
- Common dimensionality-reduction embeddings such as UMAP lack biological interpretability, so trajectories are built on a nearest-neighbor graph in reduced space with care
- Generated hypotheses about putative driver factors require further experimental validation

## Evidence
