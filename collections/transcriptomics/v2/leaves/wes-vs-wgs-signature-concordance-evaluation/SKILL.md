---
name: wes-vs-wgs-signature-concordance-evaluation
description: Use when paired WGS and WES SNV catalogs exist for the same samples and
  you need to test whether exome capture regions distort COSMIC v3.1 signature contributions
  (e.g. non-coding signatures like SBS9 or flat signatures like SBS-MM1) before reporting
  WES-based fits.
when_to_use_negative:
- When the input is from targeted capture panels covering only a few megabases — mutational
  burdens are too low for meaningful individual-patient signature fitting, even in
  WGS
- When catalogs contain fewer than ~500 mutations, since mmsig's built-in error-suppression
  and bootstrap CIs only reach excellent sensitivity and specificity on larger catalogs
- When the question is de novo signature discovery rather than fitting against a fixed
  COSMIC v3.1 reference set
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: mmsig
  role: 'Core fitting engine: mm_fit_signatures with bootstrapping, refcheck, strandbias,
    and force_include runs the EM-based signature fit on both WGS and exome-restricted
    catalogs'
  repo: https://github.com/evenrus/mmsig
- name: devtools
  role: Installs the mmsig development version from GitHub
  repo: https://github.com/evenrus/mmsig
- name: GenomicRanges
  role: Intersects SNV catalogs with SureSelect V6 + UTR capture BED intervals to
    derive exome-restricted mutation catalogs
- name: IRanges
  role: Interval overlap operations used with GenomicRanges for the capture-region
    restriction step
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Reference genome loaded by mmsig for trinucleotide context assignment and
    refcheck
- name: MutationalPatterns
  role: Imported dependency for mutational catalog construction in the mmsig workflow
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:2a4891432b7cbe220fb46c3259418789814bd82b0d4ed6b0f03da5bdbaeee196
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/evenrus/mmsig
  tools:
  - BSgenome.Hsapiens.UCSC.hg19
  - GenomicRanges
  - IRanges
  - MutationalPatterns
  - R
  - devtools
  - mmsig
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# wes-vs-wgs-signature-concordance-evaluation

## Summary

A paired evaluation skill that refits mutational signatures with mmsig::mm_fit_signatures on both whole-genome SNV catalogs and exome-restricted catalogs (SureSelect V6 + UTR capture intervals) to quantify how capture-region restriction biases signature estimates, e.g. the ~12% underestimation of the non-coding AID signature SBS9 in CLL.

## When to use

Use when paired WGS and WES SNV catalogs exist for the same samples and you need to test whether exome capture regions distort COSMIC v3.1 signature contributions (e.g. non-coding signatures like SBS9 or flat signatures like SBS-MM1) before reporting WES-based fits.

## When NOT to use

- When the input is from targeted capture panels covering only a few megabases — mutational burdens are too low for meaningful individual-patient signature fitting, even in WGS
- When catalogs contain fewer than ~500 mutations, since mmsig's built-in error-suppression and bootstrap CIs only reach excellent sensitivity and specificity on larger catalogs
- When the question is de novo signature discovery rather than fitting against a fixed COSMIC v3.1 reference set

## Inputs

- WGS SNV catalog in mmsig 5-column format (sample, chr, pos, ref, alt; e.g. mm_5_col)
- Paired WES SNV catalog for the same samples (or WGS catalog to restrict in silico)
- SureSelect V6 + UTR capture BED interval file
- COSMIC v3.1 signature reference matrix (mmsig signature_ref, SBS84 and SBS35 excluded)
- Reference genome package BSgenome.Hsapiens.UCSC.hg19

## Outputs

- Per-sample signature contribution table (sig_out$estimate) for both WGS and WES-restricted catalogs
- Paired per-sample WES-minus-WGS differences for each signature (e.g. SBS9 delta in CLL)
- Paired T-test statistic and p-value for WES-vs-WGS concordance (e.g. mean 12% SBS9 underestimation, p < 0.001)
- Bootstrap confidence intervals (sig_out$bootstrap via bootSigsPlot) and strand-bias flags (sig_out$strand_bias_mm1)
- plot_signatures figures comparing paired WES/WGS estimates (cf. Fig. 8)

## How to apply

Load the WGS SNV catalogs in mmsig 5-column format (sample, chr, pos, ref, alt — e.g. mm_5_col) and the COSMIC v3.1 signature_ref subset (SBS1, SBS2, SBS5, SBS8, SBS9, SBS13, SBS18, SBS-MM1, with SBS84 and SBS35 removed per the README). Intersect each SNV catalog with the SureSelect V6 + UTR capture BED using GenomicRanges/IRanges overlap operations to build exome-restricted catalogs. Refit both catalogs with mm_fit_signatures(input.format='vcf', strandbias=TRUE, bootstrap=TRUE, refcheck=TRUE, force_include=c('SBS1','SBS5')), using 1000 bootstrap iterations as recommended by the README for stable results. Extract fitted contributions from sig_out$estimate, pair WES-restricted estimates with matched WGS estimates per sample, and test differences with a paired T-test; in MM expect excellent SBS-MM1 concordance, while in CLL SBS9 exome estimates run ~12% lower than WGS (p < 0.001) because the nc-AID process is predominantly active in non-coding regions. The rationale is that coding/non-coding trinucleotide contexts differ, so capture restriction systematically reshapes the apparent signature profile.

## Related tools

- **mmsig** (Core fitting engine: mm_fit_signatures with bootstrapping, refcheck, strandbias, and force_include runs the EM-based signature fit on both WGS and exome-restricted catalogs) — https://github.com/evenrus/mmsig
- **devtools** (Installs the mmsig development version from GitHub) — https://github.com/evenrus/mmsig
- **GenomicRanges** (Intersects SNV catalogs with SureSelect V6 + UTR capture BED intervals to derive exome-restricted mutation catalogs)
- **IRanges** (Interval overlap operations used with GenomicRanges for the capture-region restriction step)
- **BSgenome.Hsapiens.UCSC.hg19** (Reference genome loaded by mmsig for trinucleotide context assignment and refcheck)
- **MutationalPatterns** (Imported dependency for mutational catalog construction in the mmsig workflow)

## Examples

```
library(mmsig); data(mm_5_col); data(signature_ref); sig_ref <- signature_ref[c("sub","tri","SBS1","SBS2","SBS5","SBS8","SBS9","SBS13","SBS18","SBS-MM1")]; sig_out <- mm_fit_signatures(muts.input=mm_5_col_subset, sig.input=sig_ref, input.format="vcf", strandbias=TRUE, bootstrap=TRUE, iterations=1000, refcheck=TRUE, force_include=c("SBS1","SBS5"))
```

## Evaluation signals

- In multiple myeloma, SBS-MM1 WES-vs-WGS estimates should show excellent concordance (the coding-restricted fit matches WGS)
- In CLL, per-sample WES-minus-WGS SBS9 differences should average about -12% with paired T-test p < 0.001, reflecting SBS9 (nc-AID) activity in non-coding regions
- Bootstrap output should support presence calls via non-zero 95% CI for included signatures, with forced inclusion of SBS1 and SBS5 surviving refcheck
- After restricting to SureSelect V6 + UTR intervals, no SNV coordinates should lie outside the capture BED (GenomicRanges overlap count equals restricted-catalog row count)
- Fitted contribution columns in sig_out$estimate must be non-negative and sum to ~1 per sample

## Limitations

- Whole-exome sequencing caveat: the expected signature profile in coding and non-coding regions can be quite different, so capture-region restriction systematically biases non-coding signatures like SBS9
- Flat mutational signatures may be falsely removed by mmsig because all mutations can be re-classified to another signature with a low cosine similarity
- Signature fitting accuracy is optimal only with catalogs of more than 500 mutations, which are consistently obtainable by WGS; 1000 bootstrap iterations are recommended but significantly increase runtime
- Different mutational processes occupy largely the same feature space, so signature analysis is subject to a degree of uncertainty

## Evidence

- [discussion] Exome caveat: "with the caveat that the expected signature profile in coding and non-coding regions can be quite different"
- [discussion] Catalog size: "The accuracy of mutational signature fitting, in general, is optimal when applied to catalogs of more than 500 mutations, which can be consistently obtained by WGS"
- [discussion] Panel limitation: "Targeted capture sequencing panels covering a few megabases often yield too low mutational burdens to allow meaningful mutational signature fitting for individual patients."
- [readme] Signature set: "# remove canonical AID (SBS84) for genome-wide analysis"
- [readme] Fit parameters: "strandbias = TRUE,
                             bootstrap = TRUE,
                             iterations = 20, # 1000 iterations recommended for stable results"
- [readme] Bootstrap cost: "Bootstrapping large datasets with many iterations can significantly increase runtime."
