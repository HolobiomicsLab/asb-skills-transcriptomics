---
name: bootstrap-confidence-interval-estimation
description: Use when fitting COSMIC v3.1 reference signatures (e.g., SBS1/SBS2/SBS5/SBS8/SBS9/SBS13/SBS18/SBS-MM1)
  to WGS SNV catalogs in 5-column format and you must decide whether low-contribution
  signatures are truly present versus fitting noise.
when_to_use_negative:
- Catalogs with fewer than ~500 mutations (e.g., targeted capture panels covering
  a few megabases) — mutational burdens are too low for meaningful per-patient signature
  fitting and interval estimation
- Whole-exome data where coding/non-coding signature profiles differ from the reference
  used for fitting
- De novo signature extraction (NMF) workflows — this skill applies to the reference-based
  fitting step only
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: mmsig
  role: Provides mm_fit_signatures with bootstrap resampling, bootSigsPlot for CI
    visualization, and strand-bias testing used to call signature presence
  repo: https://github.com/evenrus/mmsig
- name: devtools
  role: Used to install the mmsig development version from GitHub
  repo: https://github.com/evenrus/mmsig
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Reference genome loaded by mmsig for refcheck and transcriptional strand-bias
    computation
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_006
  - task_007
  - task_005
  - task_003
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:a54ca3b42b1d358540e4e0d60064fbfba80c88937014b25600a70d1cd0cd92b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/evenrus/mmsig
  tools:
  - BSgenome
  - BSgenome.Hsapiens.UCSC.hg19
  - MutationalPatterns
  - R
  - deconstructSigs
  - devtools
  - ggplot2
  - mmsig
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# bootstrap-confidence-interval-estimation

## Summary

Estimates 95% confidence intervals for mutational signature contributions by bootstrapping resampling of SNV catalogs within mmsig's mm_fit_signatures, and uses a non-zero 95% CI as the dynamic error-suppression criterion for calling a signature present.

## When to use

Use when fitting COSMIC v3.1 reference signatures (e.g., SBS1/SBS2/SBS5/SBS8/SBS9/SBS13/SBS18/SBS-MM1) to WGS SNV catalogs in 5-column format and you must decide whether low-contribution signatures are truly present versus fitting noise.

## When NOT to use

- Catalogs with fewer than ~500 mutations (e.g., targeted capture panels covering a few megabases) — mutational burdens are too low for meaningful per-patient signature fitting and interval estimation
- Whole-exome data where coding/non-coding signature profiles differ from the reference used for fitting
- De novo signature extraction (NMF) workflows — this skill applies to the reference-based fitting step only

## Inputs

- SNV mutation catalog in 5-column VCF format (sample, chr, pos, ref, alt), e.g., the mm_5_col dataset
- Signature reference matrix in 96-channel trinucleotide format, e.g., mmsig signature_ref (COSMIC v3.1) subset to context-appropriate signatures
- 96-channel class definitions, e.g., mm_96_classes
- Reference genome for refcheck/strand bias, e.g., BSgenome.Hsapiens.UCSC.hg19

## Outputs

- sig_out$bootstrap: per-sample, per-signature bootstrap contribution distributions
- 95% CIs computed as the 2.5th and 97.5th bootstrap percentiles per signature per sample
- Signature presence/absence calls based on the non-zero 95% CI criterion
- bootSigsPlot visualization of bootstrap confidence intervals per sample
- sig_out$strand_bias_mm1 strand-bias table with MM1_flag supporting presence decisions

## How to apply

Run mm_fit_signatures with bootstrap=TRUE and iterations=1000 (the recommended count for stable results; 20 is acceptable only for quick demos), set.seed(1) for reproducibility, together with cos_sim_threshold=0.01 and force_include=c('SBS1','SBS5') so ubiquitous signatures are never suppressed. For each sample and signature, extract the per-signature bootstrap contribution distributions from sig_out$bootstrap and compute 95% CIs as the 2.5th and 97.5th percentiles. Call a signature present only when its bootstrapped 95% CI excludes zero — this dynamic error-suppression rule adapts to signature distinctiveness (e.g., calling distinctive SBS2 down to 4% contribution) without the hard 6% exposure cutoff used by deconstructSigs. Visualize the intervals per sample with bootSigsPlot and, for transcription-coupled signatures like SBS-MM1 or SBS35, corroborate presence with the significant transcriptional strand bias test (Poisson test on transcribed-vs-untranscribed counts) in sig_out$strand_bias_mm1. On simulated genomes with known contributions, the 95% CI should enclose the true contribution ~94% of the time, which calibrates interval reliability.

## Related tools

- **mmsig** (Provides mm_fit_signatures with bootstrap resampling, bootSigsPlot for CI visualization, and strand-bias testing used to call signature presence) — https://github.com/evenrus/mmsig
- **devtools** (Used to install the mmsig development version from GitHub) — https://github.com/evenrus/mmsig
- **BSgenome.Hsapiens.UCSC.hg19** (Reference genome loaded by mmsig for refcheck and transcriptional strand-bias computation)

## Examples

```
sig_out <- mm_fit_signatures(muts.input=mm_5_col_subset, sig.input=sig_ref, input.format="vcf", bootstrap=TRUE, iterations=1000, refcheck=TRUE, cos_sim_threshold=0.01, force_include=c("SBS1","SBS5")); bootSigsPlot(sig_out$bootstrap)
```

## Evaluation signals

- Signatures are called present only when the bootstrap 95% CI is non-zero — no presence call should be made from a CI enclosing zero
- On simulated catalogs with known contributions, aggregated 95% CI coverage should be approximately 0.94
- CIs are computed as the 2.5th and 97.5th percentiles of sig_out$bootstrap distributions
- Sanity check against reported benchmarks: SBS9 in CLL1078 estimated at 30% with 95% CI 27–34%; SBS35 false-positive rate in de novo AML ~21% (vs 70% for deconstructSigs' 6% threshold)
- bootSigsPlot renders per-sample intervals; a successful run shows forced SBS1/SBS5 with nonzero intervals across samples

## Limitations

- Flat mutational signatures may be falsely removed because all mutations can be re-classified to another signature with a relatively low cosine similarity distance
- Bootstrap CI coverage is ~94%, not the nominal 95%, so intervals are slightly anti-conservative
- Bootstrapping with many iterations on large datasets significantly increases runtime; 20 iterations (README demo) is insufficient for stable results — 1000 is recommended
- Different mutational processes occupy largely the same feature space, so CI-based presence calls remain subject to inherent uncertainty; strand-bias corroboration is recommended for ambiguous signatures

## Evidence

- [discussion] requiring non-zero 95% CI to call a mutational signature as present was highly effective at improving specificity: "requiring non-zero 95% CI to call a mutational signature as present was highly effective at improving specificity"
- [readme] mmsig applies cosine-similarity dynamic error suppression plus bootstrap CIs and strand-bias assessment: "applies cosine similarities for dynamic error suppression as well as bootstrapping-based confidence intervals and assessment of transcriptional strand bias"
- [readme] README recommends 1000 iterations for stable bootstrap results: "iterations = 20, # 1000 iterations recommended for stable results"
- [methods] Forcing ubiquitous signatures prevents their erroneous suppression: "`mm_fit_signatures(muts.input, sig.input, input.format, sample.sigt.profs, bootstrap, iterations, strandbias, refcheck, cos_sim_threshold, force_include, dbg)`"
- [results] Bootstrapped 95% CI calibration on simulated genomes with known contributions: [evidence span withheld — anchor status: not_found]
