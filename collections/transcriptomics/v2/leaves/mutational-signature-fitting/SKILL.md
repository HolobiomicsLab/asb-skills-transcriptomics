---
name: mutational-signature-fitting
description: Use when assigning reference mutational signatures (e.g. COSMIC v3.1
  SBS1, SBS2, SBS5, SBS8, SBS9, SBS13, SBS18, SBS-MM1) to whole-genome or whole-exome
  SNV catalogs from hematological malignancies and deciding signature presence/absence
  without hard contribution thresholds.
when_to_use_negative:
- Mutational catalogs from targeted capture panels covering only a few megabases,
  which often yield too low mutational burdens for meaningful individual-patient signature
  fitting
- Fitting with a flat signature that mmsig may falsely remove because all its mutations
  can be re-classified to another signature at a relatively low cosine similarity
  threshold
- De novo signature extraction — this skill assumes a curated reference matrix; de
  novo extraction (e.g. NMF) is a separate preceding step
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: mmsig
  role: Core EM-based fitting algorithm with cosine-similarity dynamic error suppression,
    bootstrapping CIs and strand-bias testing (mm_fit_signatures)
  repo: github.com/evenrus/mmsig
- name: devtools
  role: Install the mmsig development version from GitHub
  repo: github.com/evenrus/mmsig
- name: BSgenome.Hsapiens.UCSC.hg19
  role: hg19 reference genome required by mmsig for refCheck and strand-bias computation
- name: deconstructSigs
  role: Comparator threshold-based fitting algorithm (standard 6% contribution cut-off)
    used in benchmarks
- name: MutationalPatterns
  role: Comparator threshold-based fitting algorithm used in sensitivity/specificity
    benchmarks
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_006
  - task_007
  - task_008
  - task_005
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:b11793fe5c5ec60ac7797cf6c8b048d988a9da96e41bfd47a466a5e13600e411
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BSgenome
  - BSgenome.Hsapiens.UCSC.hg19
  - GenomicRanges
  - IRanges
  - MutationalPatterns
  - R
  - deconstructSigs
  - devtools
  - ggplot2
  - mmsig
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# mutational-signature-fitting

## Summary

Fit an input 96-channel SNV catalog against a fixed reference signature matrix (COSMIC v3.1) using mmsig's expectation-maximization fitting with dynamic cosine-similarity error suppression, bootstrapped 95% CIs, and transcriptional strand-bias testing to decide which signatures are truly present.

## When to use

Use when assigning reference mutational signatures (e.g. COSMIC v3.1 SBS1, SBS2, SBS5, SBS8, SBS9, SBS13, SBS18, SBS-MM1) to whole-genome or whole-exome SNV catalogs from hematological malignancies and deciding signature presence/absence without hard contribution thresholds.

## When NOT to use

- Mutational catalogs from targeted capture panels covering only a few megabases, which often yield too low mutational burdens for meaningful individual-patient signature fitting
- Fitting with a flat signature that mmsig may falsely remove because all its mutations can be re-classified to another signature at a relatively low cosine similarity threshold
- De novo signature extraction — this skill assumes a curated reference matrix; de novo extraction (e.g. NMF) is a separate preceding step

## Inputs

- SNV catalog in 5-column VCF-like format (sample, chr, pos, ref, alt), e.g. mmsig example data mm_5_col
- Signature reference matrix (sub, tri, and per-signature probabilities), e.g. mmsig signature_ref built from COSMIC v3.1
- Reference genome package BSgenome.Hsapiens.UCSC.hg19 (for refCheck and strand-bias annotation)

## Outputs

- sig_out$estimate — fitted per-sample signature contribution profiles
- sig_out$bootstrap — bootstrap exposure distributions for 95% CI presence calling (plottable with bootSigsPlot)
- sig_out$strand_bias_mm1 — SBS-MM1 transcribed-vs-untranscribed counts, ratio, Poisson p-value and MM1_flag per sample
- Signature contribution plots from plot_signatures

## How to apply

Install mmsig from GitHub (devtools::install_github('evenrus/mmsig')) and load hg19 via BSgenome.Hsapiens.UCSC.hg19. Set up the signature reference from signature_ref by trimming to hematological-malignancy signatures and removing context-inappropriate ones: drop canonical AID (SBS84) for genome-wide analysis and drop the platinum signature SBS35 when patients are not platinum exposed. Provide SNVs as a 5-column table (sample, chr, pos, ref, alt), optionally cleaned with vcfCleaner and refCheck, then run mm_fit_signatures with input.format='vcf', strandbias=TRUE, bootstrap=TRUE, refcheck=TRUE, cos_sim_threshold=0.01, force_include=c('SBS1','SBS5') and set.seed(1), using 1000 bootstrap iterations for stable results (20 suffices for quick examples). Call a signature present only when its bootstrap 95% CI excludes zero — the dynamic error-suppression rule — rather than applying hard cut-offs like deconstructSigs' standard 6% threshold; confirm transcription-coupled signatures (e.g. SBS-MM1) with significant transcriptional strand bias via sig_out$strand_bias_mm1 (Poisson test). Visualize estimates with plot_signatures (fixed signature order, e.g. SBS1, SBS2, SBS13, SBS5, SBS8, SBS9, SBS18, SBS-MM1) and bootstrap CIs with bootSigsPlot.

## Related tools

- **mmsig** (Core EM-based fitting algorithm with cosine-similarity dynamic error suppression, bootstrapping CIs and strand-bias testing (mm_fit_signatures)) — github.com/evenrus/mmsig
- **devtools** (Install the mmsig development version from GitHub) — github.com/evenrus/mmsig
- **BSgenome.Hsapiens.UCSC.hg19** (hg19 reference genome required by mmsig for refCheck and strand-bias computation)
- **deconstructSigs** (Comparator threshold-based fitting algorithm (standard 6% contribution cut-off) used in benchmarks)
- **MutationalPatterns** (Comparator threshold-based fitting algorithm used in sensitivity/specificity benchmarks)

## Examples

```
sig_out <- mm_fit_signatures(muts.input=mm_5_col_subset, sig.input=sig_ref, input.format="vcf", strandbias=TRUE, bootstrap=TRUE, iterations=1000, refcheck=TRUE, cos_sim_threshold=0.01, force_include=c("SBS1","SBS5"))
```

## Evaluation signals

- Signatures are called present only when the bootstrap 95% CI is non-zero; check that output exposures respect this rule
- In simulated dilution series (>500 mutations per catalog), mmsig sensitivity should reach 100% for the distinctive SBS2 at 4% contribution and >95% for the less distinctive SBS-MM1 at 12% contribution, outperforming 6%-threshold methods that need ~7–8%
- On platinum-naive AML WGS samples, the SBS35 false-positive rate should be ~21% with mmsig (vs ~70% for deconstructSigs), and platinum-exposed t-AML patients should show SBS35 with non-zero CI plus significant strand bias in C[C>A]C and C[C>T]C contexts
- force_include signatures SBS1 and SBS5 appear in every fitted sample; cos_sim_threshold is 0.01 and refcheck=TRUE in the reference call
- Median reconstruction accuracy with error-suppression drops only ~0.005 relative to unfiltered fits while avoiding false positive SBS-MM1 calls

## Limitations

- Flat mutational signatures may be falsely removed by mmsig because all the mutations can be re-classified to another mutational signature with a relatively low cosine similarity
- Whole-exome fitting caveats: the expected signature profile in coding and non-coding regions can differ; e.g. CLL SBS9 exome-based estimates were ~12% lower than paired WGS estimates (paired T-test p<0.001) because nc-AID is predominantly non-coding
- Fitting accuracy is optimal only with catalogs of more than 500 mutations, consistently obtainable by WGS; bootstrapping large datasets with many iterations significantly increases runtime
- The 20-iteration bootstrap in the README example is only for speed; 1000 iterations are recommended for stable results

## Evidence

- [abstract] dynamic error-suppression via bootstrap CI: "Our newly released R package mmsig comes with a dynamic error-suppression procedure that improves specificity in important clinical and biological settings."
- [discussion] hard cut-off caveat: "Error-suppression using hard cut-offs, such as the standard 6% threshold imposed by deconstructSigs, will be inappropriately strict in these cases."
- [discussion] CI-based specificity criterion: "requiring non-zero 95% CI to call a mutational signature as present was highly effective at improving specificity"
- [readme] reference setup rationale: "# remove canonical AID (SBS84) for genome-wide analysis
# remove the platinum signature (SBS35) because the patients are not platinum exposed"
- [readme] core invocation parameters: "cos_sim_threshold = 0.01,
                             force_include = c("SBS1", "SBS5")"
- [readme] bootstrap iteration guidance: "iterations = 20, # 1000 iterations recommended for stable results"
- [readme] runtime caveat: "# Bootstrapping large datasets with many iterations can significantly increase runtime."
