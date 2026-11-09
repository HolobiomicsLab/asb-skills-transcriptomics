---
name: sensitivity-specificity-curve-visualization
description: Use when comparing how reliably signature-fitting algorithms detect low-contribution
  signatures in simulated catalogs with known ground-truth contributions, and you
  need sensitivity/specificity curves per algorithm.
when_to_use_negative:
- The catalogs have unknown or non-simulated signature contributions, so no ground
  truth exists to score sensitivity/specificity against
- Mutational catalogs are too small for meaningful fitting — targeted capture panels
  covering a few megabases yield too low mutational burdens for individual-patient
  signature fitting
- The goal is de novo signature discovery rather than benchmarking reference-signature
  fitting performance
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_0092
tools:
- name: mmsig
  role: Fitting algorithm under evaluation; its bootstrap 95% CI error-suppression
    rule defines signature presence calls for the sensitivity/specificity scoring
  repo: https://github.com/evenrus/mmsig
- name: deconstructSigs
  role: Threshold-based comparison algorithm called present at its standard 6% contribution
    cut-off
- name: MutationalPatterns
  role: Threshold-based comparison algorithm fitted both with and without standard
    filters for curve comparison
- name: ggplot2
  role: Used to plot sensitivity versus contribution curves reproducing Fig 6A/B
- name: R
  role: Implementation environment for simulation, fitting, scoring, and plotting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:5db3411736d15f8fc4839c79f34af8fa6e347f0ff507b4875863cf178e5edddf
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# sensitivity-specificity-curve-visualization

## Summary

Benchmarking visualization skill that scores mutational-signature detection algorithms (mmsig, deconstructSigs, MutationalPatterns) for sensitivity and specificity against ground-truth dilution series and plots sensitivity versus contribution curves (e.g., Fig 6A/B) to reveal the minimum detectable contribution of each signature.

## When to use

Use when comparing how reliably signature-fitting algorithms detect low-contribution signatures in simulated catalogs with known ground-truth contributions, and you need sensitivity/specificity curves per algorithm.

## When NOT to use

- The catalogs have unknown or non-simulated signature contributions, so no ground truth exists to score sensitivity/specificity against
- Mutational catalogs are too small for meaningful fitting — targeted capture panels covering a few megabases yield too low mutational burdens for individual-patient signature fitting
- The goal is de novo signature discovery rather than benchmarking reference-signature fitting performance

## Inputs

- Simulated dilution mutational catalogs (>500 mutations each) with known SBS2 and SBS-MM1 ground-truth contributions derived from the COSMIC v3.1 signature_ref matrix
- COSMIC v3.1 reference signature matrix (signature_ref) with sub/tri context columns and signature probability columns
- Per-catalog signature fitting outputs from mmsig (bootstrap estimates), deconstructSigs, and MutationalPatterns

## Outputs

- Per-algorithm sensitivity and specificity scores for SBS2 and SBS-MM1 at each dilution/contribution level
- Sensitivity versus contribution curves for all algorithms (reproducing Fig 6A/B), plotted with ggplot2
- Detected minimum contribution level at which each algorithm reaches >95% sensitivity per signature

## How to apply

Generate simulated dilution mutational catalogs from the COSMIC v3.1 reference matrix (signature_ref) with known SBS2 and SBS-MM1 contributions spanning a dilution series and more than 500 mutations per catalog. Fit each catalog with mmsig via mm_fit_signatures (bootstrap=TRUE, 1000 iterations for stable results, cos_sim_threshold=0.01, force_include=c('SBS1','SBS5')) and call signature presence using the non-zero 95% bootstrap CI error-suppression rule. Fit the same catalogs with deconstructSigs using its standard 6% contribution threshold, and with MutationalPatterns both with and without standard filters. Score per-algorithm sensitivity and specificity of SBS2 and SBS-MM1 detection at each dilution level against ground truth, then plot sensitivity versus contribution curves with ggplot2. The rationale is that mmsig's dynamic error suppression achieves 100% sensitivity for the distinctive SBS2 at 4% contribution and >95% for SBS-MM1 at 12%, whereas threshold-based algorithms center around 6% contribution with >95% sensitivity only at 7–8%.

## Related tools

- **mmsig** (Fitting algorithm under evaluation; its bootstrap 95% CI error-suppression rule defines signature presence calls for the sensitivity/specificity scoring) — https://github.com/evenrus/mmsig
- **deconstructSigs** (Threshold-based comparison algorithm called present at its standard 6% contribution cut-off)
- **MutationalPatterns** (Threshold-based comparison algorithm fitted both with and without standard filters for curve comparison)
- **ggplot2** (Used to plot sensitivity versus contribution curves reproducing Fig 6A/B)
- **R** (Implementation environment for simulation, fitting, scoring, and plotting)

## Examples

```
sig_out <- mm_fit_signatures(muts.input=sim_catalogs, sig.input=sig_ref, input.format='vcf', bootstrap=TRUE, iterations=1000, cos_sim_threshold=0.01, force_include=c('SBS1','SBS5'))
```

## Evaluation signals

- mmsig sensitivity reaches 100% for SBS2 at 4% contribution in the simulated dilution series
- mmsig sensitivity exceeds 95% for the less well-defined SBS-MM1 at 12% contribution
- Threshold-based algorithms (deconstructSigs, MutationalPatterns) center around 6% contribution and only exceed 95% sensitivity at 7–8%
- Requiring non-zero 95% CI to call a signature present improves specificity without loss of sensitivity for distinctive signatures like SBS2
- Curves reproduce the published Fig 6A/B pattern when simulated catalogs contain more than 500 mutations

## Limitations

- Flat mutational signatures may be falsely removed by mmsig because all mutations can be re-classified to another signature with relatively low cosine similarity
- Sensitivity varies by signature distinctiveness: less well-defined signatures like SBS-MM1 require higher contributions (~12%) for reliable detection
- Fitting accuracy is optimal only for catalogs of more than 500 mutations; whole-exome sequencing profiles can differ between coding and non-coding regions

## Evidence

- [results] mmsig sensitivity for distinctive vs less-defined signatures: [evidence span withheld — anchor status: not_found]
- [results] threshold-based algorithm performance: [evidence span withheld — anchor status: not_found]
- [discussion] error-suppression mechanism: "requiring non-zero 95% CI to call a mutational signature as present was highly effective at improving specificity"
- [discussion] catalog size requirement: [evidence span withheld — anchor status: not_found]
- [discussion] mmsig calling rationale: "this dynamic threshold means that mmsig is able to call low contribution of APOBEC with high accuracy, while avoiding extensive false positive calls of other (less distinctive) mutational"
