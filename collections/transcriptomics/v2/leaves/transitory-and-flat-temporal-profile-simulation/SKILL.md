---
name: transitory-and-flat-temporal-profile-simulation
description: Use when building a ground-truth bulk multi-omics simulation over a time-series
  or two-group design, where DEGs must carry labeled temporal profiles (scaled fold
  change, transitory, or flat) and regulator layers must reflect activator/repressor
  coupling.
when_to_use_negative:
- Input is single-cell data — this skill covers the bulk mosim workflow only; single-cell
  simulation goes through sc_mosim, which relies on SPARSim functionality
- The goal is a spatially resolved GRN simulation — MOSim is not prepared to simulate
  GRNs with spatial resolution
- The target omics layers are proteomics or metabolomics, which are outside MOSim's
  simulated regulatory omics (RNA-seq, ATAC-seq, miRNA-seq, ChIP-seq, Methyl-seq,
  TFs)
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MOSim
  role: R/Bioconductor package providing the mosim function that generates bulk RNA-seq
    counts plus regulatory omics layers with activator/repressor profile coupling
  repo: github.com/ConesaLab/MOSim
- name: SPARSim
  role: Provides the count-data parameterization style (mean/dispersion from a seed
    sample) used when drawing negative-binomial counts
- name: R
  role: Implementation language; stats random functions draw the negative-binomial
    counts, dplyr/purrr orchestrate profile assignment and regulator derivation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:8f84d28d0eea07bf87bee7cf9324c6ec852be246e799f927a1dcb0185c0d2b5a
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# Transitory-and-flat temporal profile simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates bulk RNA-seq count data under the MOSim (mosim) workflow by fitting per-gene negative-binomial mean/dispersion parameters from a seed sample and assigning differentially expressed genes one of five temporal profiles (continuous induction/repression, transitory induction/repression, flat), then coupling each regulator's profile to its target genes as same-pattern (activator) or mirrored/opposite-pattern (repressor).

## When to use

Use when building a ground-truth bulk multi-omics simulation over a time-series or two-group design, where DEGs must carry labeled temporal profiles (scaled fold change, transitory, or flat) and regulator layers must reflect activator/repressor coupling.

## When NOT to use

- Input is single-cell data — this skill covers the bulk mosim workflow only; single-cell simulation goes through sc_mosim, which relies on SPARSim functionality
- The goal is a spatially resolved GRN simulation — MOSim is not prepared to simulate GRNs with spatial resolution
- The target omics layers are proteomics or metabolomics, which are outside MOSim's simulated regulatory omics (RNA-seq, ATAC-seq, miRNA-seq, ChIP-seq, Methyl-seq, TFs)

## Inputs

- Seed bulk RNA-seq sample (count matrix used to estimate per-gene negative-binomial mean and dispersion parameters)
- Design parameters specifying time points, biological replicates, number of DEGs, and profile label distribution (continuous/transitory/flat induction or repression)
- Regulator–target gene mapping with activator/repressor signs (e.g., TFs, ATAC-seq, ChIP-seq, miRNA-seq, Methyl-seq regulatory relationships)

## Outputs

- Simulated bulk RNA-seq negative-binomial count matrix (genes × time points × replicates)
- Regulator profile table recording same-pattern (activator) or opposite-pattern (repressor) coupling to target genes
- Per-gene DEG label assignment (continuous induction/repression, transitory induction/repression, flat)

## How to apply

Load the seed bulk RNA-seq sample and estimate per-gene mean and dispersion parameters for negative-binomial modelling (SPARSim-style parameterization drawn with R stats random functions). Randomly select a subset of features as DEGs and assign each a label — continuous induction or repression with a scaled fold change, transitory induction or repression, or flat — according to the supplied design parameters for the time-point or two-group design. Draw negative-binomial counts for every gene, time point, and replicate from the estimated mean/dispersion, then apply profile-specific fold changes for flat and two-group designs. Derive regulator profiles by setting each activator's temporal pattern identical to its target genes' pattern and each repressor's pattern to the mirrored/opposite pattern (e.g., continuous induction vs continuous repression). Finally write the simulated bulk count matrix and the regulator profile table to disk as the module's outputs.

## Related tools

- **MOSim** (R/Bioconductor package providing the mosim function that generates bulk RNA-seq counts plus regulatory omics layers with activator/repressor profile coupling) — github.com/ConesaLab/MOSim
- **SPARSim** (Provides the count-data parameterization style (mean/dispersion from a seed sample) used when drawing negative-binomial counts)
- **R** (Implementation language; stats random functions draw the negative-binomial counts, dplyr/purrr orchestrate profile assignment and regulator derivation)

## Examples

```
sim <- mosim(omics = c("RNA-seq", "TF"), times = c(0, 6, 12), replicates = 3)
```

## Evaluation signals

- Simulated counts are consistent with the seed sample's estimated negative-binomial mean and dispersion parameters
- Emitted regulator profiles correctly correspond to same-pattern (activator) or opposite-pattern (repressor) relationships with their assigned target genes
- k-means clustering of the simulated feature profiles recovers the intended temporal expression patterns (e.g., separates flat, transitory, and continuous profile classes)
- Pearson's correlation for bulk gene–regulator pairs exceeds correlations computed against 20,000 random feature pairs with no simulated regulatory effects
- Each DEG carries exactly one of the five profile labels and fold changes are applied as specified for flat and two-group designs

## Limitations

- The paper reports no quantitative evaluation of the regulator-coupling mechanism in isolation, so accept this module qualitatively by checking count consistency and profile correspondence
- MOSim cannot simulate GRNs with spatial resolution; extension to spatial information is deferred to future work
- Not applicable to omics layers like proteomics and metabolomics, which may influence gene regulation in more complex or uncertain ways

## Evidence
