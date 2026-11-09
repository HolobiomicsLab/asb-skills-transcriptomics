---
name: reconstruction-accuracy-comparison-across-fitting-algorithms
description: Use when comparing mutational signature fitting outcomes across reference
  signature versions (e.g., COSMIC v2 vs COSMIC v3.1) or across fitting algorithms
  (mmsig, deconstructSigs, MutationalPatterns) on WGS catalogs with more than 500
  mutations.
when_to_use_negative:
- Mutational catalogs from targeted capture panels covering only a few megabases,
  which yield too low a mutational burden for meaningful individual-patient signature
  fitting
- Catalogs with fewer than 500 mutations, where fitting accuracy (and thus any reconstruction-accuracy
  comparison) is not optimal
- Whole-exome data interpreted without the caveat that expected signature profiles
  differ between coding and non-coding regions, which biases reference comparisons
edam_topics:
- http://edamontology.org/topic_0749
tools:
- name: mmsig
  role: 'Primary fitting engine: expectation-maximization fitting with bootstrapping-based
    CIs, strand-bias assessment, and refCheck; supplies mm_fit_signatures, plot_signatures,
    bootSigsPlot and the mm_5_col/mm_96_classes/signature_ref data'
  repo: https://github.com/evenrus/mmsig
- name: deconstructSigs
  role: Secondary fitting algorithm run on the same catalogs for cross-algorithm reconstruction-accuracy
    comparison
- name: MutationalPatterns
  role: Secondary fitting algorithm run on the same catalogs for cross-algorithm reconstruction-accuracy
    comparison
- name: devtools
  role: Installs the mmsig development version from GitHub
  repo: https://github.com/evenrus/mmsig
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Genome build required by mmsig for refCheck and transcriptional strand-bias
    computations
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:6aeedbc0ff318d26a69eebf11a545958503dbd94f19ec4d26a68704b6b1f1e7d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# reconstruction-accuracy-comparison-across-fitting-algorithms

## Summary

Benchmarks how the choice of COSMIC reference version (v2 vs v3.1) and fitting algorithm (mmsig, deconstructSigs, MutationalPatterns) affect per-sample signature exposures and reconstruction accuracy in 82 multiple myeloma WGS samples. It exposes reference-version sensitivity that is otherwise hidden when a single pipeline is reported.

## When to use

Use when comparing mutational signature fitting outcomes across reference signature versions (e.g., COSMIC v2 vs COSMIC v3.1) or across fitting algorithms (mmsig, deconstructSigs, MutationalPatterns) on WGS catalogs with more than 500 mutations.

## When NOT to use

- Mutational catalogs from targeted capture panels covering only a few megabases, which yield too low a mutational burden for meaningful individual-patient signature fitting
- Catalogs with fewer than 500 mutations, where fitting accuracy (and thus any reconstruction-accuracy comparison) is not optimal
- Whole-exome data interpreted without the caveat that expected signature profiles differ between coding and non-coding regions, which biases reference comparisons

## Inputs

- mm_5_col sample-level 5-column VCF-style SNV mutation table from the mmsig package (sample, chr, pos, ref, alt)
- signature_ref from mmsig (COSMIC reference with sub/tri context columns), subset to SBS1, SBS2, SBS5, SBS8, SBS9, SBS13, SBS18, SBS-MM1 with SBS84 and SBS35 removed
- COSMIC v3.1 reference signature matrix for the re-fit comparison
- mm_96_classes 96-channel mutation class definitions

## Outputs

- Per-sample signature exposure/estimate table for each reference version and algorithm
- Bootstrap 95% confidence intervals per signature (bootSigsPlot input)
- Reconstruction accuracy comparison (cosine similarity) v2 vs v3.1 per algorithm (mmsig, deconstructSigs, MutationalPatterns)
- Exposure comparison table and v2-vs-v3.1 comparison figure

## How to apply

Load the mm_5_col mutation catalog and mm_96_classes from the mmsig package, then build a COSMIC v2 reference supplemented with SBS-MM1, dropping canonical AID (SBS84) for genome-wide analysis and the platinum signature SBS35 because the patients are not platinum exposed. Run mm_fit_signatures with input.format='vcf', strandbias=TRUE, bootstrap=TRUE, iterations=1000 (the recommended number for stable results), refcheck=TRUE, cos_sim_threshold=0.01, and force_include=c('SBS1','SBS5'), with set.seed(1) for reproducibility. Re-run the identical fitting against the COSMIC v3.1 signature_ref for the same samples, and optionally repeat with deconstructSigs and MutationalPatterns for cross-algorithm comparison. Compare per-sample exposures between the two references and verify the reported directionality: higher SBS1/SBS2/SBS8/SBS9 estimates with v2, higher SBS5 with v3.1, and higher reconstruction accuracy for all three algorithms with v3.1 (as in Fig. 3), since the cleaner v3.1 signatures reduce inter-bleeding. Visualize fitted exposures with plot_signatures and bootstrap 95% CIs with bootSigsPlot, then export the exposure comparison table and v2-vs-v3.1 figure.

## Related tools

- **mmsig** (Primary fitting engine: expectation-maximization fitting with bootstrapping-based CIs, strand-bias assessment, and refCheck; supplies mm_fit_signatures, plot_signatures, bootSigsPlot and the mm_5_col/mm_96_classes/signature_ref data) — https://github.com/evenrus/mmsig
- **deconstructSigs** (Secondary fitting algorithm run on the same catalogs for cross-algorithm reconstruction-accuracy comparison)
- **MutationalPatterns** (Secondary fitting algorithm run on the same catalogs for cross-algorithm reconstruction-accuracy comparison)
- **devtools** (Installs the mmsig development version from GitHub) — https://github.com/evenrus/mmsig
- **BSgenome.Hsapiens.UCSC.hg19** (Genome build required by mmsig for refCheck and transcriptional strand-bias computations)

## Examples

```
set.seed(1); sig_out <- mm_fit_signatures(muts.input=mm_5_col, sig.input=sig_ref, input.format='vcf', strandbias=TRUE, bootstrap=TRUE, iterations=1000, refcheck=TRUE, cos_sim_threshold=0.01, force_include=c('SBS1','SBS5'))
```

## Evaluation signals

- Directionality check vs the paper's Fig. 3: v2 yields higher estimated contributions for SBS1, SBS2, SBS8, and SBS9 and a lower estimate for SBS5 compared with COSMIC v3.1
- Reconstruction accuracy is higher with the v3.1 reference for all three fitting algorithms (mmsig, deconstructSigs, MutationalPatterns)
- set.seed(1) and identical mm_fit_signatures parameters (iterations=1000, cos_sim_threshold=0.01, force_include=c('SBS1','SBS5')) reproduce exposures across runs
- Signatures called as present have non-zero bootstrap 95% CI (mmsig's dynamic error-suppression specificity criterion)
- Removed references are justified and consistent: SBS84 (canonical AID) dropped for genome-wide analysis, SBS35 dropped because patients are not platinum exposed

## Limitations

- Hard cut-offs such as the standard 6% threshold imposed by deconstructSigs will be inappropriately strict in cases of inter-bleeding or low-abundance signatures
- Flat mutational signatures may be falsely removed by mmsig because all mutations can be re-classified to another signature with a relatively low cosine-similarity penalty
- With whole-exome sequencing the expected signature profile in coding and non-coding regions can differ, confounding reference-version comparisons
- Targeted capture panels covering a few megabases often yield too low mutational burdens for meaningful individual-patient signature fitting; optimal accuracy requires catalogs of more than 500 mutations (consistently obtainable by WGS)

## Evidence

- [abstract] COSMIC v3.1 signatures are more clearly defined, improving fit specificity: "This is particularly important in the most recent mutational signature reference (COSMIC v3.1) where each signature is more clearly defined."
- [methods] mm_fit_signatures parameter surface used for both reference fits: "`mm_fit_signatures(muts.input, sig.input, input.format, sample.sigt.profs, bootstrap, iterations, strandbias, refcheck, cos_sim_threshold, force_include, dbg)`"
- [discussion] Bootstrap CI requirement as the specificity criterion for calling a signature: "requiring non-zero 95% CI to call a mutational signature as present was highly effective at improving specificity"
- [readme] Reference setup: drop SBS84 for genome-wide analysis and SBS35 for non-platinum-exposed patients: "# remove canonical AID (SBS84) for genome-wide analysis
# remove the platinum signature (SBS35) because the patients are not platinum exposed"
- [readme] 1000 bootstrap iterations recommended for stable results: "iterations = 20, # 1000 iterations recommended for stable results"
