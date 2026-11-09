---
name: bootstrap-variability-estimation-of-tf-activities
description: Use when TF activities have been inferred along FateCompass stochastic
  trajectories (e.g., mouse E15.5 pancreatic endocrine differentiation) and you need
  per-TF variability estimates before applying Z-score/variability/dynamic-correlation
  lineage-specificity criteria.
when_to_use_negative:
- No stochastic trajectory ensemble exists yet — bootstrap variability of TF activities
  is meaningless without the FateCompass trajectories from differentiation_trajectories.m
- The binding-site matrix is missing or does not have TFs/motifs as columns and expressed
  genes as rows, since the linear model of gene regulation cannot be solved
- You only need static bulk-RNA-seq motif activity inference (e.g., ISMARA on bulk
  data), not dynamic variability across cell-fate decision trajectories
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0204
tools:
- name: FateCompass
  role: Pipeline whose Matlab reproducibility scripts (tf_activities.m, bootstrapping.m,
    differential_motif_activity_analysis.m) implement TF activity estimation and bootstrapping
    across the trajectory ensemble
  repo: github.com/sarajimenez/fatecompass
- name: Python
  role: FateCompass is fully implemented in Python (fatecompass.py); Matlab is used
    specifically for the reproducibility/bootstrapping scripts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:2b02e2e632acd2ac0680132d02316a7111070d27d933f87c45087cc894913673
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
  - Python
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# bootstrap-variability-estimation-of-tf-activities

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the variability of time- and fate-specific transcription factor activities estimated by FateCompass's linear model of gene regulation across an ensemble of stochastic differentiation trajectories, using bootstrapping (bootstrapping.m) so that downstream differential TF activity calls are robust to trajectory sampling.

## When to use

Use when TF activities have been inferred along FateCompass stochastic trajectories (e.g., mouse E15.5 pancreatic endocrine differentiation) and you need per-TF variability estimates before applying Z-score/variability/dynamic-correlation lineage-specificity criteria.

## When NOT to use

- No stochastic trajectory ensemble exists yet — bootstrap variability of TF activities is meaningless without the FateCompass trajectories from differentiation_trajectories.m
- The binding-site matrix is missing or does not have TFs/motifs as columns and expressed genes as rows, since the linear model of gene regulation cannot be solved
- You only need static bulk-RNA-seq motif activity inference (e.g., ISMARA on bulk data), not dynamic variability across cell-fate decision trajectories

## Inputs

- Stochastic differentiation trajectories produced by differentiation_trajectories.m (RNA-velocity- or differentiation-potential-driven ensemble)
- TF binding-site matrix (columns = TFs/motifs, rows = expressed genes)
- Time- and fate-specific TF activity estimates from tf_activities.m (FateCompass, Matlab reproducibility scripts)

## Outputs

- Bootstrap variability estimates of TF activities across the trajectory ensemble
- Variability statistics consumed by differential_motif_activity_analysis.m to classify lineage-specific TFs (e.g., differential_tf_activity_results.csv with per-TF Z scores, e.g., Bcl11a Z = 4.1192)

## How to apply

After running the tf_activities step, which models observed gene expression as a linear combination of TF binding sites (columns = TFs/motifs, rows = expressed genes) and TF activities across the stochastic trajectory ensemble, run the bootstrapping step (bootstrapping.m, Matlab, in reproducibility/scripts) to re-estimate TF activities over resampled trajectories. This yields variability measures of the estimated TF activities across the trajectory ensemble, which feed the differential_motif_activity_analysis.m step together with the Z-score of each TF for explaining the variance of the linear model and the dynamical correlation of TF activity with TF mRNA level. In the E15.5 pancreatic reference analysis this pipeline produced 86 lineage-specific TFs (22 specific to both α and β fates, 38 β-specific, 25 α-specific), with Bcl11a having the highest Z score (4.1192). Order matters: differentiation_trajectories.m → tf_activities.m → bootstrapping.m → differential_motif_activity_analysis.m.

## Related tools

- **FateCompass** (Pipeline whose Matlab reproducibility scripts (tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m) implement TF activity estimation and bootstrapping across the trajectory ensemble) — github.com/sarajimenez/fatecompass
- **Python** (FateCompass is fully implemented in Python (fatecompass.py); Matlab is used specifically for the reproducibility/bootstrapping scripts)

## Examples

```
matlab -batch "run('reproducibility/scripts/tf_activities.m'); run('reproducibility/scripts/bootstrapping.m'); run('reproducibility/scripts/differential_motif_activity_analysis.m')"
```

## Evaluation signals

- Differential TF activity analysis on the E15.5 pancreatic dataset yields 86 lineage-specific TFs: 22 both-fate, 38 β-specific, 25 α-specific
- Bcl11a has the highest Z score of 4.1192 for explaining the variance of the linear model of gene regulation
- Bootstrapped variability estimates are produced for every TF in the binding-site matrix before the differential_motif_activity_analysis step runs
- Scripts were executed in the documented order: differentiation_trajectories.m, tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m
- Results are reproducible with the scripts in the reproducibility/scripts folder and comparable to the FateCompass Shiny App summaries

## Limitations

- TF activities rely on computationally predicted regulatory sites (binding-site matrix) considering only TF-promoter interactions; distal/long-range regulatory interactions are ignored, biasing the gene-regulatory network structure
- The pipeline depends on input parameters that must be hand tuned, which affects the trajectory ensemble and hence bootstrap variability
- Identified lineage-specific TFs remain hypotheses requiring further experimental validation

## Evidence

- [readme] Bootstrapping quantifies variability of TF activities across trajectory ensemble: "2. tf_activities.m
3. bootstrapping.m
4. differential_motif_activity_analysis.m"
- [discussion] Three criteria for differential TF activity: "considers the significance of the TF to explain the variability of the linear model of gene regulation"
- [intro] Linear model of gene regulation for TF activities: "we modeled the observed gene expression as a linear combination of the regulatory sites and the TF activities"
- [readme] Matlab reproducibility scripts: "Scripts to reproduce the results are available in the [reproducibility](reproducibility/scripts/) folder and are implemented in Matlab."
- [methods] FateCompass implemented in Python: "FateCompass is fully implemented in python."
