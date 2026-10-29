---
name: transcriptional-strand-bias-testing
description: Use when a mutational signature call from fitting (mmsig, deconstructSigs,
  MutationalPatterns) needs independent confirmation — e.g., platinum signature SBS35
  in therapy-related AML — and WGS data with strand annotations are available.
when_to_use_negative:
- Targeted capture panels covering a few megabases — mutation burdens are too low
  for meaningful individual-patient signature fitting or strand-bias testing
- Catalogs with fewer than ~500 mutations — fitting accuracy, and hence the CI/strand-bias
  confirmation logic, is suboptimal
- Flat mutational signatures that mmsig may falsely remove because mutations can be
  re-classified to another signature at low cosine-similarity cost
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: mmsig
  role: Runs EM-based signature fitting with built-in getStrandBias for Poisson-tested
    transcriptional strand bias and bootstrap CIs
  repo: https://github.com/evenrus/mmsig
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Provides genome annotation to classify mutations as transcribed vs untranscribed
- name: deconstructSigs
  role: Comparator method whose hard 6% exposure cut-off is contrasted with mmsig's
    CI + strand-bias specificity strategy
- name: devtools
  role: Installs the mmsig development version from GitHub
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:a2f4e42d17b4182329165081c78c69f2096b00cf8467a4dd14cd4f04a576d7df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/evenrus/mmsig
  tools:
  - BSgenome
  - BSgenome.Hsapiens.UCSC.hg19
  - R
  - deconstructSigs
  - devtools
  - mmsig
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# transcriptional-strand-bias-testing

## Summary

Assess transcriptional strand bias of fitted mutational signatures (e.g., SBS35, SBS-MM1) as an orthogonal confirmation that a signature is truly present, complementing the non-zero bootstrap 95% CI criterion in mmsig.

## When to use

Use when a mutational signature call from fitting (mmsig, deconstructSigs, MutationalPatterns) needs independent confirmation — e.g., platinum signature SBS35 in therapy-related AML — and WGS data with strand annotations are available.

## When NOT to use

- Targeted capture panels covering a few megabases — mutation burdens are too low for meaningful individual-patient signature fitting or strand-bias testing
- Catalogs with fewer than ~500 mutations — fitting accuracy, and hence the CI/strand-bias confirmation logic, is suboptimal
- Flat mutational signatures that mmsig may falsely remove because mutations can be re-classified to another signature at low cosine-similarity cost

## Inputs

- 5-column VCF-format SNV catalog (sample, chr, pos, ref, alt), e.g. mmsig example data mm_5_col
- Trimmed COSMIC v3.1 signature reference matrix (sub, tri, SBS columns), e.g. signature_ref with SBS84 and non-exposure SBS35 removed
- Genome build annotation (BSgenome.Hsapiens.UCSC.hg19) for transcribed/untranscribed strand assignment

## Outputs

- Per-sample strand-bias table (e.g. sig_out$strand_bias_mm1) with columns group, transcribed, untranscribed, ratio, p_poisson, MM1_flag
- Bootstrap 95% CI per signature per sample (sig_out$bootstrap, plottable with bootSigsPlot)
- Combined present/absent signature calls using non-zero CI plus significant Poisson strand-bias test

## How to apply

Run mmsig's mm_fit_signatures on the 5-column VCF-format SNV catalog (sample, chr, pos, ref, alt) with strandbias=TRUE, so getStrandBias computes transcribed-vs-untranscribed mutation counts per sample. Call a signature present when the bootstrap 95% CI is non-zero AND significant transcriptional strand bias is detected by a Poisson test — the two criteria together improved specificity, cutting the SBS35 false-positive rate in 49 de novo AML samples to ~21% for mmsig versus ~70% for deconstructSigs' 6% exposure cut-off. For SBS35 in platinum-exposed t-AML, confirm strand bias in the C[C>A]C and C[C>T] contexts; for SBS-MM1, read sig_out$strand_bias_mm1 where MM1_flag marks samples with significant skew. Treat CI-above-zero without strand bias (e.g., TCGAAB3001) as an unconfirmed call. Bootstrapping with ~1000 iterations is recommended for stable CI estimates before judging bias.

## Related tools

- **mmsig** (Runs EM-based signature fitting with built-in getStrandBias for Poisson-tested transcriptional strand bias and bootstrap CIs) — https://github.com/evenrus/mmsig
- **BSgenome.Hsapiens.UCSC.hg19** (Provides genome annotation to classify mutations as transcribed vs untranscribed)
- **deconstructSigs** (Comparator method whose hard 6% exposure cut-off is contrasted with mmsig's CI + strand-bias specificity strategy)
- **devtools** (Installs the mmsig development version from GitHub)

## Examples

```
sig_out <- mm_fit_signatures(muts.input=mm_5_col_subset, sig.input=sig_ref, input.format="vcf", strandbias=TRUE, bootstrap=TRUE, refcheck=TRUE, cos_sim_threshold=0.01, force_include=c("SBS1","SBS5"))
```

## Evaluation signals

- strand_bias_mm1 table contains per-sample transcribed/untranscribed counts, ratio, p_poisson, and MM1_flag; MM1_flag is set only for Poisson-significant skew (e.g. ratio 1.61, p ≈ 4.1e-05 for MEL_PD26411c)
- Both platinum-exposed t-AML patients must show SBS35 with non-zero CI AND significant transcriptional strand bias in C[C>A]C and C[C>T]C contexts
- TCGAAB3001 serves as a negative control for the combined criterion: CI excludes zero but no strand bias is present, so the call is not confirmed
- Reproducing the SBS35 false-positive benchmark (~21% mmsig vs ~70% deconstructSigs across 49 de novo AML samples) validates the combined criterion
- Bootstrap CIs (1000 iterations recommended) must be stable; unstable CIs invalidate the non-zero-CI component of the call

## Limitations

- Flat signatures may be falsely removed because all mutations can be re-classified to another signature with a relatively low cosine-similarity penalty
- With whole-exome sequencing, the expected signature profile in coding and non-coding regions can differ, complicating strand-bias interpretation
- Different mutational processes occupy largely the same feature space, so signature analysis is inherently uncertain — strand bias is a supporting, not standalone, criterion

## Evidence

- [discussion] CI + strand-bias as specificity criteria: "estimating 95% CI for the contribution of each mutational signature and checking for transcriptional strand bias typical of signatures associated with transcription-coupled repair"
- [readme] mmsig algorithm scope: "applies cosine similarities for dynamic error suppression as well as bootstrapping-based confidence intervals and assessment of transcriptional strand bias."
- [discussion] non-zero CI criterion: "requiring non-zero 95% CI to call a mutational signature as present was highly effective at improving specificity"
- [readme] strand bias output: "## Transcriptional strand bias for SBS-MM1

``` r
head(sig_out$strand_bias_mm1)"
- [discussion] low-burden caveat: "Targeted capture sequencing panels covering a few megabases often yield too low mutational burdens to allow meaningful mutational signature fitting for individual patients."
