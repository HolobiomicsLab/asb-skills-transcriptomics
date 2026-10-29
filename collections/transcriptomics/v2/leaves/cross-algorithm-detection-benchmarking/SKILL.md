---
name: cross-algorithm-detection-benchmarking
description: Use when you need to quantitatively compare signature-detection performance
  of multiple fitting algorithms against ground-truth contributions, e.g., validating
  a new error-suppression rule (bootstrap 95% CI) against fixed contribution thresholds.
when_to_use_negative:
- Catalogs with fewer than ~500 mutations (e.g., targeted capture panels covering
  a few megabases), where mutational burdens are too low for meaningful individual-patient
  signature fitting and benchmark conclusions do not transfer
- Whole-exome data where the expected signature profile in coding versus non-coding
  regions differs substantially, complicating ground-truth comparisons
- De novo signature discovery from large cohorts — this skill benchmarks fitting against
  known reference signatures, not extraction
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0199
tools:
- name: mmsig
  role: Primary fitting algorithm under benchmark; dynamic error-suppression via non-zero
    95% bootstrap CI (mm_fit_signatures, bootstrap=TRUE, cos_sim_threshold=0.01)
  repo: https://github.com/evenrus/mmsig
- name: deconstructSigs
  role: Comparator algorithm using a hard 6% contribution threshold for calling signatures
    present
- name: MutationalPatterns
  role: Comparator algorithm fitted both without filters and with standard filters
    applied
- name: R
  role: Implementation environment; mmsig installed via devtools::install_github("evenrus/mmsig")
- name: ggplot2
  role: Plotting sensitivity-versus-contribution benchmark curves
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:5ed3a9d5b3af3929506071ecc331c7868de3e769f0000066d1050292a19d82ec
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# cross-algorithm-detection-benchmarking

## Summary

Benchmarking the detection sensitivity and specificity of mutational signature fitting algorithms (mmsig, deconstructSigs, MutationalPatterns) on simulated dilution catalogs with known signature contributions, to compare a dynamic error-suppression approach against hard threshold-based calling.

## When to use

Use when you need to quantitatively compare signature-detection performance of multiple fitting algorithms against ground-truth contributions, e.g., validating a new error-suppression rule (bootstrap 95% CI) against fixed contribution thresholds.

## When NOT to use

- Catalogs with fewer than ~500 mutations (e.g., targeted capture panels covering a few megabases), where mutational burdens are too low for meaningful individual-patient signature fitting and benchmark conclusions do not transfer
- Whole-exome data where the expected signature profile in coding versus non-coding regions differs substantially, complicating ground-truth comparisons
- De novo signature discovery from large cohorts — this skill benchmarks fitting against known reference signatures, not extraction

## Inputs

- Simulated dilution SNV mutational catalogs (>500 mutations each) with known SBS2/SBS-MM1 contributions
- COSMIC v3.1 reference signature matrix (mmsig data(signature_ref), with 'sub' and 'tri' columns plus signature probability columns)
- 5-column VCF-like mutation table (sample, chr, pos, ref, alt), e.g., mmsig data(mm_5_col)

## Outputs

- Per-algorithm sensitivity and specificity values for SBS2 and SBS-MM1 at each dilution level
- Sensitivity-versus-contribution curves for all algorithms (e.g., ggplot2, reproducing Fig 6A/B)
- Bootstrap CI estimates from mm_fit_signatures used for presence/absence calls

## How to apply

Generate simulated dilution mutational catalogs from the COSMIC v3.1 reference signature matrix (signature_ref) with known SBS2 and SBS-MM1 contributions spanning a dilution series and more than 500 mutations per catalog. Fit each catalog with mmsig's mm_fit_signatures (bootstrap=TRUE, 1000 iterations for stable results, cos_sim_threshold=0.01, force_include=c('SBS1','SBS5')) and call presence via the non-zero 95% bootstrap CI error-suppression rule; the rationale is that this dynamic threshold adapts to signature distinctiveness rather than imposing a single cutoff. Fit the same catalogs with deconstructSigs applying its standard 6% contribution threshold, and with MutationalPatterns both without and with standard filters. Score per-algorithm sensitivity and specificity for SBS2 and SBS-MM1 at each dilution level against ground truth, then plot sensitivity versus contribution curves (e.g., ggplot2, reproducing Fig 6A/B).

## Related tools

- **mmsig** (Primary fitting algorithm under benchmark; dynamic error-suppression via non-zero 95% bootstrap CI (mm_fit_signatures, bootstrap=TRUE, cos_sim_threshold=0.01)) — https://github.com/evenrus/mmsig
- **deconstructSigs** (Comparator algorithm using a hard 6% contribution threshold for calling signatures present)
- **MutationalPatterns** (Comparator algorithm fitted both without filters and with standard filters applied)
- **R** (Implementation environment; mmsig installed via devtools::install_github("evenrus/mmsig"))
- **ggplot2** (Plotting sensitivity-versus-contribution benchmark curves)

## Examples

```
sig_out <- mm_fit_signatures(muts.input=mm_5_col_subset, sig.input=sig_ref, input.format="vcf", bootstrap=TRUE, iterations=1000, refcheck=TRUE, cos_sim_threshold=0.01, force_include=c("SBS1","SBS5"))
```

## Evaluation signals

- mmsig sensitivity reaches 100% for the distinctive SBS2 at 4% contribution
- mmsig sensitivity is >95% for the less well-defined SBS-MM1 at 12% contribution
- Threshold-based algorithms (mutationalPatterns, deconstructSigs) center around 6% contribution with >95% sensitivity only at 7–8% contribution
- Relying only on mmsig's built-in error-suppression yields excellent sensitivity and specificity, particularly for catalogs of more than 500 mutations
- Per-dilution sensitivity/specificity scores match ground-truth contributions used to generate the simulated catalogs

## Limitations

- Flat mutational signatures may be falsely removed by mmsig because all mutations can be re-classified to another signature with relatively low cosine similarity
- Different mutational processes occupy largely the same feature space, so mutational signature analysis is subject to a degree of uncertainty; hard 6% cutoffs are inappropriately strict in such cases
- Accuracy is optimal only for catalogs of more than 500 mutations; targeted panels often yield too low mutational burdens for individual-patient fitting

## Evidence

- [results] mmsig sensitivity 100% SBS2 4%; >95% SBS-MM1 12%; thresholds 6–8%: [evidence span withheld — anchor status: not_found]
- [discussion] Error-suppression hard cut-offs 6% deconstructSigs: "Error-suppression using hard cut-offs, such as the standard 6% threshold imposed by deconstructSigs, will be inappropriately strict in these cases."
- [discussion] non-zero 95% CI improves specificity: "requiring non-zero 95% CI to call a mutational signature as present was highly effective at improving specificity"
- [methods] mm_fit_signatures parameters: "`mm_fit_signatures(muts.input, sig.input, input.format, sample.sigt.profs, bootstrap, iterations, strandbias, refcheck, cos_sim_threshold, force_include, dbg)`"
- [discussion] excellent sensitivity specificity >500 mutations: "Relying only on the error-suppression built into mmsig resulted in excellent sensitivity and specificity, particularly when applied to catalogs of more than 500 mutations"
- [discussion] flat signatures falsely removed: "flat mutational signatures, which may be falsely removed by mmsig because all the mutations can be re-classified to another mutational signature with a relatively low cos"
