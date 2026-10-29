---
name: signature-presence-calling-from-nonzero-ci
description: Use when fitting reference mutational signatures to individual cancer
  samples (especially hematological malignancies with low-burden or overlapping processes
  like SBS9/AID) and you need a specific presence/absence call rather than a point
  estimate.
when_to_use_negative:
- Data from targeted capture panels covering only a few megabases — mutational burden
  is too low for meaningful individual-patient signature fitting
- Catalogs of fewer than ~500 mutations, where fitting accuracy (sensitivity/specificity
  of error suppression) is no longer optimal
- Whole-exome data where the expected signature profile in coding versus non-coding
  regions differs substantially from the genome-wide reference, and flat signatures
  may be falsely removed
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_2640
tools:
- name: mmsig
  role: 'Core R package: EM-based signature fitting with dynamic cosine-similarity
    error suppression, bootstrapped 95% CIs, and strand-bias assessment; provides
    mm_fit_signatures, bootSigsPlot, plot_signatures'
  repo: https://github.com/evenrus/mmsig
- name: devtools
  role: Used to install the mmsig development version from GitHub
- name: BSgenome.Hsapiens.UCSC.hg19
  role: hg19 reference genome required by mmsig for refCheck validation of SNV reference
    alleles and transcriptional strand-bias analysis
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:c1a9bfedb3af501b69cfe066b08c79d17d41cd14a44a5c8877d90a0f909edae2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/evenrus/mmsig
  tools:
  - BSgenome
  - BSgenome.Hsapiens.UCSC.hg19
  - R
  - devtools
  - mmsig
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_2640
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# signature-presence-calling-from-nonzero-ci

## Summary

Calls a mutational signature 'present' in individual hematological cancer samples by fitting a curated COSMIC v3.1 reference with mmsig (EM-based fitting with dynamic cosine-similarity error suppression) and requiring a non-zero 95% bootstrap confidence interval for the signature's contribution. Used here to detect nc-AID (SBS9) in IGHV-unmutated CLL WGS samples previously thought to lack this process.

## When to use

Use when fitting reference mutational signatures to individual cancer samples (especially hematological malignancies with low-burden or overlapping processes like SBS9/AID) and you need a specific presence/absence call rather than a point estimate.

## When NOT to use

- Data from targeted capture panels covering only a few megabases — mutational burden is too low for meaningful individual-patient signature fitting
- Catalogs of fewer than ~500 mutations, where fitting accuracy (sensitivity/specificity of error suppression) is no longer optimal
- Whole-exome data where the expected signature profile in coding versus non-coding regions differs substantially from the genome-wide reference, and flat signatures may be falsely removed

## Inputs

- Whole-genome (or >500-mutation) SNV catalog in 5-column format: sample, chr, pos, ref, alt (e.g., mm_5_col-style VCF-like data frame, hg19 coordinates)
- COSMIC v3.1 signature reference matrix (signature_ref, 96 trinucleotide classes per signature, subset to context-appropriate signatures)
- BSgenome.Hsapiens.UCSC.hg19 reference genome for refCheck and strand-bias computation

## Outputs

- Per-sample signature contribution estimates (sig_out$estimate) plottable with plot_signatures
- Bootstrap 95% confidence intervals per sample/signature (sig_out$bootstrap) with presence determined by non-zero CI (visualized via bootSigsPlot)
- Transcriptional strand-bias table per sample (sig_out$strand_bias_mm1-style output with transcribed/untranscribed counts, ratio, Poisson p-value)

## How to apply

Build the signature reference from the COSMIC v3.1 signature_ref dataset, removing context-inappropriate signatures — canonical AID (SBS84) for genome-wide analysis and the platinum signature SBS35 for non-platinum-exposed cohorts. Clean the SNV catalog (5-column sample/chr/pos/ref/alt format) with vcfCleaner and refCheck against hg19 (BSgenome.Hsapiens.UCSC.hg19). Run mm_fit_signatures with input.format='vcf', strandbias=TRUE, bootstrap=TRUE, refcheck=TRUE, cos_sim_threshold=0.01, chosen force_include signatures, and a fixed seed (set.seed(1)); use 1000 bootstrap iterations for stable results (the README example uses 20 for speed). Apply the presence criterion — a non-zero 95% CI for the signature's contribution — to the bootstrap output via bootSigsPlot, then render per-sample 96-channel estimates with plot_signatures for visual validation. The rationale is that dynamic error suppression plus the non-zero-CI criterion improves specificity where hard cut-offs (e.g., deconstructSigs' 6% threshold) would be inappropriately strict for biologically important low-abundance signatures.

## Related tools

- **mmsig** (Core R package: EM-based signature fitting with dynamic cosine-similarity error suppression, bootstrapped 95% CIs, and strand-bias assessment; provides mm_fit_signatures, bootSigsPlot, plot_signatures) — https://github.com/evenrus/mmsig
- **devtools** (Used to install the mmsig development version from GitHub)
- **BSgenome.Hsapiens.UCSC.hg19** (hg19 reference genome required by mmsig for refCheck validation of SNV reference alleles and transcriptional strand-bias analysis)

## Examples

```
sig_out <- mm_fit_signatures(muts.input=mm_5_col_subset, sig.input=sig_ref, input.format="vcf", strandbias=TRUE, bootstrap=TRUE, iterations=20, refcheck=TRUE, cos_sim_threshold=0.01, force_include=c("SBS1","SBS5"), dbg=FALSE)
```

## Evaluation signals

- Presence criterion: a signature is called present only when its bootstrap 95% CI excludes zero (e.g., SBS9 flagged in exactly three IGHV-unmutated CLL patients; CLL1078 at ~30%, 95% CI 27–34%)
- With set.seed(1) and a fixed iteration count, the bootstrap output is reproducible; repeated runs should give identical CI bounds
- The fitted SBS9 96-channel profile should match the reference both visually and numerically (cf. Fig. 5)
- refCheck returns no mismatches between SNV reference alleles and hg19 after vcfCleaner cleaning
- Context-appropriate signatures (SBS84 canonical AID, SBS35 platinum) are absent from the reference matrix used for genome-wide, non-platinum-exposed samples

## Limitations

- Flat mutational signatures may be falsely removed by mmsig because their mutations can be re-classified to another signature with relatively low cosine similarity
- Whole-exome sequencing caveat: expected signature profiles in coding and non-coding regions can differ, biasing fits relative to genome-wide references
- Targeted capture panels covering a few megabases yield too low a mutational burden for meaningful individual-patient signature fitting
- Accuracy is optimal only with catalogs of more than 500 mutations, consistently obtainable by WGS
- Bootstrapping large datasets with many iterations significantly increases runtime (1000 iterations recommended; README example uses 20 for speed)

## Evidence

- [discussion] Non-zero CI presence criterion: "requiring non-zero 95% CI to call a mutational signature as present was highly effective at improving specificity"
- [discussion] Dynamic error suppression rationale: "Error-suppression using hard cut-offs, such as the standard 6% threshold imposed by deconstructSigs, will be inappropriately strict in these cases."
- [discussion] WGS / >500 mutation requirement: "The accuracy of mutational signature fitting, in general, is optimal when applied to catalogs of more than 500 mutations, which can be consistently obtained by WGS"
- [readme] Reference setup: remove SBS84 and SBS35: "# remove canonical AID (SBS84) for genome-wide analysis
# remove the platinum signature (SBS35) because the patients are not platinum exposed"
- [readme] mm_fit_signatures call with key parameters: "sig_out <- mm_fit_signatures(muts.input=mm_5_col_subset, 
                             sig.input=sig_ref,
                             input.format = "vcf","
- [readme] 1000 bootstrap iterations recommended: "iterations = 20, # 1000 iterations recommended for stable results"
- [readme] Bootstrap CI plotting: "bootSigsPlot(sig_out$bootstrap)"
- [abstract] Low-abundance APOBEC detection claim: "In particular, mmsig allows accurate detection of mutational signatures with low abundance, such as those introduced by APOBEC cytidine deaminases."
- [discussion] Flat signatures may be falsely removed: [evidence span withheld — anchor status: not_found]
