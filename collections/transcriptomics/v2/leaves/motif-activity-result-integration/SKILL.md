---
name: motif-activity-result-integration
description: Use when you have differentially active TF sets from two or more FateCompass
  (or similar motif-activity) runs and need to quantify their intersection and partition
  factors into shared vs. system-specific groups.
when_to_use_negative:
- You have raw scRNA-seq counts and have not yet run the FateCompass TF activity estimation
  and differential motif activity analysis — integrate only finished differentially
  active TF sets
- The datasets being compared lack comparable TF binding-site matrices or lineage
  annotations (e.g. cancer studies where prior biological knowledge for differentiation-potential
  mode is unavailable)
- The goal is method validation on simulated data rather than cross-system comparison
  — use the Dyngen ground-truth workflow instead
edam_topics:
- http://edamontology.org/topic_0204
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3308
tools:
- name: FateCompass
  role: Pipeline whose differential motif activity analysis produces the differentially
    active TF sets that are integrated across the mouse in vivo and human in vitro
    datasets
  repo: https://github.com/sarajimenez/fatecompass
- name: MATLAB
  role: Environment in which the reproducibility scripts (differentiation_trajectories.m
    through differential_motif_activity_analysis.m) are executed and the intersection
    table is generated
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:b53e23e60d5c905aa6b1811455d21da973eae331ea49127f9bf55c516485f02a
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
  - MATLAB
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0204
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# Motif activity result integration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Integrates differentially active motif/TF activity outputs from FateCompass across multiple datasets (here, mouse E15.5 in vivo and human in vitro pancreatic endocrine differentiation) to quantify cross-system overlap of lineage-specific transcription factors and classify species- or system-specific factors.

## When to use

Use when you have differentially active TF sets from two or more FateCompass (or similar motif-activity) runs and need to quantify their intersection and partition factors into shared vs. system-specific groups.

## When NOT to use

- You have raw scRNA-seq counts and have not yet run the FateCompass TF activity estimation and differential motif activity analysis — integrate only finished differentially active TF sets
- The datasets being compared lack comparable TF binding-site matrices or lineage annotations (e.g. cancer studies where prior biological knowledge for differentiation-potential mode is unavailable)
- The goal is method validation on simulated data rather than cross-system comparison — use the Dyngen ground-truth workflow instead

## Inputs

- Differentially active beta-specific TF sets from the differential_motif_activity_analysis.m output for each dataset (mouse E15.5 in vivo and human in vitro)
- FateCompass MATLAB outputs from tf_activities.m and bootstrapping.m for both trajectory scenarios (RNA velocity driven and differentiation potential driven)
- Pre-processed adata objects with cell-type ('clusters') annotations and TF binding-site matrices

## Outputs

- beta_tf_comparison.csv — TF comparison table with overlap statistics and species-specific group labels
- Intersection count and overlap fraction of differentially active TFs between the two systems
- Classified mouse-specific and human-specific TF groupings (early/late waves)

## How to apply

Run the MATLAB reproducibility scripts in order for each dataset — differentiation_trajectories.m, tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m — under both trajectory scenarios (RNA velocity driven and differentiation potential driven). Extract the differentially active beta-specific TF sets from the differential motif activity analysis results for the mouse E15.5 in vivo and human in vitro datasets. Compute the set intersection of the two TF lists with set-intersection logic to obtain the shared-factor count and overlap fraction (~8% in the reference case, 10 shared factors including Esrrg, Neurod1, and Nkx6-1). Classify non-overlapping factors into system-specific dynamic groupings (mouse-specific: e.g. Sox9, Meis2; human-specific early wave: SOX4, PTF1A, CDX2; late wave: MAFA, PAX6, GATA4). Write the comparison table with overlap statistics and group labels to beta_tf_comparison.csv.

## Related tools

- **FateCompass** (Pipeline whose differential motif activity analysis produces the differentially active TF sets that are integrated across the mouse in vivo and human in vitro datasets) — https://github.com/sarajimenez/fatecompass
- **MATLAB** (Environment in which the reproducibility scripts (differentiation_trajectories.m through differential_motif_activity_analysis.m) are executed and the intersection table is generated)

## Examples

```
matlab -batch differentiation_trajectories.m; matlab -batch tf_activities.m; matlab -batch bootstrapping.m; matlab -batch differential_motif_activity_analysis.m
```

## Evaluation signals

- The intersection count matches the reference overlap (~8% of differentially active beta-specific TFs, 10 shared factors)
- Shared-factor list contains the expected core factors Esrrg, Neurod1, and Nkx6-1
- Mouse-specific group contains Sox9 and Meis2; human-specific groups contain the early wave (SOX4, PTF1A, CDX2) and late wave (MAFA, PAX6, GATA4)
- beta_tf_comparison.csv is produced with both overlap statistics and species-specific group labels
- Results are consistent across both trajectory scenarios (RNA velocity driven and differentiation potential driven)

## Limitations

- FateCompass estimates TF activities using binding sites only in promoter regions; distal/long-range interactions are not considered, biasing the gene-regulatory network structure
- Pipeline input parameters must be hand tuned, which can affect the differentially active TF sets being integrated
- Identified putative driver factors require further experimental validation before being treated as confirmed differentiation regulators

## Evidence

- [methods] Workflow step 3 — intersection computation: [evidence span withheld — anchor status: not_found]
