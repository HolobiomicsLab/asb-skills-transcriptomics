---
name: in-vivo-in-vitro-trajectory-result-comparison
description: Use when two FateCompass differential TF activity result sets exist (e.g.
  mouse in vivo E15.5 and human in vitro beta differentiation) and the goal is to
  quantify shared versus system-specific beta-cell regulators.
when_to_use_negative:
- Only one dataset/trajectory is available — there is no second TF set to intersect
- The two systems target different lineages or terminal cell types, so beta-specific
  factor sets are not comparable
- The analysis goal is validating predictions against ground truth rather than cross-system
  comparison (use the Dyngen in silico validation workflow instead)
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: FateCompass
  role: Generates the time- and lineage-specific TF activity results (differential
    motif activity analysis) for each dataset that are then intersected across systems
  repo: https://github.com/sarajimenez/fatecompass
- name: MATLAB
  role: Language in which the reproducibility scripts (differentiation_trajectories.m,
    tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m) are
    implemented and executed per dataset
  repo: https://github.com/sarajimenez/fatecompass
- name: Dyngen
  role: Simulation engine used separately to generate ground-truth in silico data
    for validating FateCompass predictions before cross-system comparison
  repo: https://github.com/dynverse/dyngen
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:9f641a8a13f21dab2c334716cf2762e31776734034e938b91fa78dfc301141e8
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# in-vivo-in-vitro-trajectory-result-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the overlap between lineage-specific transcription factors inferred by FateCompass for mouse in vivo pancreatic development (E15.5) and human in vitro beta-cell differentiation, classifying non-overlapping factors as species- or system-specific.

## When to use

Use when two FateCompass differential TF activity result sets exist (e.g. mouse in vivo E15.5 and human in vitro beta differentiation) and the goal is to quantify shared versus system-specific beta-cell regulators.

## When NOT to use

- Only one dataset/trajectory is available — there is no second TF set to intersect
- The two systems target different lineages or terminal cell types, so beta-specific factor sets are not comparable
- The analysis goal is validating predictions against ground truth rather than cross-system comparison (use the Dyngen in silico validation workflow instead)

## Inputs

- Differentially active beta-specific TF set from mouse E15.5 in vivo dataset (differential_motif_activity_analysis.m output)
- Differentially active beta-specific TF set from human in vitro beta-cell differentiation dataset
- FateCompass results from both RNA velocity driven and differentiation potential driven scenarios
- TF binding site matrix for each species (columns = TFs/motifs, rows = expressed genes)

## Outputs

- TF comparison table with overlap fraction, shared-factor count, and species-specific group labels (beta_tf_comparison.csv)
- List of shared factors (e.g. Esrrg, Neurod1, Nkx6-1)
- Mouse-specific and human-specific factor groupings with early/late wave annotation

## How to apply

First run the FateCompass Matlab reproducibility scripts in order — differentiation_trajectories.m, tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m — for each pancreatic endocrine differentiation dataset under both the RNA velocity driven and differentiation potential driven scenarios. Extract the differentially active beta-specific TF sets from the differential motif activity analysis results for the mouse E15.5 in vivo and human in vitro datasets. Compute the set intersection of the two TF lists to obtain the overlap count and fraction (expected ~8%, i.e. 10 shared factors including Esrrg, Neurod1, and Nkx6-1). Classify non-overlapping factors into mouse-specific (e.g. Sox9, Meis2) and human-specific groups, noting that human-specific factors appear in two waves (early: SOX4, PTF1A, CDX2; late: MAFA, PAX6, GATA4). Write the comparison table with overlap statistics and species-specific group labels to CSV for downstream interpretation.

## Related tools

- **FateCompass** (Generates the time- and lineage-specific TF activity results (differential motif activity analysis) for each dataset that are then intersected across systems) — https://github.com/sarajimenez/fatecompass
- **MATLAB** (Language in which the reproducibility scripts (differentiation_trajectories.m, tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m) are implemented and executed per dataset) — https://github.com/sarajimenez/fatecompass
- **Dyngen** (Simulation engine used separately to generate ground-truth in silico data for validating FateCompass predictions before cross-system comparison) — https://github.com/dynverse/dyngen

## Examples

```
matlab -batch "run('differential_motif_activity_analysis.m');"  # per dataset; then intersect the beta-specific TF sets and write beta_tf_comparison.csv
```

## Evaluation signals

- Shared-factor count matches the reported overlap (10 factors) and overlap fraction is ~8% of the differentially active beta-specific TFs
- Intersection includes Esrrg, Neurod1, and Nkx6-1
- Mouse-specific set contains Sox9 and Meis2; human-specific set contains SOX4, PTF1A, CDX2 (early) and MAFA, PAX6, GATA4 (late)
- Comparison is reproduced under both the RNA velocity driven and differentiation potential driven scenarios
- Output table beta_tf_comparison.csv contains overlap statistics plus species-specific group labels for every TF

## Limitations

- The comparison uses FateCompass TF activities estimated from binding sites in promoter regions only, ignoring distal regulatory elements and long-range interactions
- Reliance on computationally predicted regulatory sites in the binding-site matrix introduces bias into the gene-regulatory network structure underlying each species' result
- Pipeline input parameters must be hand tuned per dataset, so cross-system differences may partly reflect parameter choices rather than biology
- Human-specific factors identified in the in vitro system (e.g. the enterochromaffin-like population factors) require further experimental validation

## Evidence

- [readme] Matlab reproducibility script ordering: [evidence span withheld — anchor status: not_found]
- [results] Overlap result from source card finding: [evidence span withheld — anchor status: not_found]
