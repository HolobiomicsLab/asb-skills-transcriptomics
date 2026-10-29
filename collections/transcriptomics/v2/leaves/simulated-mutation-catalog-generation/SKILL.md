---
name: simulated-mutation-catalog-generation
description: Use when you need catalogs of known ground-truth signature composition
  (e.g., dilution series of SBS2 1–10% or SBS-MM1 1–20% into an average MM background)
  to benchmark signature fitting or bootstrap CI coverage.
when_to_use_negative:
- You have real patient mutation catalogs and need signature fitting rather than ground-truth
  simulation — use mm_fit_signatures directly on the VCF/5-column input instead
- Your analysis requires de novo signature extraction from a cohort (NMF) rather than
  fitting against COSMIC v3.1 reference profiles
- Catalogs have fewer than ~500 mutations, where the article notes fitting accuracy
  is not optimal
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: mmsig
  role: Provides signature_ref (COSMIC v3.1 profiles) and mm_96_classes used as the
    sampling basis for simulated catalogs, and mm_fit_signatures/bootstrap for downstream
    coverage evaluation
  repo: https://github.com/evenrus/mmsig
- name: R
  role: Runtime environment for multinomial sampling of simulated catalogs and mmsig
    execution
- name: devtools
  role: Used to install the mmsig package from GitHub
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:d847f5ece2010542868c90708eb69f57c0dd110a3f94ff9734383ce88f5b2025
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/evenrus/mmsig
  tools:
  - R
  - devtools
  - mmsig
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# simulated-mutation-catalog-generation

## Summary

Generates simulated 96-channel trinucleotide mutation catalogs with known signature contributions by multinomial sampling over mmsig's signature_ref COSMIC v3.1 profiles and mm_96_classes channel definitions, enabling calibration of fitting accuracy and bootstrap CI coverage.

## When to use

Use when you need catalogs of known ground-truth signature composition (e.g., dilution series of SBS2 1–10% or SBS-MM1 1–20% into an average MM background) to benchmark signature fitting or bootstrap CI coverage.

## When NOT to use

- You have real patient mutation catalogs and need signature fitting rather than ground-truth simulation — use mm_fit_signatures directly on the VCF/5-column input instead
- Your analysis requires de novo signature extraction from a cohort (NMF) rather than fitting against COSMIC v3.1 reference profiles
- Catalogs have fewer than ~500 mutations, where the article notes fitting accuracy is not optimal

## Inputs

- mmsig signature_ref data object (COSMIC v3.1 SBS reference profiles with sub/tri columns)
- mmsig mm_96_classes data object (96 trinucleotide class definitions)
- Pre-determined signature contribution vector (e.g., median MM profile: SBS1 4.8%, SBS2 5.1%, SBS5 34.1%, SBS8 18.2%, SBS9 18.9%, SBS-MM1 18.8%)
- Dilution-series specification (SBS2 1–10%, SBS-MM1 1–20%) and sample size per catalog (e.g., 5000 mutations)

## Outputs

- Simulated 96-channel mutation catalogs with known signature contributions (e.g., simulated_catalogs.csv, each summing to exactly 5000 mutations)
- Poisson-noise perturbed variants of each simulated catalog
- Ground-truth contribution labels per catalog and signature for downstream coverage/sensitivity evaluation

## How to apply

Load mmsig's signature_ref (COSMIC v3.1 SBS profiles with sub/tri columns) and mm_96_classes, then build the average multiple-myeloma 96-class probability profile from SBS1 (4.8%), SBS2 (5.1%), SBS5 (34.1%), SBS8 (18.2%), SBS9 (18.9%) and SBS-MM1 (18.8%). Draw multinomial random samples over the 96 trinucleotide classes (e.g., 100 catalogs of 5000 mutations with set.seed(1) for reproducibility), constructing dilution series by re-weighting SBS2 across 1–10% and SBS-MM1 across 1–20% within the base profile before sampling. Optionally generate Poisson-noise perturbed variants to mimic count stochasticity, and write catalogs to simulated_catalogs.csv. The known pre-determined contributions then serve as ground truth for coverage tests, e.g., checking whether mmsig's bootstrapped 95% CIs (2.5th/97.5th percentiles from sig_out$bootstrap at 1000 iterations, cos_sim_threshold=0.01) enclose the true contribution ~94% of the time.

## Related tools

- **mmsig** (Provides signature_ref (COSMIC v3.1 profiles) and mm_96_classes used as the sampling basis for simulated catalogs, and mm_fit_signatures/bootstrap for downstream coverage evaluation) — https://github.com/evenrus/mmsig
- **R** (Runtime environment for multinomial sampling of simulated catalogs and mmsig execution)
- **devtools** (Used to install the mmsig package from GitHub)

## Examples

```
set.seed(1); sim <- rmultinom(100, 5000, prob = base_mm_profile); sim_catalogs <- data.frame(mm_96_classes, sim)
```

## Evaluation signals

- Each simulated catalog sums to exactly 5000 mutations (or the specified catalog size)
- Realized SBS2/SBS-MM1 fractions match the specified dilution levels (1–10% and 1–20%) within multinomial sampling tolerance
- Fitting simulated catalogs with bootstrap=TRUE, iterations=1000, cos_sim_threshold=0.01, refcheck=TRUE yields ~0.94 aggregated 95% CI coverage of the true contributions (paper's Results Fig. 6)
- bootSigsPlot on representative catalogs shows bootstrap CI distributions consistent with the pre-determined contributions (QC check)

## Limitations

- Multinomial sampling without Poisson perturbation omits count stochasticity; both plain and Poisson-noise variants were generated to mimic real catalogs
- Simulations assume the COSMIC v3.1 reference profiles and a fixed MM background, so results reflect within-reference-set behavior and may not generalize to novel signatures
- mmsig's dynamic error-suppression (cos_sim_threshold) can falsely remove flat signatures, affecting coverage for low-distinctiveness signatures
- Accuracy of signature fitting is optimal only for catalogs of more than 500 mutations, consistently obtainable by WGS; targeted panels of a few megabases yield too low mutational burdens

## Evidence

- [other] Build the average multiple-myeloma 96-class probability profile by combining SBS1 (4.8%), SBS2 (5.1%), SBS5 (34.1%), SBS8 (18.2%), SBS9 (18.9%) and SBS-MM1 (18.8%) at their stated median contributions.: [evidence span withheld — anchor status: not_found]
- [methods] Fit simulated catalogs with mmsig bootstrap to test CI coverage: "`mm_fit_signatures(muts.input, sig.input, input.format, sample.sigt.profs, bootstrap, iterations, strandbias, refcheck, cos_sim_threshold, force_include, dbg)`"
- [readme] Install mmsig from GitHub with devtools: "devtools::install_github("evenrus/mmsig")"
- [discussion] Catalog size requirement for accurate fitting: "The accuracy of mutational signature fitting, in general, is optimal when applied to catalogs of more than 500 mutations, which can be consistently obtained by WGS"
- [discussion] Flat signatures may be falsely removed by mmsig's error suppression: "flat mutational signatures, which may be falsely removed by mmsig because all the mutations can be re-classified to another mutational signature with a relatively low cos"
