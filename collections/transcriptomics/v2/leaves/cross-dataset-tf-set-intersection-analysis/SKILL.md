---
name: cross-dataset-tf-set-intersection-analysis
description: Use when two FateCompass differential motif activity results exist for
  orthologous cell fates (e.g., beta cells) in distinct species or systems, and you
  need to compute the intersection fraction and label species-specific TF groupings.
when_to_use_negative:
- Only one dataset is available, or the two systems are not orthologous at the fate
  level (e.g., alpha vs beta comparison), making a cross-species beta-TF intersection
  meaningless.
- Differential motif activity analysis has not yet been run (bootstrapping.m and differential_motif_activity_analysis.m
  outputs are missing), so no differentially active TF sets exist to intersect.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: FateCompass
  role: Python pipeline that estimates time- and lineage-specific TF activities whose
    differential activity results feed the cross-dataset intersection
  repo: https://github.com/sarajimenez/fatecompass
- name: MATLAB
  role: Language of the reproducibility scripts (differentiation_trajectories.m, tf_activities.m,
    bootstrapping.m, differential_motif_activity_analysis.m) that must be executed
    per dataset before intersecting TF sets
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:db0be10cbc51b03e5fa79b8f1f0b202f600e5558f516b02818edbe87da9c826c
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

# cross-dataset-tf-set-intersection-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies the overlap between differentially active beta-specific transcription factor sets inferred by FateCompass for two independent systems (mouse E15.5 in vivo pancreatic development vs human in vitro beta-cell differentiation), classifying shared vs species-specific factors.

## When to use

Use when two FateCompass differential motif activity results exist for orthologous cell fates (e.g., beta cells) in distinct species or systems, and you need to compute the intersection fraction and label species-specific TF groupings.

## When NOT to use

- Only one dataset is available, or the two systems are not orthologous at the fate level (e.g., alpha vs beta comparison), making a cross-species beta-TF intersection meaningless.
- Differential motif activity analysis has not yet been run (bootstrapping.m and differential_motif_activity_analysis.m outputs are missing), so no differentially active TF sets exist to intersect.

## Inputs

- FateCompass differential motif activity analysis results for the mouse E15.5 in vivo pancreatic development dataset
- FateCompass differential motif activity analysis results for the human in vitro beta-cell differentiation dataset
- Pre-processed adata objects with cell type ('clusters') annotations and TF binding site matrices for each dataset

## Outputs

- beta_tf_comparison.csv containing the TF comparison table with overlap statistics and species-specific group labels
- Intersection statistics: overlap fraction (~8%) and count of shared beta-specific TFs (10 factors)

## How to apply

Run the Matlab reproducibility scripts (differentiation_trajectories.m, tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m) for each pancreatic endocrine differentiation dataset under both the RNA velocity driven and differentiation potential driven scenarios. Extract the differentially active beta-specific TF sets from the differential motif activity analysis results for the mouse E15.5 in vivo and human in vitro datasets. Compute the set intersection, quantifying the overlap fraction (~8%) and count of shared factors (10 factors including Esrrg, Neurod1, and Nkx6-1). Classify non-overlapping factors into mouse-specific groups (e.g., Sox9, Meis2) and human-specific waves (early: SOX4, PTF1A, CDX2; late: MAFA, PAX6, GATA4). Write the comparison table with overlap statistics and species-specific labels to beta_tf_comparison.csv.

## Related tools

- **FateCompass** (Python pipeline that estimates time- and lineage-specific TF activities whose differential activity results feed the cross-dataset intersection) — https://github.com/sarajimenez/fatecompass
- **MATLAB** (Language of the reproducibility scripts (differentiation_trajectories.m, tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m) that must be executed per dataset before intersecting TF sets)

## Examples

```
matlab -batch "run('reproducibility/scripts/differential_motif_activity_analysis.m');"  # per dataset/scenario, then intersect the beta-specific TF sets into beta_tf_comparison.csv
```

## Evaluation signals

- Intersection fraction between mouse in vivo and human in vitro beta-specific TF sets is ~8% (10 overlapping factors).
- Shared factors include Esrrg, Neurod1, and Nkx6-1.
- Mouse-specific labels include Sox9 and Meis2; human-specific factors appear in two waves (early: SOX4, PTF1A, CDX2; late: MAFA, PAX6, GATA4).
- Results are consistent under both the RNA velocity driven and differentiation potential driven scenarios.
- beta_tf_comparison.csv contains overlap statistics plus species-specific group labels for all factors.

## Limitations

- FateCompass estimates TF activities using binding sites only in promoter regions; long-range/distal regulatory interactions are not considered, which may affect TF set composition.
- The pipeline depends on input parameters that must be hand tuned, potentially altering the differentially active TF sets being intersected.
- Reliance on computationally predicted regulatory sites in the binding-site matrix introduces bias in the gene-regulatory network structure underlying TF activities.
- Low overlap (~8%) may reflect genuine species/system specificity but also system-level differences (in vivo vs in vitro) that cannot be disentangled by intersection alone.

## Evidence

- [other] Low cross-species overlap of beta-specific TFs: [evidence span withheld — anchor status: not_found]
