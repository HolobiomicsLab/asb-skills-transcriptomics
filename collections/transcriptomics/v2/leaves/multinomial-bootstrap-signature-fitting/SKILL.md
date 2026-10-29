---
name: multinomial-bootstrap-signature-fitting
description: Use when fitting reference signatures (e.g. COSMIC v3.1) to individual
  hematological cancer genomes with ≥500 mutations and you need contribution estimates
  with 95% bootstrap confidence intervals and error suppression.
when_to_use_negative:
- Targeted capture panels covering only a few megabases yield too low mutational burdens
  for meaningful per-patient signature fitting.
- The goal is de novo discovery of novel signatures — this skill fits a fixed reference
  set (e.g. COSMIC v3.1) rather than extracting new signatures by NMF.
- Catalogs with fewer than ~500 mutations, where fitting accuracy (including APOBEC
  SBS2/SBS13 discrimination) degrades.
edam_operation: http://edamontology.org/operation_3778
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: mmsig
  role: R package providing EM-based signature fitting (mm_fit_signatures), dynamic
    error suppression via cosine similarity, bootstrap 95% CIs, refCheck, and bootSigsPlot
    QC visualization
  repo: https://github.com/evenrus/mmsig
- name: devtools
  role: Used to install the mmsig development version from GitHub (devtools::install_github('evenrus/mmsig'))
  repo: https://github.com/evenrus/mmsig
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Required dependency for trinucleotide-context assignment and refCheck of reference
    alternatives against the genome
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:fb7fff4e133858beada6b13cff3543df20c8b442562de286351faf04c2ad8072
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/evenrus/mmsig
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - R
  - devtools
  - mmsig
  edam_operation: http://edamontology.org/operation_3778
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# multinomial-bootstrap-signature-fitting

## Summary

Fit reference mutational signatures (e.g. COSMIC SBS signatures) to a tumor's 96-channel trinucleotide catalog with mmsig's expectation-maximization fitting plus bootstrapping to obtain 95% confidence intervals for each signature's contribution, supporting presence/absence calls via non-zero CIs.

## When to use

Use when fitting reference signatures (e.g. COSMIC v3.1) to individual hematological cancer genomes with ≥500 mutations and you need contribution estimates with 95% bootstrap confidence intervals and error suppression.

## When NOT to use

- Targeted capture panels covering only a few megabases yield too low mutational burdens for meaningful per-patient signature fitting.
- The goal is de novo discovery of novel signatures — this skill fits a fixed reference set (e.g. COSMIC v3.1) rather than extracting new signatures by NMF.
- Catalogs with fewer than ~500 mutations, where fitting accuracy (including APOBEC SBS2/SBS13 discrimination) degrades.

## Inputs

- Somatic SNV mutation list per sample as a 5-column table (sample, chr, pos, ref, alt; mm_5_col format) with input.format='vcf'
- Pre-computed 96-channel mutation catalog in mm_96_classes trinucleotide format
- Signature reference matrix: signature_ref (COSMIC SBS signatures) subset to sub/tri columns plus chosen SBS signatures

## Outputs

- sig_out$estimate: per-sample maximum-likelihood signature contribution estimates
- sig_out$bootstrap: per-signature bootstrap contribution distributions for deriving 95% confidence intervals
- bootSigsPlot visualization of bootstrap confidence intervals per sample/signature
- sig_out$strand_bias_mm1: transcriptional strand-bias table with transcribed/untranscribed counts, ratio and Poisson p-values

## How to apply

Prepare a 96-channel trinucleotide catalog (or 5-column VCF-like input for mm_5_col-style data) and a signature reference built from the mmsig signature_ref COSMIC matrix subset to relevant signatures (e.g. drop canonical AID SBS84 and platinum SBS35 when not applicable). Run mm_fit_signatures with bootstrap=TRUE and 1000 iterations (the vignette demo uses 20 only for speed), cos_sim_threshold=0.01 for dynamic error suppression, refcheck=TRUE, and optionally force_include signatures such as SBS1/SBS5. The EM fit yields per-signature contribution estimates (sig_out$estimate) and bootstrap distributions (sig_out$bootstrap); compute 95% CIs as the 2.5th and 97.5th percentiles of bootstrap contributions and treat signatures with CIs excluding zero as present — a rule shown to be highly effective at improving specificity. Visually QC the intervals with bootSigsPlot; this calibration achieves ~94% coverage of true contributions in simulated genomes.

## Related tools

- **mmsig** (R package providing EM-based signature fitting (mm_fit_signatures), dynamic error suppression via cosine similarity, bootstrap 95% CIs, refCheck, and bootSigsPlot QC visualization) — https://github.com/evenrus/mmsig
- **devtools** (Used to install the mmsig development version from GitHub (devtools::install_github('evenrus/mmsig'))) — https://github.com/evenrus/mmsig
- **BSgenome.Hsapiens.UCSC.hg19** (Required dependency for trinucleotide-context assignment and refCheck of reference alternatives against the genome)

## Examples

```
sig_out <- mm_fit_signatures(muts.input=mm_5_col_subset, sig.input=sig_ref, input.format='vcf', bootstrap=TRUE, iterations=1000, refcheck=TRUE, cos_sim_threshold=0.01, force_include=c('SBS1','SBS5'), dbg=FALSE)
```

## Evaluation signals

- Coverage calibration: on simulated genomes with known signature contributions, the bootstrapped 95% CI should enclose the true contribution ~94% of the time (paper's simulated-genomes analysis, Results Fig. 6).
- Specificity: requiring a non-zero 95% CI to call a signature present should suppress false positives on less distinctive signatures while still calling low APOBEC (SBS2) contribution accurately.
- Fitted estimate should reproduce the input catalog: bootstrap_fit_signatures internally checks cosine similarity against cos_sim_threshold=0.01, so fitted catalogs with cosine similarity below the threshold are flagged/refit.
- QC plot: bootSigsPlot(sig_out$bootstrap) should show sensible per-signature CI widths without implausibly wide intervals on flat signatures.

## Limitations

- Flat mutational signatures may be falsely removed because all mutations can be re-classified to another signature with relatively low cosine similarity loss.
- With whole-exome sequencing, the expected signature profile in coding and non-coding regions can differ from the reference.
- Accuracy is optimal only for catalogs of more than 500 mutations; fewer mutations reduce fitting reliability.
- Hard cut-offs such as deconstructSigs' standard 6% threshold are inappropriately strict compared with mmsig's dynamic error suppression.

## Evidence

- [readme] mmsig is based on an expectation maximization algorithm for mutational signature fitting and applies cosine similarities for dynamic error suppression as well as bootstrapping-based confidence intervals: "mmsig is based on an expectation maximization algorithm for mutational signature fitting and applies cosine similarities for dynamic error suppression as well as bootstrapping-based confidence"
- [methods] mm_fit_signatures call signature: "`mm_fit_signatures(muts.input, sig.input, input.format, sample.sigt.profs, bootstrap, iterations, strandbias, refcheck, cos_sim_threshold, force_include, dbg)`"
- [readme] 1000 iterations recommended for stable results: "iterations = 20, # 1000 iterations recommended for stable results"
- [discussion] non-zero 95% CI rule for specificity: "requiring non-zero 95% CI to call a mutational signature as present was highly effective at improving specificity"
- [discussion] accuracy optimal above 500 mutations: "The accuracy of mutational signature fitting, in general, is optimal when applied to catalogs of more than 500 mutations, which can be consistently obtained by WGS"
