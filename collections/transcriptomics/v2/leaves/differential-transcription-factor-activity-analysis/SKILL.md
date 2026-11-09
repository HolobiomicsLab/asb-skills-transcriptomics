---
name: differential-transcription-factor-activity-analysis
description: Use when single-cell RNA-seq differentiation data (adata with cluster
  annotations and RNA velocity or differentiation potential) must be analyzed to identify
  time- and fate-specific transcription factor regulators along stochastic trajectories.
when_to_use_negative:
- Input is bulk RNA-seq rather than single-cell transcriptomics — ISMARA/DOROTHEA
  already handle bulk data and FateCompass targets scRNA-seq trajectories
- The system lacks robust RNA velocity profiles AND no prior biological knowledge
  is available to define a differentiation potential gradient (article notes this
  is difficult in cases such as cancer studies)
- The goal is exact reconstruction of gene-regulatory interactions, including distal
  regulatory sites — FateCompass considers only TF-promoter binding sites
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: FateCompass
  role: 'Implements the full pipeline: probabilistic trajectory estimation, linear
    model of gene regulation, bootstrapping, and differential TF activity analysis
    (fatecompass.py in Python; reproducibility scripts in Matlab)'
  repo: https://github.com/sarajimenez/fatecompass
- name: Scanpy
  role: Suggested for basic pre-processing of the adata object and preparation of
    the TF binding-site matrix
- name: MATLAB
  role: Runs the reproducibility scripts differentiation_trajectories.m, tf_activities.m,
    bootstrapping.m, differential_motif_activity_analysis.m in order
- name: Dyngen
  role: Simulation engine generating in silico ground-truth data used to validate
    the differential TF activity predictions
  repo: https://github.com/dynverse/dyngen
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:2fada12b4b2e0db769b73120cd25a3d676d67a94a4545f454ccc4753fd672fdb
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
  - Dyngen
  - FateCompass
  - MATLAB
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# differential-transcription-factor-activity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

FateCompass-based differential TF activity analysis that classifies lineage-specific transcription factors during cell-fate decisions using three statistical criteria (Z-score significance, change of regulatory activity, and dynamical correlation of TF activity with TF mRNA level), applied after inferring TF activities with a linear model of gene regulation.

## When to use

Use when single-cell RNA-seq differentiation data (adata with cluster annotations and RNA velocity or differentiation potential) must be analyzed to identify time- and fate-specific transcription factor regulators along stochastic trajectories.

## When NOT to use

- Input is bulk RNA-seq rather than single-cell transcriptomics — ISMARA/DOROTHEA already handle bulk data and FateCompass targets scRNA-seq trajectories
- The system lacks robust RNA velocity profiles AND no prior biological knowledge is available to define a differentiation potential gradient (article notes this is difficult in cases such as cancer studies)
- The goal is exact reconstruction of gene-regulatory interactions, including distal regulatory sites — FateCompass considers only TF-promoter binding sites

## Inputs

- Pre-processed adata object with cell-type ('clusters') annotations and optionally the RNA velocity field (e.g. Veres stage-5 adata, 25,299 cells)
- TF binding-site matrix with columns = TFs/motifs and rows = expressed genes
- Stochastic differentiation trajectories with fate-probability distributions per cell (from FateCompass tf_activities/differentiation_trajectories steps)

## Outputs

- Time- and fate-specific TF activity estimates with bootstrapping-based variability measures
- Classified lineage-specific TF sets (e.g. both-fate, beta-specific, alpha-specific counts) in differential_tf_activity_results.csv
- Cross-species TF comparison table with overlap statistics (e.g. beta_tf_comparison.csv)

## How to apply

First run differentiation_trajectories.m to build Markov-chain stochastic trajectories from progenitors toward terminal fates, driven either by RNA velocity (when velocity profiles are robust) or by the gradient of differentiation potential. Then run tf_activities.m to model observed gene expression as a linear combination of regulatory sites (from a TF binding-site matrix with columns = TFs/motifs, rows = expressed genes) and TF activities, extending the ISMARA linear model to single-cell data, followed by bootstrapping.m to quantify variability of the TF activity estimates across the trajectory ensemble. Finally run differential_motif_activity_analysis.m applying three criteria: Z-score significance of the TF to explain variability of the linear model, change of regulatory activity throughout the cell-fate decision, and dynamical correlation of TF activity with TF mRNA level. Validate against reference counts, e.g. 86 lineage-specific TFs (22 both-fate, 38 beta-specific, 25 alpha-specific) with Bcl11a Z score 4.1192 in the mouse E15.5 dataset, and 126 differentially active TFs (including 20 SC-beta-specific and 25 SC-EC-specific) in the Veres stage-5 dataset; write results such as differential_tf_activity_results.csv.

## Related tools

- **FateCompass** (Implements the full pipeline: probabilistic trajectory estimation, linear model of gene regulation, bootstrapping, and differential TF activity analysis (fatecompass.py in Python; reproducibility scripts in Matlab)) — https://github.com/sarajimenez/fatecompass
- **Scanpy** (Suggested for basic pre-processing of the adata object and preparation of the TF binding-site matrix)
- **MATLAB** (Runs the reproducibility scripts differentiation_trajectories.m, tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m in order)
- **Dyngen** (Simulation engine generating in silico ground-truth data used to validate the differential TF activity predictions) — https://github.com/dynverse/dyngen

## Examples

```
matlab -batch "run('differential_motif_activity_analysis.m')"  # run after differentiation_trajectories.m, tf_activities.m and bootstrapping.m in reproducibility/scripts/
```

## Evaluation signals

- Mouse E15.5 pancreatic dataset: differential analysis must recover 86 lineage-specific TFs — 22 specific for both alpha and beta fates, 38 beta-specific, 25 alpha-specific
- Bcl11a must have the highest Z score (4.1192) for explaining the variance of the linear model of gene regulation
- Veres stage-5 (SC-beta) dataset: expect 126 differentially active TFs, including 20 SC-beta-specific and 25 SC-EC-specific factors; cross-check with the FateCompass Shiny App reference
- Mouse vs human beta-specific TF overlap should be ~8% (10 shared factors including Esrrg, Neurod1, Nkx6-1), with mouse-specific Sox9/Meis2 and human-specific SOX4/PTF1A/CDX2 (early) and MAFA/PAX6/GATA4 (late)
- Bootstrapped TF activity estimates should be robust across the trajectory ensemble before differential calling

## Limitations

- TF activities are estimated using binding sites only in promoter regions; long-range/distal regulatory interactions are not considered
- Reliance on computationally predicted regulatory sites in the binding-site matrix introduces a bias on the gene-regulatory network structure
- Pipeline input parameters must be hand tuned (tutorials are provided to guide setup); differentiation-potential mode depends on prior biological knowledge
- Whether identified putative driver factors are involved in endocrine cell differentiation remains untested; hypotheses require further experimental validation

## Evidence
