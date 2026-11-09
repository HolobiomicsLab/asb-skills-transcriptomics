---
name: mean-dispersion-parameter-estimation
description: 'Use when simulating bulk RNA-seq count data with MOSim: a seed RNA-seq
  sample must be converted into per-gene mean/dispersion NB parameters so that simulated
  counts, DEG profiles (continuous/transitory induction, repression, flat), and regulator
  couplings are statistically consistent with real data.'
when_to_use_negative:
- Simulating single-cell data with sc_mosim — that module draws on SPARSim functionality
  for scRNA-seq counts rather than this bulk seed-sample mean/dispersion estimation
  workflow
- The goal is simulation of omics layers outside MOSim's scope, e.g. proteomics or
  metabolomics, which the authors state are not applicable
- A spatially resolved GRN simulation is required, since MOSim 'is not prepared to
  simulate GRN with spatial resolution'
edam_operation: http://edamontology.org/operation_3633
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: MOSim
  role: 'R package whose mosim() function performs the bulk simulation: seed-sample
    mean/dispersion estimation, NB count generation, DEG profile assignment, and regulator
    profile coupling'
  repo: github.com/ConesaLab/MOSim
- name: SPARSim
  role: Source of the count-data parameterization style used when drawing negative-binomial
    counts from estimated mean/dispersion parameters
- name: R
  role: Implementation language; stats package random functions are used to draw NB
    counts for every gene, time point, and replicate
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:fd19119a781ef59475cb9f7ef1544c8e65ce6178f5e9dc88352d5dd34e0c81aa
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
  - SPARSim
  - dplyr
  - purrr
  - stats
  edam_operation: http://edamontology.org/operation_3633
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# mean-dispersion-parameter-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimates per-gene mean and dispersion parameters from a seed bulk RNA-seq sample to parameterize negative-binomial count simulation in the MOSim bulk (mosim) workflow, the step that grounds simulated RNA-seq counts in realistic NB distributions before temporal DEG profiles and regulatory omics layers are overlaid.

## When to use

Use when simulating bulk RNA-seq count data with MOSim: a seed RNA-seq sample must be converted into per-gene mean/dispersion NB parameters so that simulated counts, DEG profiles (continuous/transitory induction, repression, flat), and regulator couplings are statistically consistent with real data.

## When NOT to use

- Simulating single-cell data with sc_mosim — that module draws on SPARSim functionality for scRNA-seq counts rather than this bulk seed-sample mean/dispersion estimation workflow
- The goal is simulation of omics layers outside MOSim's scope, e.g. proteomics or metabolomics, which the authors state are not applicable
- A spatially resolved GRN simulation is required, since MOSim 'is not prepared to simulate GRN with spatial resolution'

## Inputs

- Seed bulk RNA-seq count matrix (real sample(s) used to estimate per-gene NB mean and dispersion)
- Experimental design parameters specifying number of DEGs, time points, replicates, and DEG profile labels (continuous induction/repression, transitory, flat)
- Regulator–target assignment table (activator/repressor labels) for regulatory omics layers (ATAC-seq, ChIP-seq, miRNA-seq, Methyl-seq, TFs)

## Outputs

- Simulated bulk RNA-seq negative-binomial count matrix (genes x time points x replicates) with profile-specific fold changes applied
- Regulator profile table where activators match and repressors mirror their target genes' temporal patterns

## How to apply

Load the seed bulk RNA-seq sample and estimate per-gene mean and dispersion parameters for negative-binomial modelling, using a SPARSim-style parameterization of the count distributions. Draw negative-binomial counts for every gene, time point, and replicate with R stats random functions from these estimated mean/dispersion values. Then select a subset of features as DEGs and assign each a label — continuous induction/repression (scaled fold change), transitory induction/repression, or flat profile — according to the supplied design parameters, applying profile-specific fold changes for flat and two-group designs. Finally, derive regulator profiles: each activator takes the same temporal pattern as its target genes and each repressor takes the mirrored/opposite pattern. The rationale is that anchoring simulation on the seed sample's estimated NB parameters is what makes downstream benchmarking of multi-omics integration and GRN inference methods meaningful, since MOSim's ground truth (differentially abundant features and active regulatory relationships) depends on counts that mimic real bulk RNA-seq.

## Related tools

- **MOSim** (R package whose mosim() function performs the bulk simulation: seed-sample mean/dispersion estimation, NB count generation, DEG profile assignment, and regulator profile coupling) — github.com/ConesaLab/MOSim
- **SPARSim** (Source of the count-data parameterization style used when drawing negative-binomial counts from estimated mean/dispersion parameters)
- **R** (Implementation language; stats package random functions are used to draw NB counts for every gene, time point, and replicate)

## Evaluation signals

- Simulated counts are consistent with the seed sample's estimated per-gene mean and dispersion parameters
- Emitted regulator profiles correctly correspond to same-pattern relationships for activators and opposite/mirrored patterns for repressors relative to their assigned target genes
- k-means clustering of the simulated feature profiles recovers the assigned temporal DEG profile labels (continuous induction/repression, transitory induction/repression, flat)
- Pearson's correlation between bulk gene–regulator pairs exceeds the distribution obtained from 20,000 random feature pairs with no simulated regulatory effects
- Simulated counts remain non-negative integers on the negative-binomial count scale across all time points and replicates

## Limitations

- The paper reports no quantitative evaluation of this mechanism in isolation; validation is qualitative (count consistency with seed mean/dispersion and correct activator/repressor profile correspondence)
- MOSim's regulatory scope covers ATAC-seq, ChIP-seq, miRNA-seq, Methyl-seq and TFs — not proteomics or metabolomics layers
- MOSim cannot simulate GRNs with spatial resolution

## Evidence
