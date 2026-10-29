---
name: regulator-target-profile-matching
description: Use when simulating bulk multi-omics regulatory layers with MOSim and
  you must decide how a regulator's simulated count/time profile relates to the expression
  pattern of its assigned target genes.
when_to_use_negative:
- The input is real experimental regulator data (e.g., measured ChIP-seq or Methyl-seq
  counts) rather than a simulation task — this skill only constructs synthetic regulator
  profiles coupled to simulated gene expression.
- Simulating single-cell data with sc_mosim, where regulatory layers are restricted
  to scATAC-seq and TFs and count generation relies on SPARSim single-cell functionality
  rather than the bulk activator/repressor mirroring mechanism.
- Modeling omics layers like proteomics or metabolomics, which the paper explicitly
  does not cover.
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0204
tools:
- name: MOSim
  role: R package whose mosim() function performs the bulk simulation, including derivation
    of regulator profiles from target-gene DEG patterns
  repo: github.com/ConesaLab/MOSim
- name: SPARSim
  role: Provides the count-data parameterization style used when drawing negative-binomial
    counts from seed-sample mean/dispersion estimates
- name: stats
  role: R stats random functions used to draw negative-binomial counts for every gene,
    time point, and replicate
- name: dplyr
  role: Data manipulation for assigning DEG labels and constructing the regulator
    profile table
- name: purrr
  role: Functional iteration over regulator-target pairs when deriving activator/repressor
    profiles
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:08cee423c0beec62e225f714d4077268f2c3456e9468ef3b587aa246db79739d
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
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# regulator-target-profile-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

In MOSim's bulk simulation workflow, regulatory omics layers (ATAC-seq, ChIP-seq, miRNA-seq, Methyl-seq, TFs) receive count profiles derived from the genes they regulate: activator regulators mirror the temporal DEG profile of their targets, while repressors follow the opposite pattern, coupling regulator abundance to gene expression.

## When to use

Use when simulating bulk multi-omics regulatory layers with MOSim and you must decide how a regulator's simulated count/time profile relates to the expression pattern of its assigned target genes.

## When NOT to use

- The input is real experimental regulator data (e.g., measured ChIP-seq or Methyl-seq counts) rather than a simulation task — this skill only constructs synthetic regulator profiles coupled to simulated gene expression.
- Simulating single-cell data with sc_mosim, where regulatory layers are restricted to scATAC-seq and TFs and count generation relies on SPARSim single-cell functionality rather than the bulk activator/repressor mirroring mechanism.
- Modeling omics layers like proteomics or metabolomics, which the paper explicitly does not cover.

## Inputs

- Seed bulk RNA-seq count sample (e.g., STATegra mouse multi-omics data) for per-gene mean/dispersion estimation
- Simulated negative-binomial RNA-seq count matrix across genes, time points, and replicates
- Regulator-to-target gene assignment map for regulatory layers (ATAC-seq, ChIP-seq, miRNA-seq, Methyl-seq, TFs)
- Design parameters specifying DEG subset, DEG labels (continuous induction/repression, transitory induction/repression, flat), and experimental design (flat / two-group / time-course)

## Outputs

- Simulated bulk count matrix for regulatory omics layers with activator/repressor-derived profiles
- Regulator profile table recording each regulator's temporal pattern and its same-pattern/opposite-pattern relationship to assigned target genes

## How to apply

After drawing negative-binomial counts for every gene, time point, and replicate from per-gene mean and dispersion parameters estimated on the seed bulk RNA-seq sample (SPARSim-style parameterization), assign each DEG one of the temporal labels — continuous induction/repression (scaled fold change), transitory induction/repression, or flat — according to the design parameters, and apply profile-specific fold changes for flat and two-group designs. For each regulator–target link, derive the regulator's own count profile: an activator is given the identical temporal pattern as its target genes (e.g., continuous induction paired with continuous induction), whereas a repressor is given the mirrored/opposite pattern (e.g., continuous induction vs continuous repression). Emit the simulated bulk count matrix together with the regulator profile table so downstream users can trace which regulators share or invert their targets' patterns. Validate qualitatively (the paper reports no isolated quantitative evaluation of this mechanism) by checking that simulated counts remain consistent with the seed sample's estimated mean/dispersion and that emitted regulator profiles correctly correspond to same-pattern (activator) or opposite-pattern (repressor) relationships; at the paper level, Pearson's correlation on bulk gene–regulator pairs, benchmarked against 20,000 random feature pairs with no simulated regulatory effects, confirms active regulatory relationships.

## Related tools

- **MOSim** (R package whose mosim() function performs the bulk simulation, including derivation of regulator profiles from target-gene DEG patterns) — github.com/ConesaLab/MOSim
- **SPARSim** (Provides the count-data parameterization style used when drawing negative-binomial counts from seed-sample mean/dispersion estimates)
- **stats** (R stats random functions used to draw negative-binomial counts for every gene, time point, and replicate)
- **dplyr** (Data manipulation for assigning DEG labels and constructing the regulator profile table)
- **purrr** (Functional iteration over regulator-target pairs when deriving activator/repressor profiles)

## Examples

```
BiocManager::install("MOSim"); sim <- mosim(RNAseq = seed_rnaseq, regulator = c("ATAC-seq", "ChIP-seq", "miRNA-seq", "Methyl-seq", "TF"))
```

## Evaluation signals

- Simulated regulatory-omics counts are consistent with the seed sample's estimated per-gene mean and dispersion parameters
- Each activator's emitted temporal profile is identical to its assigned target genes' profile (same-pattern check)
- Each repressor's emitted temporal profile is the mirrored/opposite pattern of its target genes' profile (e.g., continuous induction vs continuous repression)
- Pearson's correlation for bulk gene–regulator pairs is compared against 20,000 random feature pairs with no simulated regulatory effects
- k-means clustering of simulated feature profiles recovers the intended temporal expression patterns

## Limitations

- The paper reports no quantitative evaluation of this activator/repressor coupling mechanism in isolation; acceptance is qualitative.
- Single-cell simulation is restricted to scRNA-seq and scATAC-seq, so this bulk coupling pattern does not generalize to other single-cell regulatory layers.
- MOSim does not cover proteomics or metabolomics layers and cannot simulate GRNs with spatial resolution.

## Evidence
