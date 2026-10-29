---
name: differential-expression-profile-labelling
description: Use when building a synthetic bulk RNA-seq time-course with ground-truth
  DEG classes — e.g. seeding a simulator from a real sample's NB mean/dispersion and
  needing labelled induction/repression/flat temporal profiles.
when_to_use_negative:
- When simulating single-cell data with sc_mosim — this labelling belongs to the bulk
  mosim negative-binomial workflow, whereas sc_mosim relies on SPARSim single-cell
  functionality
- When analysing real experimental data for DE testing — this skill only assigns synthetic
  ground-truth labels, it does not perform statistical differential expression calling
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: MOSim
  role: R package whose mosim function implements the bulk simulation, DEG profile
    labelling, and activator/repressor profile coupling
  repo: https://github.com/ConesaLab/MOSim
- name: R
  role: Language in which the negative-binomial count draws and profile-specific fold
    changes are computed via stats random functions
- name: SPARSim
  role: Source of the count-data parameterization style used for the negative-binomial
    simulation of gene counts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/bib/bbaf110
    title: 'MOSim: bulk and single-cell multilayer regulatory network simulator'
  source_content_hash: sha256:1282edfc5a8417d4aebe0c8153c5d1df2bbde300cb2ca74f4f207e229ed26389
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/ConesaLab/MOSim
  repo_url: https://github.com/ConesaLab/MOSim
  tools:
  - MOSim
  - R
  - SPARSim
  - dplyr
  - purrr
  - stats
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# differential-expression-profile-labelling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Assigns simulated genes discrete temporal differential-expression labels (continuous induction/repression, transitory induction/repression, or flat) in the MOSim bulk workflow, then applies profile-specific fold changes to negative-binomial base counts.

## When to use

Use when building a synthetic bulk RNA-seq time-course with ground-truth DEG classes — e.g. seeding a simulator from a real sample's NB mean/dispersion and needing labelled induction/repression/flat temporal profiles.

## When NOT to use

- When simulating single-cell data with sc_mosim — this labelling belongs to the bulk mosim negative-binomial workflow, whereas sc_mosim relies on SPARSim single-cell functionality
- When analysing real experimental data for DE testing — this skill only assigns synthetic ground-truth labels, it does not perform statistical differential expression calling

## Inputs

- Seed bulk RNA-seq count matrix used to estimate per-gene mean and dispersion parameters
- Design parameters specifying the subset of differentially expressed genes and profile class proportions

## Outputs

- Labelled DEG profile table (continuous induction/repression, transitory induction/repression, flat)
- Simulated bulk negative-binomial count matrix over genes, time points, and replicates
- Regulator profile table with activator (same-pattern) and repressor (opposite-pattern) assignments

## How to apply

Load a seed bulk RNA-seq sample and estimate per-gene mean and dispersion parameters for negative-binomial modelling (SPARSim-style parameterization via R stats random functions). Randomly select a subset of features as DEGs and label each profile according to the supplied design parameters: continuous induction or repression (scaled fold change), transitory induction or repression, or flat. Draw negative-binomial counts for every gene, time point, and replicate from the estimated mean/dispersion, then apply the profile-specific fold changes to base counts for flat and two-group designs. Derive regulator profiles afterwards by setting each activator's pattern identical to its target genes' pattern and each repressor's pattern to the mirrored/opposite pattern (e.g., continuous induction vs continuous repression).

## Related tools

- **MOSim** (R package whose mosim function implements the bulk simulation, DEG profile labelling, and activator/repressor profile coupling) — https://github.com/ConesaLab/MOSim
- **R** (Language in which the negative-binomial count draws and profile-specific fold changes are computed via stats random functions)
- **SPARSim** (Source of the count-data parameterization style used for the negative-binomial simulation of gene counts)

## Examples

```
library(MOSim); mosim(omics = "RNA-seq", RNAseq = seed_counts, times = c(0, 12, 24, 36), diff.time = c(12, 24), reps = 3)
```

## Evaluation signals

- Verify simulated counts are consistent with the seed sample's estimated mean and dispersion parameters
- Verify regulator profiles correctly correspond to same-pattern (activator) or opposite-pattern (repressor) relationships with their assigned target genes
- Apply k-means clustering to simulated feature profiles and check that recovered clusters match the assigned temporal expression patterns
- Confirm clustering alignment with the simulation settings via PCA on log(x+1)-transformed data

## Limitations

- The paper reports no quantitative evaluation of the profile-labelling mechanism in isolation, so it must be accepted qualitatively
- Regulator omics layers depend on the gene-expression labels; profile assignment errors propagate to ATAC-seq, ChIP-seq, miRNA-seq, Methyl-seq, and TF layers
- MOSim cannot simulate GRNs with spatial resolution and is not applicable to proteomics or metabolomics layers

## Evidence

- [other] Randomly select a subset of features as differentially expressed genes and assign each a label: continuous induction/repression (scaled fold change), transitory induction/repression, or flat profile: [evidence span withheld — anchor status: not_found]
- [other] regulators labelled as activators are simulated with the same temporal profile as their associated genes, while repressors follow the opposite pattern: [evidence span withheld — anchor status: not_found]
