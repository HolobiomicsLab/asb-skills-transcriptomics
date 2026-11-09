---
name: negative-binomial-count-simulation
description: 'Use when building a bulk RNA-seq simulator with realistic count distributions:
  you need NB draws per gene, time point and replicate, plus controlled DEG profiles
  (induction, repression, transitory, flat) and coupled regulator profiles.'
when_to_use_negative:
- The goal is single-cell multi-omics simulation — use the sc_mosim module, which
  relies on SPARSim functionality rather than this bulk mosim NB workflow.
- The input data is real experimental RNA-seq to be analyzed rather than a seed sample
  for parameter estimation in a simulation.
- Simulation requires spatial resolution or omics layers like proteomics and metabolomics,
  which MOSim does not support.
edam_operation: http://edamontology.org/operation_0438
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: MOSim
  role: R package providing the mosim bulk simulation function that implements NB
    count generation and regulator profile coupling
  repo: github.com/ConesaLab/MOSim
- name: SPARSim
  role: Source of the SPARSim-style parameterization used for the negative-binomial
    count draws
- name: R stats
  role: Provides the R stats random functions used to draw negative-binomial counts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:cce9155d735eeb36f5f092733b1a4d43858e614940f4d9fea93d9bf920495799
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  - R stats
  - SPARSim
  - dplyr
  - purrr
  - stats
  edam_operation: http://edamontology.org/operation_0438
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# negative-binomial-count-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulates bulk RNA-seq count matrices by drawing negative-binomial counts from per-gene mean and dispersion parameters estimated on a seed RNA-seq sample (SPARSim-style parameterization), then applying profile-specific fold changes for DEG patterns.

## When to use

Use when building a bulk RNA-seq simulator with realistic count distributions: you need NB draws per gene, time point and replicate, plus controlled DEG profiles (induction, repression, transitory, flat) and coupled regulator profiles.

## When NOT to use

- The goal is single-cell multi-omics simulation — use the sc_mosim module, which relies on SPARSim functionality rather than this bulk mosim NB workflow.
- The input data is real experimental RNA-seq to be analyzed rather than a seed sample for parameter estimation in a simulation.
- Simulation requires spatial resolution or omics layers like proteomics and metabolomics, which MOSim does not support.

## Inputs

- Seed bulk RNA-seq count sample used to estimate per-gene negative-binomial mean and dispersion parameters
- Design parameters specifying the DEG subset and temporal profile labels (continuous induction/repression, transitory induction/repression, flat)
- Regulator-to-target-gene assignment with activator/repressor labels

## Outputs

- Simulated bulk RNA-seq negative-binomial count matrix (genes x time points x replicates) with profile-specific fold changes applied
- Regulator profile table encoding activator (same-pattern) and repressor (opposite-pattern) relationships to target genes

## How to apply

Start from a seed bulk RNA-seq sample and estimate per-gene mean and dispersion parameters for negative-binomial modelling. Randomly select a subset of features as differentially expressed genes and label each as continuous induction/repression (scaled fold change), transitory induction/repression, or flat, according to the design parameters; supports flat and two-group designs. Draw NB counts for every gene, time point, and replicate using the estimated mean/dispersion (R stats random functions, SPARSim-style parameterization), then apply profile-specific fold changes to the base counts. For the regulatory layers, set each activator's temporal profile identical to its target genes' pattern and each repressor's pattern to the mirrored/opposite pattern, and write the simulated bulk count matrix plus the regulator profile table as outputs. Validate qualitatively (the paper reports no isolated quantitative evaluation of this mechanism) by checking counts are consistent with the seed's estimated mean/dispersion and that regulator profiles correctly mirror or match their targets.

## Related tools

- **MOSim** (R package providing the mosim bulk simulation function that implements NB count generation and regulator profile coupling) — github.com/ConesaLab/MOSim
- **SPARSim** (Source of the SPARSim-style parameterization used for the negative-binomial count draws)
- **R stats** (Provides the R stats random functions used to draw negative-binomial counts)

## Examples

```
BiocManager::install("MOSim"); library(MOSim); sim <- mosim(omics = "RNA-seq")
```

## Evaluation signals

- Simulated counts are consistent with the seed sample's estimated per-gene mean and dispersion (NB parameterization check).
- Each activator's simulated temporal profile is identical to its assigned target genes' profile.
- Each repressor's simulated temporal profile is the mirrored/opposite pattern of its target genes (e.g., continuous induction vs continuous repression).
- K-means clustering of simulated feature profiles recovers the temporal expression patterns specified in the design settings.
- Regulator–gene Pearson correlations exceed those of 20,000 random feature pairs with no simulated regulatory effects.

## Limitations

- The paper reports no quantitative evaluation of the bulk NB count and regulator-coupling mechanism in isolation; validation is qualitative.
- MOSim's regulator layers beyond RNA-seq are limited to ATAC-seq, miRNA-seq, ChIP-seq, Methyl-seq and TFs for bulk; proteomics and metabolomics layers are not supported.
- MOSim cannot simulate GRNs with spatial resolution; extension to spatial information is deferred to future work.

## Evidence
