---
name: paired-signature-estimate-statistical-comparison
description: Use when the same tumor samples have signature-fitting estimates from
  two matched contexts (e.g., WES capture-restricted vs WGS SNV catalogs) and you
  must quantify per-sample systematic differences, such as exome-vs-WGS shifts for
  SBS9.
when_to_use_negative:
- Mutation catalogs come from targeted capture panels covering only a few megabases,
  where mutational burdens are too low for meaningful individual-patient signature
  fitting.
- Estimates are unpaired (no matched WES/WGS catalog for the same tumor), so a paired
  T-test on per-sample differences would be invalid.
- The goal is de novo signature extraction (e.g., NMF) rather than statistically comparing
  fits of COSMIC v3.1 reference signatures across sequencing contexts.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0622
tools:
- name: mmsig
  role: Fits COSMIC v3.1 reference signatures to both WGS and exome-restricted catalogs
    via mm_fit_signatures (bootstrapping, strandbias, refcheck), yielding sig_out$estimate
    used for per-sample pairing and the paired T-test
  repo: github.com/evenrus/mmsig
- name: GenomicRanges
  role: Intersects per-sample SNV catalogs with SureSelect V6 + UTR capture BED regions
    to derive the exome-restricted catalogs being compared
- name: IRanges
  role: Provides the interval overlap operations underlying the capture-region intersection
    of SNV catalogs
- name: BSgenome.Hsapiens.UCSC.hg19
  role: hg19 reference genome package loaded by mmsig to support refcheck and transcriptional
    strand-bias assessment
- name: devtools
  role: Installs the mmsig development version from GitHub before running the paired
    fitting comparison
  repo: github.com/evenrus/mmsig
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:ab237028990e1d6edcbb1ceead556bc3ff3d88eee9b5132b11896566bd6cf320
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - GenomicRanges
  - IRanges
  - MutationalPatterns
  - R
  - devtools
  - mmsig
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# paired-signature-estimate-statistical-comparison

## Summary

Statistically compares mutational-signature fitting estimates obtained from matched sequencing contexts — whole-exome capture-restricted (SureSelect V6 + UTR) catalogs vs full WGS catalogs — by pairing per-sample estimates and testing systematic differences. In the source work this exposed that exome-restricted SBS9 (nc-AID) estimates in CLL run ~12% lower than WGS (paired T-test p < 0.001) while SBS-MM1 concordance in MM is excellent.

## When to use

Use when the same tumor samples have signature-fitting estimates from two matched contexts (e.g., WES capture-restricted vs WGS SNV catalogs) and you must quantify per-sample systematic differences, such as exome-vs-WGS shifts for SBS9.

## When NOT to use

- Mutation catalogs come from targeted capture panels covering only a few megabases, where mutational burdens are too low for meaningful individual-patient signature fitting.
- Estimates are unpaired (no matched WES/WGS catalog for the same tumor), so a paired T-test on per-sample differences would be invalid.
- The goal is de novo signature extraction (e.g., NMF) rather than statistically comparing fits of COSMIC v3.1 reference signatures across sequencing contexts.

## Inputs

- Per-sample SNV catalogs in 5-column format (sample, chr, pos, ref, alt; e.g. mmsig's mm_5_col) for whole-genome sequencing
- SureSelect V6 + UTR capture BED interval file used to restrict catalogs to exome regions
- mmsig signature_ref reference built from COSMIC v3.1 (SBS1, SBS2, SBS5, SBS8, SBS9, SBS13, SBS18, SBS-MM1)
- Fitted signature contribution tables (sig_out$estimate) from both WGS and WES-restricted catalogs, per sample

## Outputs

- Paired per-sample WES-vs-WGS signature contribution table (one row per sample per signature)
- Per-sample WES-minus-WGS differences for the target signature (e.g., SBS9 in CLL)
- Paired T-test statistic and p-value (expected p < 0.001 for the ~12% CLL SBS9 exome deficit)
- plot_signatures visualization of paired estimates and bootSigsPlot bootstrap confidence intervals

## How to apply

Build two matched catalogs per tumor: the full WGS SNV catalog (5-column sample/chr/pos/ref/alt format, e.g. mmsig's mm_5_col) and an exome-restricted catalog derived by intersecting SNVs with SureSelect V6 + UTR capture BED intervals using GenomicRanges/IRanges overlaps. Refit both catalogs with the identical mmsig signature reference (signature_ref subset to SBS1, SBS2, SBS5, SBS8, SBS9, SBS13, SBS18, SBS-MM1, excluding SBS84 and SBS35) using mm_fit_signatures with bootstrap=TRUE, 1000 iterations (README-recommended for stable results), strandbias=TRUE, refcheck=TRUE, cos_sim_threshold=0.01, and force_include=c('SBS1','SBS5'). Extract per-sample contributions from sig_out$estimate and join WES-restricted to WGS estimates by sample ID to form pairs. For coding-active processes (SBS-MM1 in MM) assess concordance of the paired estimates; for non-coding-restricted processes (SBS9 nc-AID in CLL) compute per-sample WES-minus-WGS differences and test the shift with a paired T-test — the rationale is that expected signature profiles in coding and non-coding regions can differ. Judge success when MM shows excellent WES-vs-WGS SBS-MM1 concordance and CLL SBS9 exome estimates average ~12% lower than WGS with paired T-test p < 0.001, as in Fig. 8; visualize paired estimates with plot_signatures and bootstrap CIs with bootSigsPlot.

## Related tools

- **mmsig** (Fits COSMIC v3.1 reference signatures to both WGS and exome-restricted catalogs via mm_fit_signatures (bootstrapping, strandbias, refcheck), yielding sig_out$estimate used for per-sample pairing and the paired T-test) — github.com/evenrus/mmsig
- **GenomicRanges** (Intersects per-sample SNV catalogs with SureSelect V6 + UTR capture BED regions to derive the exome-restricted catalogs being compared)
- **IRanges** (Provides the interval overlap operations underlying the capture-region intersection of SNV catalogs)
- **BSgenome.Hsapiens.UCSC.hg19** (hg19 reference genome package loaded by mmsig to support refcheck and transcriptional strand-bias assessment)
- **devtools** (Installs the mmsig development version from GitHub before running the paired fitting comparison) — github.com/evenrus/mmsig

## Examples

```
sig_out <- mm_fit_signatures(muts.input=wes_5col, sig.input=sig_ref, input.format='vcf', bootstrap=TRUE, iterations=1000, force_include=c('SBS1','SBS5')); t.test(wes_sbs9, wgs_sbs9, paired=TRUE)
```

## Evaluation signals

- CLL per-sample WES-minus-WGS SBS9 differences are consistently negative, averaging about -12%, with paired T-test p < 0.001 (Fig. 8 expectation)
- MM SBS-MM1 paired WES-vs-WGS estimates show excellent concordance (per-sample values track closely rather than diverging)
- sig_out$estimate contains contributions for the same signature set (SBS1, SBS2, SBS5, SBS8, SBS9, SBS13, SBS18, SBS-MM1) in both sequencing contexts, enabling 1:1 per-sample pairing
- Both arms use identical mm_fit_signatures settings: force_include=c('SBS1','SBS5'), cos_sim_threshold=0.01, refcheck=TRUE, strandbias=TRUE, and 1000 bootstrap iterations
- Companion quality outputs (sig_out$bootstrap via bootSigsPlot and sig_out$strand_bias_mm1) are produced for CI and strand-bias inspection

## Limitations

- The expected signature profile in coding and non-coding regions can be quite different, so exome-restricted fits are not interchangeable with WGS fits — this is exactly the source of the ~12% SBS9 underestimate in CLL
- Targeted capture sequencing panels covering a few megabases often yield too low mutational burdens to allow meaningful individual-patient signature fitting
- Fitting accuracy is optimal only for catalogs of more than 500 mutations, consistently obtainable by WGS; smaller catalogs reduce reliability
- Flat mutational signatures may be falsely removed by mmsig because mutations can be re-classified to another signature at relatively low cosine similarity
- Bootstrapping large catalogs with many iterations significantly increases runtime (README: 1000 iterations recommended for stable results)

## Evidence

- [methods] mm_fit_signatures(muts.input, sig.input, input.format, sample.sigt.profs, bootstrap, iterations, strandbias, refcheck, cos_sim_threshold, force_include, dbg): "mm_fit_signatures(muts.input, sig.input, input.format, sample.sigt.profs, bootstrap, iterations, strandbias, refcheck, cos_sim_threshold, force_include, dbg)"
- [discussion] with the caveat that the expected signature profile in coding and non-coding regions can be quite different: "with the caveat that the expected signature profile in coding and non-coding regions can be quite different"
- [discussion] Targeted capture sequencing panels covering a few megabases often yield too low mutational burdens to allow meaningful mutational signature fitting for individual patients.: "Targeted capture sequencing panels covering a few megabases often yield too low mutational burdens to allow meaningful mutational signature fitting for individual patients."
- [discussion] The accuracy of mutational signature fitting, in general, is optimal when applied to catalogs of more than 500 mutations, which can be consistently obtained by WGS: "The accuracy of mutational signature fitting, in general, is optimal when applied to catalogs of more than 500 mutations, which can be consistently obtained by WGS"
- [readme] mmsig is based on an expectation maximization algorithm for mutational signature fitting and applies cosine similarities for dynamic error suppression: "mmsig is based on an expectation maximization algorithm for mutational signature fitting and applies cosine similarities for dynamic error suppression"
- [readme] iterations = 20, # 1000 iterations recommended for stable results: "iterations = 20, # 1000 iterations recommended for stable results"
