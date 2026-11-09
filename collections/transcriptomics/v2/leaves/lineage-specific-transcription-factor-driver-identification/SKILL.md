---
name: lineage-specific-transcription-factor-driver-identification
description: Use when scRNA-seq data of a differentiating system has annotated clusters/terminal
  fates and you need to estimate fate probabilities and identify which TFs drive each
  final fate.
when_to_use_negative:
- When no prior biological knowledge (e.g., fate marker genes) is available, since
  differentiation-potential mode depends on prior knowledge — difficult in settings
  such as cancer studies
- When you need distal (enhancer) regulatory interactions, since FateCompass estimates
  TF activities using binding sites only in promoter regions
- When RNA velocity profiles are not robust and no fate markers are available to define
  terminal states for the Markov chain
edam_operation: http://edamontology.org/operation_0533
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: FateCompass
  role: 'Core pipeline: estimates transition probabilities from RNA velocity or differentiation
    potential, computes TF activities with a linear model of gene regulation, and
    performs differential motif activity analysis'
  repo: https://github.com/sarajimenez/fatecompass
- name: Dyngen
  role: Simulates a ground-truth in silico dataset (e.g., 1000 cells, 1035 genes,
    130 TFs) with known gene-regulatory network for validation of fate-probability
    and driver recovery
  repo: https://github.com/dynverse/dyngen
- name: Scanpy
  role: Recommended for basic pre-processing of the scRNA-seq counts into the AnnData
    input object
- name: scVelo
  role: Recommended for computing the RNA velocity field optionally supplied in the
    AnnData object
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100512
    title: Characterization of cell-fate decision landscapes by estimating transcription
      factor dynamics
  source_content_hash: sha256:8f48e80263b82381b2601e8f6370acac0bb9a95a20c975860130d48f3b5c4e69
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
  - Python
  - Scanpy
  - scVelo
  edam_operation: http://edamontology.org/operation_0533
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# lineage-specific-transcription-factor-driver-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify time- and lineage-specific transcription factor drivers of cell-fate decisions from single-cell RNA-seq by combining FateCompass's probabilistic Markov-chain trajectory framework (driven by RNA velocity or differentiation potential) with a linear model of gene regulation that infers TF activities, followed by differential motif activity analysis.

## When to use

Use when scRNA-seq data of a differentiating system has annotated clusters/terminal fates and you need to estimate fate probabilities and identify which TFs drive each final fate.

## When NOT to use

- When no prior biological knowledge (e.g., fate marker genes) is available, since differentiation-potential mode depends on prior knowledge — difficult in settings such as cancer studies
- When you need distal (enhancer) regulatory interactions, since FateCompass estimates TF activities using binding sites only in promoter regions
- When RNA velocity profiles are not robust and no fate markers are available to define terminal states for the Markov chain

## Inputs

- Pre-processed AnnData (adata) object with cell-type ('clusters') annotations and optionally an RNA velocity field
- TF binding-site matrix with TFs/motifs as columns and expressed genes as rows
- Marker genes defining terminal fates (for differentiation-potential mode)
- fatecompass.yml conda environment specification and fatecompass.py module

## Outputs

- Per-cell fate probability estimates toward each terminal state
- Stochastic trajectories of gene expression and TF activities
- Differential motif/TF activity analysis results ranking lineage-specific drivers (e.g., a CSV of fate probabilities and driver list such as fatecompass_dyngen_results.csv)

## How to apply

Pre-process the scRNA-seq counts into an AnnData object with at least 'cluster' annotations (optionally RNA velocity), using Scanpy and scVelo, and build a TF binding-site matrix with TFs/motifs as columns and expressed genes as rows. Create the FateCompass conda environment (conda env create -f fatecompass.yml) and run the fatecompass.py module in either RNA-velocity mode or differentiation-potential mode, the latter biasing Markov-chain transition probabilities with the gradient of differentiation potential using marker genes to define terminal fates. Extract per-cell transition probabilities toward each terminal fate, then compute TF activities with the linear model of gene regulation that models gene expression as a linear combination of regulatory sites and TF activities (an extension of the ISMARA model with data-diffusion-based regularization). Run differential motif activity analysis, ranking TFs by Z-score significance in the linear model, change of regulatory activity across the fate decision, and dynamical correlation between TF activity and TF mRNA level. The workflow order in the reproducibility scripts is: differentiation_trajectories, tf_activities, bootstrapping, differential_motif_activity_analysis. Validate the setup against a ground-truth in silico system (e.g., Dyngen simulation) where the true fate drivers are known.

## Related tools

- **FateCompass** (Core pipeline: estimates transition probabilities from RNA velocity or differentiation potential, computes TF activities with a linear model of gene regulation, and performs differential motif activity analysis) — https://github.com/sarajimenez/fatecompass
- **Dyngen** (Simulates a ground-truth in silico dataset (e.g., 1000 cells, 1035 genes, 130 TFs) with known gene-regulatory network for validation of fate-probability and driver recovery) — https://github.com/dynverse/dyngen
- **Scanpy** (Recommended for basic pre-processing of the scRNA-seq counts into the AnnData input object)
- **scVelo** (Recommended for computing the RNA velocity field optionally supplied in the AnnData object)

## Examples

```
conda env create -f fatecompass.yml; conda install notebook; jupyter notebook tutorials/fatecompass_basics_differentiation_potential.ipynb
```

## Evaluation signals

- Fate probabilities are accurately recovered on a Dyngen ground-truth dataset for known terminal states (e.g., sEndC and sEndD)
- Differential TF activity analysis recovers the known ground-truth fate drivers (e.g., B11 for sEndC and B14 for sEndD)
- The output fate probabilities and driver list match the reported validation results when saved (e.g., fatecompass_dyngen_results.csv)
- TF activity ranking reflects the three filter criteria: linear-model Z score, change of regulatory activity across the fate decision, and dynamical correlation of TF activity with TF mRNA level
- Application to pancreatic endocrine differentiation recapitulates known lineage-specific regulators (e.g., CDX2 for EC fate) alongside novel candidates

## Limitations

- Differentiation-potential mode requires prior biological knowledge (fate markers), which is hard to supply in settings such as cancer studies
- TF activities use only computationally predicted promoter binding sites; long-range/distal regulatory interactions are not modeled, biasing the inferred gene-regulatory network structure
- Pipeline input parameters must be hand tuned, though detailed tutorials guide parameter setup
- Trajectories rely on nearest-neighbor graphs in reduced gene-expression space; UMAP-like embeddings lack biological interpretability
- Predicted driver TFs require further experimental validation before being considered confirmed

## Evidence
