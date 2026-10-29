---
name: dilution-catalog-simulation
description: 'Use when you need a controlled benchmark of signature-detection sensitivity/specificity:
  synthetic 96-channel catalogs with known low-percentage signature contributions
  are required to compare fitting algorithms'' detection thresholds.'
when_to_use_negative:
- The goal is de novo signature discovery from a large cohort rather than benchmarking
  reference-signature fitting sensitivity against known ground truth.
- Catalogs have too low a mutational burden (<500 mutations), e.g. from targeted capture
  panels covering a few megabases, where individual-patient signature fitting is not
  meaningful.
- Input is real tumor catalogs with unknown signature composition — the simulation
  requires ground-truth contributions for scoring.
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3676
tools:
- name: mmsig
  role: Fits simulated dilution catalogs with bootstrap-based error suppression to
    measure sensitivity of the non-zero 95% CI presence rule
  repo: https://github.com/evenrus/mmsig
- name: deconstructSigs
  role: Baseline threshold-based fitter applying its standard 6% contribution cut-off
    for calling signatures present
- name: MutationalPatterns
  role: Baseline fitter run both without filters and with standard filters applied
    to the same simulated catalogs
- name: R
  role: Implementation environment for simulation, fitting, and benchmarking
  repo: https://github.com/evenrus/mmsig
- name: ggplot2
  role: Plots sensitivity versus contribution curves for all algorithms
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:1bdbfe4d71ac3282b1c8737662315e52c683b1d43e367a2d64b717d56118f66f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/evenrus/mmsig
  tools:
  - MutationalPatterns
  - R
  - deconstructSigs
  - ggplot2
  - mmsig
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3676
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# dilution-catalog-simulation

## Summary

Generates simulated dilution mutational catalogs (>500 mutations each) from the COSMIC v3.1 reference signature matrix with known SBS2 and SBS-MM1 contributions, enabling ground-truth benchmarking of mutational signature fitting algorithms such as mmsig, deconstructSigs, and MutationalPatterns.

## When to use

Use when you need a controlled benchmark of signature-detection sensitivity/specificity: synthetic 96-channel catalogs with known low-percentage signature contributions are required to compare fitting algorithms' detection thresholds.

## When NOT to use

- The goal is de novo signature discovery from a large cohort rather than benchmarking reference-signature fitting sensitivity against known ground truth.
- Catalogs have too low a mutational burden (<500 mutations), e.g. from targeted capture panels covering a few megabases, where individual-patient signature fitting is not meaningful.
- Input is real tumor catalogs with unknown signature composition — the simulation requires ground-truth contributions for scoring.

## Inputs

- COSMIC v3.1 mutational signature reference matrix (signature_ref, with 'sub' and 'tri' columns and signature columns such as SBS1, SBS2, SBS5, SBS13, SBS-MM1)
- Dilution series specification of ground-truth signature contributions for SBS2 and SBS-MM1
- Simulated 96-channel mutational catalogs (>500 mutations per catalog)

## Outputs

- Per-algorithm sensitivity and specificity scores for SBS2 and SBS-MM1 at each dilution level
- Sensitivity-versus-contribution curves reproducing Fig 6A/B (ggplot2)
- mmsig bootstrap confidence intervals from which non-zero 95% CI presence calls are derived

## How to apply

Simulate dilution catalogs by mixing reference signatures from the COSMIC v3.1 signature_ref matrix, spanning a dilution series of known SBS2 and SBS-MM1 contributions while keeping each catalog above 500 mutations, the level at which fitting accuracy is optimal. Fit each simulated catalog with mmsig via mm_fit_signatures (bootstrap=TRUE, iterations=1000 for stable results, cos_sim_threshold=0.01, force_include=c('SBS1','SBS5')) and call signatures present using the non-zero 95% bootstrap CI error-suppression rule. In parallel, fit the same catalogs with deconstructSigs using its standard 6% contribution threshold and with MutationalPatterns both unfiltered and with standard filters. Score per-algorithm sensitivity and specificity against the ground-truth contributions at each dilution level and plot sensitivity-versus-contribution curves (Fig 6A/B style) with ggplot2. The rationale is that distinctiveness varies by signature: mmsig's dynamic error suppression reached 100% sensitivity for the distinctive SBS2 at 4% contribution and >95% for the less well-defined SBS-MM1 at 12%, versus ~6–8% for threshold-based algorithms.

## Related tools

- **mmsig** (Fits simulated dilution catalogs with bootstrap-based error suppression to measure sensitivity of the non-zero 95% CI presence rule) — https://github.com/evenrus/mmsig
- **deconstructSigs** (Baseline threshold-based fitter applying its standard 6% contribution cut-off for calling signatures present)
- **MutationalPatterns** (Baseline fitter run both without filters and with standard filters applied to the same simulated catalogs)
- **R** (Implementation environment for simulation, fitting, and benchmarking) — https://github.com/evenrus/mmsig
- **ggplot2** (Plots sensitivity versus contribution curves for all algorithms)

## Examples

```
sig_out <- mm_fit_signatures(muts.input=sim_catalogs, sig.input=sig_ref, input.format="vcf", bootstrap=TRUE, iterations=1000, cos_sim_threshold=0.01, force_include=c("SBS1","SBS5"))
```

## Evaluation signals

- mmsig sensitivity for SBS2 reaches 100% at 4% contribution in the dilution series
- mmsig sensitivity for SBS-MM1 exceeds 95% at 12% contribution
- deconstructSigs and MutationalPatterns with standard filters center around 6% contribution, reaching >95% sensitivity only at 7–8%
- Each simulated catalog contains more than 500 mutations
- mmsig fits use bootstrap=TRUE with iterations=1000 and cos_sim_threshold=0.01, and presence is called only when the 95% bootstrap CI excludes zero

## Limitations

- Flat mutational signatures may be falsely removed by mmsig because all mutations can be re-classified to another signature with a relatively low cosine-similarity loss.
- Different mutational processes occupy largely the same feature space, so signature analysis — and hence the sensitivity estimates being benchmarked — is subject to inherent uncertainty.
- Accuracy is optimal only for catalogs of more than 500 mutations, consistently obtainable by WGS; whole-exome data carry the caveat that expected signature profiles in coding and non-coding regions differ.

## Evidence

- [results] After applying standard filtering, mmsig's sensitivity varied by signature distinctiveness: [evidence span withheld — anchor status: not_found]
- [discussion] requiring non-zero 95% CI to call a signature present improved specificity: "requiring non-zero 95% CI to call a mutational signature as present was highly effective at improving specificity"
- [discussion] fitting accuracy optimal with >500 mutations: "The accuracy of mutational signature fitting, in general, is optimal when applied to catalogs of more than 500 mutations, which can be consistently obtained by WGS"
- [discussion] deconstructSigs standard 6% threshold: "Error-suppression using hard cut-offs, such as the standard 6% threshold imposed by deconstructSigs, will be inappropriately strict in these cases."
- [readme] README mm_fit_signatures invocation with bootstrap and force_include: "sig_out <- mm_fit_signatures(muts.input=mm_5_col_subset, sig.input=sig_ref, input.format = "vcf""
