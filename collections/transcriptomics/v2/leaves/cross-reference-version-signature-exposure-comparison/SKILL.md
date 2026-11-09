---
name: cross-reference-version-signature-exposure-comparison
description: Use when fitting mutational signatures to a mutation catalog and you
  need to assess sensitivity of per-sample signature exposures to the reference signature
  set version (e.g. COSMIC v2 vs v3.1), or to justify which reference version's cleaner
  signature definitions to report.
when_to_use_negative:
- Targeted capture panels covering only a few megabases, where mutational burdens
  are too low for meaningful individual-patient signature fitting
- Mutation catalogs with fewer than ~500 mutations, where signature fitting accuracy
  is not optimal
- De novo signature extraction tasks — this skill assumes reference-based fitting
  against fixed COSMIC signature sets
edam_operation: http://edamontology.org/operation_0331
edam_topics:
- http://edamontology.org/topic_3673
- http://edamontology.org/topic_0622
tools:
- name: mmsig
  role: R package providing mm_fit_signatures (expectation maximization fitting with
    cosine-similarity error suppression, bootstrapping, refcheck, strandbias) used
    to fit both reference versions
  repo: https://github.com/evenrus/mmsig
- name: deconstructSigs
  role: Alternative fitting algorithm included in the three-algorithm reconstruction-accuracy
    comparison (imposes a standard 6% hard threshold)
- name: MutationalPatterns
  role: Alternative fitting algorithm included in the reconstruction-accuracy comparison
    across reference versions
- name: BSgenome.Hsapiens.UCSC.hg19
  role: Genome annotation loaded by mmsig for strand bias and refcheck steps
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s42003-021-01938-0
    title: 'mmsig: a fitting approach to accurately identify somatic mutational signatures
      in hematological malignancies'
  source_content_hash: sha256:f7b1a7924f2a06134aa33e2413c627f67495a200a6a4521e2518262f94a5b13d
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
  edam_operation: http://edamontology.org/operation_0331
  edam_topics:
  - http://edamontology.org/topic_3673
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cross-reference-version-signature-exposure-comparison

## Summary

Quantifies how the choice of COSMIC mutational signature reference version (v2 vs v3.1) changes fitted signature exposures and reconstruction accuracy, by re-running the same fitting workflow (e.g. mmsig on multiple myeloma samples) against both references and comparing per-sample estimates.

## When to use

Use when fitting mutational signatures to a mutation catalog and you need to assess sensitivity of per-sample signature exposures to the reference signature set version (e.g. COSMIC v2 vs v3.1), or to justify which reference version's cleaner signature definitions to report.

## When NOT to use

- Targeted capture panels covering only a few megabases, where mutational burdens are too low for meaningful individual-patient signature fitting
- Mutation catalogs with fewer than ~500 mutations, where signature fitting accuracy is not optimal
- De novo signature extraction tasks — this skill assumes reference-based fitting against fixed COSMIC signature sets

## Inputs

- mm_5_col mutation catalog (5-column VCF-like sample/chr/pos/ref/alt data from the mmsig package)
- mm_96_classes (mmsig) 96-channel class definitions
- COSMIC v2 signature reference supplemented with SBS-MM1, excluding SBS84 and SBS35
- COSMIC v3.1 signature_ref from the mmsig package

## Outputs

- Per-sample signature exposure table for each reference version (sig_out$estimate)
- Bootstrap 95% confidence intervals for each signature contribution (sig_out$bootstrap)
- v2-vs-v3.1 exposure comparison table and comparison figure (Fig. 3-style)
- Reconstruction accuracy comparison across mmsig, deconstructSigs, and MutationalPatterns

## How to apply

Build two signature references for the same sample set: a COSMIC v2 reference supplemented with SBS-MM1 (dropping canonical AID SBS84 for genome-wide analysis and platinum SBS35 when patients are not platinum exposed) and the COSMIC v3.1 signature_ref. Run mm_fit_signatures on the mm_5_col catalog with input.format='vcf', strandbias=TRUE, bootstrap=TRUE, iterations=1000 (recommended for stable results), refcheck=TRUE, cos_sim_threshold=0.01, force_include=c('SBS1','SBS5'), with set.seed(1) for reproducibility, then repeat identically against v3.1. Compare per-sample exposures between references and verify the reported directionality: v2 yields higher estimates for SBS1, SBS2, SBS8, SBS9 and lower SBS5 than v3.1, while the cleaner v3.1 signatures give higher reconstruction accuracy for all three fitting algorithms (mmsig, deconstructSigs, MutationalPatterns) as in Fig. 3. Visualize with plot_signatures and bootSigsPlot, and save the exposure comparison table and v2-vs-v3.1 figure.

## Related tools

- **mmsig** (R package providing mm_fit_signatures (expectation maximization fitting with cosine-similarity error suppression, bootstrapping, refcheck, strandbias) used to fit both reference versions) — https://github.com/evenrus/mmsig
- **deconstructSigs** (Alternative fitting algorithm included in the three-algorithm reconstruction-accuracy comparison (imposes a standard 6% hard threshold))
- **MutationalPatterns** (Alternative fitting algorithm included in the reconstruction-accuracy comparison across reference versions)
- **BSgenome.Hsapiens.UCSC.hg19** (Genome annotation loaded by mmsig for strand bias and refcheck steps)

## Examples

```
set.seed(1); sig_out <- mm_fit_signatures(muts.input=mm_5_col, sig.input=sig_ref, input.format="vcf", strandbias=TRUE, bootstrap=TRUE, iterations=1000, refcheck=TRUE, cos_sim_threshold=0.01, force_include=c("SBS1","SBS5"))
```

## Evaluation signals

- Directionality check: COSMIC v2 exposures are higher for SBS1, SBS2, SBS8, SBS9 and lower for SBS5 relative to v3.1
- Reconstruction accuracy is higher with v3.1 than v2 for all three algorithms (mmsig, deconstructSigs, MutationalPatterns), reproducing Fig. 3
- set.seed(1) with identical mm_fit_signatures parameters (iterations=1000, cos_sim_threshold=0.01, force_include=c('SBS1','SBS5')) reproduces exposures across runs
- Forced inclusion of SBS1 and SBS5 present in both reference fits; excluded signatures (SBS84, SBS35) absent from both references
- Bootstrap output (bootSigsPlot) shows non-zero 95% CI only for signatures called present

## Limitations

- Bootstrapping large datasets with many iterations significantly increases runtime; the README example uses 20 iterations with 1000 recommended for stable results
- Flat mutational signatures may be falsely removed by mmsig's dynamic error suppression because all mutations can be re-classified to another signature at relatively low cosine similarity
- Different mutational processes occupy largely the same feature space, so signature analysis carries inherent uncertainty; whole-exome data may show different expected profiles in coding vs non-coding regions

## Evidence

- [other] v2 vs v3.1 exposure directionality: [evidence span withheld — anchor status: not_found]
- [abstract] v3.1 cleaner signatures: "This is particularly important in the most recent mutational signature reference (COSMIC v3.1) where each signature is more clearly defined."
- [readme] remove SBS84 and SBS35: "# remove canonical AID (SBS84) for genome-wide analysis # remove the platinum signature (SBS35) because the patients are not platinum exposed"
- [readme] recommended iterations: "iterations = 20, # 1000 iterations recommended for stable results"
- [methods] install and load: "devtools::install_github("evenrus/mmsig")"
- [other] three-algorithm comparison: [evidence span withheld — anchor status: not_found]
