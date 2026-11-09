---
name: bootstrapping-robustness-assessment
description: Use when TF activities inferred with FateCompass's linear model of gene
  regulation must be robustness-tested across bootstrap resamples before calling differentially
  active TFs on differentiation trajectories.
when_to_use_negative:
- 'Upstream steps have not been run: fate-probability distributions (differentiation_trajectories.m)
  and TF activity estimates (tf_activities.m) are missing, while the README fixes
  bootstrapping.m as step 3 of 4.'
- No transcription-factor activity question is at stake, e.g., generic bootstrap validation
  of clustering or differential expression unrelated to the binding-site-matrix-based
  gene-regulation model.
- Only raw, unpre-processed counts are available without 'clusters' annotations or
  a TF binding-site matrix; FateCompass expects a pre-processed adata (e.g., via Scanpy)
  plus the binding-site matrix.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: FateCompass
  role: Pipeline whose linear model of gene regulation yields the TF activity estimates
    that bootstrapping.m stress-tests; reproducibility scripts are in reproducibility/scripts
  repo: github.com/sarajimenez/fatecompass
- name: MATLAB
  role: Implementation language of the reproducibility scripts, including bootstrapping.m
    executed as step 3 of the documented run order
- name: Scanpy
  role: Suggested pre-processing of the adata input (cell-type annotations, expressed-gene
    rows of the binding-site matrix) before the trajectory and activity steps
  repo: https://scanpy.readthedocs.io/en/stable/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:0b9c01d6b0b5b50abde115a97825a6ae5ff80bd89b21fbd9306739dac0c0a98b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/sarajimenez/fatecompass
  tools:
  - FateCompass
  - MATLAB
  - Python
  - Scanpy
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# bootstrapping-robustness-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Bootstrap-resampling step of the FateCompass workflow: it re-estimates transcription factor activities obtained from the linear model of gene regulation across resamples to assess robustness before the differential motif activity analysis calls differentially active TFs.

## When to use

Use when TF activities inferred with FateCompass's linear model of gene regulation must be robustness-tested across bootstrap resamples before calling differentially active TFs on differentiation trajectories.

## When NOT to use

- Upstream steps have not been run: fate-probability distributions (differentiation_trajectories.m) and TF activity estimates (tf_activities.m) are missing, while the README fixes bootstrapping.m as step 3 of 4.
- No transcription-factor activity question is at stake, e.g., generic bootstrap validation of clustering or differential expression unrelated to the binding-site-matrix-based gene-regulation model.
- Only raw, unpre-processed counts are available without 'clusters' annotations or a TF binding-site matrix; FateCompass expects a pre-processed adata (e.g., via Scanpy) plus the binding-site matrix.

## Inputs

- Pre-processed adata object (cell-type/'clusters' annotated; e.g., Veres stage 5, 25,299 cells) with fate-probability distributions from differentiation_trajectories.m
- TF binding-site matrix (columns = TFs/motifs, rows = expressed genes)
- TF activity estimates from tf_activities.m (linear model of gene regulation output)

## Outputs

- Bootstrap-resampled TF activity estimates (activity distributions across resamples) with per-TF robustness summaries
- Robustness-supported input for differential_motif_activity_analysis.m (Z-score, activity-change, activity-expression-correlation filters)

## How to apply

Run bootstrapping.m from the reproducibility/scripts folder (implemented in Matlab) strictly as step 3 of the documented order: after differentiation_trajectories.m has built the Markov-chain fate-probability distributions and tf_activities.m has solved the linear model of gene regulation (Balwierz et al. 2014) on the TF binding-site matrix (columns = TFs/motifs, rows = expressed genes). The script resamples and re-solves the activity model to quantify how stable each TF's activity estimate is across bootstrap resamples. Feed this robustness information into differential_motif_activity_analysis.m, which calls differentially active TFs using three criteria: Z-score significance of the TF to explain the variability of the linear model, change of regulatory activity throughout the fate decision, and dynamical correlation of TF activity with the TF mRNA level. In the Veres et al. stage-5 run (25,299 annotated cells; differentiation-potential-driven trajectories from NKX6-1+ progenitors to SC-α, SC-β, SC-EC), bootstrap support backed the call of 126 differentially active TFs, including 20 SC-β-specific and 25 SC-EC-specific factors. Compare the final TF set against the reference results in the FateCompass Shiny App.

## Related tools

- **FateCompass** (Pipeline whose linear model of gene regulation yields the TF activity estimates that bootstrapping.m stress-tests; reproducibility scripts are in reproducibility/scripts) — github.com/sarajimenez/fatecompass
- **MATLAB** (Implementation language of the reproducibility scripts, including bootstrapping.m executed as step 3 of the documented run order)
- **Scanpy** (Suggested pre-processing of the adata input (cell-type annotations, expressed-gene rows of the binding-site matrix) before the trajectory and activity steps) — https://scanpy.readthedocs.io/en/stable/

## Examples

```
matlab -batch "run('reproducibility/scripts/bootstrapping.m')"
```

## Evaluation signals

- Script order honored: bootstrapping.m runs third — after differentiation_trajectories.m and tf_activities.m, and before differential_motif_activity_analysis.m.
- Resampled activity estimates cover the same TF columns as the binding-site matrix and are conditioned on the fate-probability distributions toward the terminal states (SC-α, SC-β, SC-EC).
- Robustness is reflected downstream: called TFs pass all three criteria (Z-score significance in the linear model, activity change through the fate decision, dynamical activity–mRNA correlation).
- Recovered counts are consistent with the reference results: 126 differentially active TFs for Veres stage 5, including 20 SC-β-specific and 25 SC-EC-specific factors, matchable in the FateCompass Shiny App (https://jimenezs.shinyapps.io/FateCompass_results/).

## Limitations

- The binding-site matrix relies on computationally predicted regulatory sites, which biases the gene-regulatory network structure; only promoter binding sites are considered, so distal/long-range interactions are missed.
- Pipeline input parameters must be hand tuned, although detailed tutorials are provided to guide parameter setup.
- TF activities are inferred from gene expression rather than measured directly, and identified putative driver factors remain to be experimentally validated.

## Evidence

- [readme] 1. differentiation_trajectories.m 2. tf_activities.m 3. bootstrapping.m 4. differential_motif_activity_analysis.m: "1. differentiation_trajectories.m 2. tf_activities.m 3. bootstrapping.m 4. differential_motif_activity_analysis.m"
- [readme] Scripts to reproduce the results are available in the [reproducibility](reproducibility/scripts/) folder and are implemented in Matlab.: "Scripts to reproduce the results are available in the [reproducibility](reproducibility/scripts/) folder and are implemented in Matlab."
- [readme] a pre-processed data object (adata), with annotations of at least cell types (‘clusters’): "a pre-processed data object (adata), with annotations of at least cell types (‘clusters’)"
- [abstract] By combining a probabilistic framework with RNA velocities or differentiation potential, we estimate transition probabilities: "By combining a probabilistic framework with RNA velocities or differentiation potential, we estimate transition probabilities"
- [discussion] considers the significance of the TF to explain the variability of the linear model of gene regulation: "considers the significance of the TF to explain the variability of the linear model of gene regulation"
- [discussion] the dynamical correlation of the TF activity with the TF mRNA level: "the dynamical correlation of the TF activity with the TF mRNA level"
