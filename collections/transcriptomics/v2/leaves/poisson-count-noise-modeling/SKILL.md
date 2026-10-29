---
name: poisson-count-noise-modeling
description: Use when benchmarking mutational signature fitting algorithms and you
  need simulated mutation count catalogs that reflect realistic sampling stochasticity
  rather than exact multinomial expectations.
when_to_use_negative:
- You need de novo signature extraction from real tumor catalogs rather than simulated
  benchmarking data with known ground-truth admixtures
- Targeted capture panel data with too low a mutational burden — the article notes
  such panels 'often yield too low mutational burdens to allow meaningful mutational
  signature fitting for individual patients'
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0769
tools:
- name: mmsig
  role: R package supplying signature_ref and mm_96_classes data objects used to construct
    the 96-class MM probability profile for simulation
  repo: https://github.com/evenrus/mmsig
- name: R
  role: Language in which the multinomial sampling, Poisson perturbation, and seed-fixed
    simulation are implemented
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:a0f4991d0c1ba35100d1142310dc8c41b9d0ccf133b764dabb0b68bc55b5ca2f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/evenrus/mmsig
  tools:
  - R
  - mmsig
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0769
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# poisson-count-noise-modeling

## Summary

Generates Poisson-noise perturbed variants of simulated mutational catalogs to mimic count stochasticity when benchmarking signature fitting sensitivity. In the mmsig validation, 100 simulated 5000-mutation MM catalogs per dilution level (SBS2 1–10%, SBS-MM1 1–20%) were perturbed with Poisson noise before being written to simulated_catalogs.csv.

## When to use

Use when benchmarking mutational signature fitting algorithms and you need simulated mutation count catalogs that reflect realistic sampling stochasticity rather than exact multinomial expectations.

## When NOT to use

- You need de novo signature extraction from real tumor catalogs rather than simulated benchmarking data with known ground-truth admixtures
- Targeted capture panel data with too low a mutational burden — the article notes such panels 'often yield too low mutational burdens to allow meaningful mutational signature fitting for individual patients'

## Inputs

- mmsig data object signature_ref (COSMIC v3.1 SBS reference profiles with sub/tri columns)
- mmsig data object mm_96_classes (96 trinucleotide class definitions)
- Dilution specifications: SBS2 1–10%, SBS-MM1 1–20% admixed into the average MM background profile

## Outputs

- simulated_catalogs.csv containing the dilution-series catalogs and their Poisson-noise perturbed variants
- 100 simulated catalogs of 5000 mutations per dilution level (plus Poisson-noise variants)

## How to apply

Build the base 96-trinucleotide probability profile by weighting the reference signatures at their median MM contributions (SBS1 4.8%, SBS2 5.1%, SBS5 34.1%, SBS8 18.2%, SBS9 18.9%, SBS-MM1 18.8%) using the signature_ref and mm_96_classes objects from mmsig. For each dilution level (SBS2 re-weighted across 1–10%, SBS-MM1 across 1–20%), draw 100 multinomial samples of 5000 mutations with set.seed(1) for reproducibility. Then perturb each catalog with Poisson noise to mimic count stochasticity, producing noisy variants alongside the exact catalogs. Validate that each catalog sums to exactly 5000 mutations and that realized SBS2/SBS-MM1 fractions match the specified dilutions within sampling tolerance.

## Related tools

- **mmsig** (R package supplying signature_ref and mm_96_classes data objects used to construct the 96-class MM probability profile for simulation) — https://github.com/evenrus/mmsig
- **R** (Language in which the multinomial sampling, Poisson perturbation, and seed-fixed simulation are implemented)

## Examples

```
set.seed(1); sims <- t(apply(replicate(100, rmultinom(1, 5000, prob = mm_profile)), 1, identity)); noisy <- t(sapply(sims, function(x) rpois(length(x), lambda = x)))
```

## Evaluation signals

- Each simulated catalog sums to exactly 5000 mutations
- Realized SBS2 fractions fall within sampling tolerance of the specified 1–10% dilution levels
- Realized SBS-MM1 fractions fall within sampling tolerance of the specified 1–20% dilution levels
- Both the exact and Poisson-noise perturbed variants of each catalog are present in simulated_catalogs.csv
- Re-running with set.seed(1) reproduces the same catalogs

## Limitations

- Poisson perturbation only models count stochasticity; it does not capture sequencing artifacts or systematic biases in real catalogs
- mmsig fitting accuracy is optimal only for catalogs of more than 500 mutations, so simulations below that size may misestimate sensitivity
- Flat mutational signatures may be falsely removed by mmsig due to cosine-similarity re-classification, which affects interpretation of benchmark results for such signatures

## Evidence

- [methods] The simulated genomes were generated as a dilution series admixing 1–10% SBS2 and 1–20% SBS-MM1 into a background resembling an average MM genome: [evidence span withheld — anchor status: not_found]
- [methods] Poisson-noise perturbed variants of each simulated catalog: [evidence span withheld — anchor status: not_found]
- [discussion] excellent sensitivity and specificity with mmsig built-in error-suppression for catalogs of more than 500 mutations: "Relying only on the error-suppression built into mmsig resulted in excellent sensitivity and specificity, particularly when applied to catalogs of more than 500 mutations"
- [discussion] flat mutational signatures may be falsely removed by mmsig: [evidence span withheld — anchor status: not_found]
