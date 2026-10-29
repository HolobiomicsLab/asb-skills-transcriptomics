---
name: pancreatic-endocrine-differentiation-trajectory-analysis
description: Use when comparing lineage-specific TF sets inferred by FateCompass across
  two pancreatic endocrine differentiation datasets (e.g., mouse E15.
when_to_use_negative:
- When only static TF-gene correlation or regulon enrichment is needed without dynamic
  change over the fate-decision process (existing correlation-based methods do not
  describe the dynamic change of TF activity)
- When long-range enhancer-gene interactions must be modeled — FateCompass considers
  only TF-promoter interactions
- When no prior biological knowledge is available to define lineage potentials, since
  the differentiation-potential mode requires this input (problematic e.g. in cancer
  studies)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: FateCompass
  role: Python pipeline that estimates stochastic trajectories and lineage-specific
    TF activities from scRNA-seq via RNA velocity or differentiation potential plus
    a linear model of gene regulation
  repo: https://github.com/sarajimenez/fatecompass
- name: MATLAB
  role: Implementation language of the reproducibility scripts (differentiation_trajectories.m,
    tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m) executed
    in order per dataset and scenario
- name: Scanpy
  role: Suggested tool for basic pre-processing of the input adata object
  repo: https://scanpy.readthedocs.io/en/stable/
- name: scVelo
  role: Suggested tool for computing the RNA velocity field supplied to FateCompass
  repo: https://scvelo.readthedocs.io/en/stable/
- name: Dyngen
  role: Simulation engine used to generate an in silico ground-truth dataset for validating
    FateCompass predictions
  repo: https://github.com/dynverse/dyngen
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:0153f93a42f07844ea14c1ded50de9b72e9f76bfd94f2b0a4ec02960d43674b2
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
  - Scanpy
  - scVelo
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# pancreatic-endocrine-differentiation-trajectory-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A FateCompass-based workflow for inferring dynamic stochastic differentiation trajectories and lineage-specific transcription factor (TF) activities from pancreatic endocrine single-cell RNA-seq data, then quantifying cross-species (mouse in vivo vs human in vitro) overlap of beta-specific TFs.

## When to use

Use when comparing lineage-specific TF sets inferred by FateCompass across two pancreatic endocrine differentiation datasets (e.g., mouse E15.5 in vivo and human in vitro beta-cell differentiation), or when dynamic, time-resolved TF activity changes are needed rather than static correlation-based regulator lists.

## When NOT to use

- When only static TF-gene correlation or regulon enrichment is needed without dynamic change over the fate-decision process (existing correlation-based methods do not describe the dynamic change of TF activity)
- When long-range enhancer-gene interactions must be modeled — FateCompass considers only TF-promoter interactions
- When no prior biological knowledge is available to define lineage potentials, since the differentiation-potential mode requires this input (problematic e.g. in cancer studies)

## Inputs

- Pre-processed adata (AnnData) object with cluster annotations and optionally an RNA velocity field
- TF binding-site matrix (columns = TFs/motifs, rows = expressed genes)
- Two pancreatic endocrine differentiation scRNA-seq datasets: mouse E15.5 in vivo and human in vitro beta-cell differentiation
- MATLAB reproducibility scripts from github.com/sarajimenez/fatecompass (reproducibility/scripts/)

## Outputs

- beta_tf_comparison.csv containing overlap statistics and species-specific group labels
- Differentially active beta-specific TF sets for each species/system
- Count of shared factors (10) and overlap fraction (~8%)
- Bootstrapped and differentially-analyzed motif activity results per trajectory scenario

## How to apply

Run the MATLAB reproducibility scripts in order for each dataset and each trajectory scenario: differentiation_trajectories.m, tf_activities.m, bootstrapping.m, then differential_motif_activity_analysis.m — once with RNA-velocity-driven trajectories and once with differentiation-potential-driven trajectories. FateCompass estimates transition probabilities with a discrete Markov process on a nearest-neighbor network, directed by RNA velocity or biased by the gradient of differentiation potential, then infers TF activities via a linear model of gene regulation (extended from ISMARA) modeling expression as a linear combination of promoter binding sites and TF activities. Filter differentially active TFs by Z score (significance to explain model variability), change of regulatory activity across the fate decision, and dynamical correlation of TF activity with TF mRNA level. Extract the beta-specific differentially active TF sets for mouse in vivo and human in vitro, compute their set intersection (expected ~8% overlap, 10 shared factors), and classify non-overlapping factors as mouse-specific (e.g., Sox9, Meis2) or human-specific waves (early: SOX4, PTF1A, CDX2; late: MAFA, PAX6, GATA4). Write the comparison table with overlap statistics and species-specific labels to beta_tf_comparison.csv.

## Related tools

- **FateCompass** (Python pipeline that estimates stochastic trajectories and lineage-specific TF activities from scRNA-seq via RNA velocity or differentiation potential plus a linear model of gene regulation) — https://github.com/sarajimenez/fatecompass
- **MATLAB** (Implementation language of the reproducibility scripts (differentiation_trajectories.m, tf_activities.m, bootstrapping.m, differential_motif_activity_analysis.m) executed in order per dataset and scenario)
- **Scanpy** (Suggested tool for basic pre-processing of the input adata object) — https://scanpy.readthedocs.io/en/stable/
- **scVelo** (Suggested tool for computing the RNA velocity field supplied to FateCompass) — https://scvelo.readthedocs.io/en/stable/
- **Dyngen** (Simulation engine used to generate an in silico ground-truth dataset for validating FateCompass predictions) — https://github.com/dynverse/dyngen

## Examples

```
conda env create -f fatecompass.yml && conda install notebook  # then run reproducibility/scripts/differentiation_trajectories.m -> tf_activities.m -> bootstrapping.m -> differential_motif_activity_analysis.m
```

## Evaluation signals

- The intersection of mouse and human beta-specific TF sets yields ~8% overlap with exactly 10 shared factors
- The shared-factor list contains Esrrg, Neurod1, and Nkx6-1
- All four MATLAB scripts ran successfully, in the documented order, for both the RNA-velocity-driven and differentiation-potential-driven scenarios
- beta_tf_comparison.csv is produced with species-specific labels: mouse-specific (Sox9, Meis2) and human-specific in two waves (early SOX4/PTF1A/CDX2; late MAFA/PAX6/GATA4)
- Differential TF activity analysis passes the three FateCompass criteria: Z-score significance, change of regulatory activity over the fate decision, and dynamical correlation of TF activity with TF mRNA level

## Limitations

- TF activities are estimated using binding sites only in promoter regions; distal/long-range regulatory interactions are not considered, which biases the gene-regulatory network structure
- Pipeline input parameters must be hand tuned; detailed tutorials are provided to guide parameter setup
- Trajectories built in a reduced gene-expression space rely on nearest-neighbor graphs, and UMAP-style dimensionality reductions lack biological interpretability
- Putative driver factors identified (e.g., EC-specific CDX2) require further experimental validation of their involvement in endocrine cell differentiation

## Evidence
