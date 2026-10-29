---
name: sqtl-effect-size-stratified-analysis
description: Use when nominal sQTL test output (all-tests.nominal.tsv) and FDR-thresholded
  sQTL labels are available and the goal is to test whether sQTLs alter splice site
  strength more often/strongly than non-sQTLs, scaling with md effect size.
when_to_use_negative:
- The analysis target is RNA-binding protein (RBP) binding site disruption rather
  than splice site motifs — the paper reports many more variants modify RBP binding
  sites, which requires a different scoring workflow (e.g. the gkmSVM-based analysis).
- Variants of interest do not overlap annotated donor or acceptor splice site intervals
  — PWM splice site strength is undefined for them.
- Only permuted-pass or FDR-adjusted summaries are available without per-variant nominal
  tests (all-tests.nominal.tsv), since md effect size per variant is needed for stratification.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: sQTLseekeR2
  role: R package underlying the sqtlseeker2-nf pipeline that produces the nominal
    per-variant test statistics (snpId, md effect size, pv) and FDR-based sQTL labels
    consumed by this analysis
  repo: https://github.com/guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow pipeline generating the sQTL mapping outputs (all-tests.nominal.tsv
    and sqtls-${level}fdr.nominal.tsv) used as inputs
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:82bf0762d1a53ff8016275764792646641768a9cce99d9010e276989f0bcb7be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# sqtl-effect-size-stratified-analysis

## Summary

Reproduce the paper's splice-site strength analysis: score reference and alternative alleles of splice-site-overlapping variants with position weight matrices, compare sQTLs (FDR ≤ 0.05) versus non-sQTLs, and stratify absolute strength change |PWM(alt) - PWM(ref)| by sQTL effect size (md) bins to test the Fig. 4a trend.

## When to use

Use when nominal sQTL test output (all-tests.nominal.tsv) and FDR-thresholded sQTL labels are available and the goal is to test whether sQTLs alter splice site strength more often/strongly than non-sQTLs, scaling with md effect size.

## When NOT to use

- The analysis target is RNA-binding protein (RBP) binding site disruption rather than splice site motifs — the paper reports many more variants modify RBP binding sites, which requires a different scoring workflow (e.g. the gkmSVM-based analysis).
- Variants of interest do not overlap annotated donor or acceptor splice site intervals — PWM splice site strength is undefined for them.
- Only permuted-pass or FDR-adjusted summaries are available without per-variant nominal tests (all-tests.nominal.tsv), since md effect size per variant is needed for stratification.

## Inputs

- all-tests.nominal.tsv from sqtlseeker2-nf nominal pass (chr, start, end, snpId, md, pv)
- sqtls-${level}fdr.nominal.tsv with sQTL labels at default fdr 0.05
- Annotated splice site (donor and acceptor) interval coordinates, e.g. from GTEx/Gencode annotation
- Reference genome FASTA for extracting reference and alternative allele splice site sequences
- Human splice site position weight matrices for donor and acceptor sites

## Outputs

- Per-variant table of reference and alternative PWM strength scores for donor/acceptor sites
- Absolute splice site strength change |PWM(alt) - PWM(ref)| per splice-site variant
- Odds ratio (metric_splicesite_strength_or) of strength-altering variants among sQTLs vs non-sQTLs (expected OR ≈ 1.79; 63% of sQTLs vs 49% of non-sQTLs)
- Wilcoxon p-value for the sQTL vs non-sQTL difference in absolute strength change (reported 1.98 × 10−7)
- Stratified plot/table of absolute strength change by md effect size bins (Fig. 4a analogue)

## How to apply

Load per-variant tests from the sqtlseeker2-nf output folder via all-tests.nominal.tsv (columns chr, start, end, snpId, md effect size, pv) and apply sQTL labels from sqtls-${level}fdr.nominal.tsv with the default FDR 0.05 threshold. Intersect tested variant coordinates with annotated donor and acceptor splice site intervals to retain variants overlapping splice junction motifs. Score each variant's reference and alternative allele sequence with position weight matrices built from human splice sites to obtain donor/acceptor strength, then compute the absolute change |PWM(alt) - PWM(ref)| per variant. Partition splice-site variants into sQTLs and non-sQTLs and compute the odds ratio (metric_splicesite_strength_or) of strength-altering variants between the two groups. Finally bin the absolute strength change by md effect size (md has a minimum reported value of 0.05 in sqtlseeker2-nf) and test for a monotonic increase with sQTL effect size, as in Fig. 4a. Rationale: the paper reports that while many variants affect splicing patterns by altering splice site sequences, many more modify RNA-binding protein binding sites, so this analysis quantifies the splice-site-direct mechanism specifically.

## Related tools

- **sQTLseekeR2** (R package underlying the sqtlseeker2-nf pipeline that produces the nominal per-variant test statistics (snpId, md effect size, pv) and FDR-based sQTL labels consumed by this analysis) — https://github.com/guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow pipeline generating the sQTL mapping outputs (all-tests.nominal.tsv and sqtls-${level}fdr.nominal.tsv) used as inputs) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
./nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- Odds ratio of strength-altering variants among sQTLs vs non-sQTLs must be greater than 1 (paper reports OR = 1.79, 63% vs 49%)
- Wilcoxon test of absolute strength change between sQTLs and non-sQTLs should be significant (paper reports p = 1.98 × 10−7)
- Mean/median absolute strength change must increase monotonically with sQTL effect size md bins, reproducing the Fig. 4a pattern
- Only variants overlapping annotated donor or acceptor splice site intervals should carry PWM scores; the variant coordinate join against all-tests.nominal.tsv must be lossless on chr/start/end/snpId
- sQTL/non-sQTL partition must respect the fdr 0.05 threshold from sqtls-${level}fdr.nominal.tsv

## Limitations

- sQTL mapping cannot distinguish whether effects at a locus are caused by the same variant or two different variants in LD, which can blur per-variant effect size stratification
- The pipeline applies a minimum effect size filter (--min_md, default 0.05), so variants below this md threshold are absent from the output and cannot enter the stratification
- LOFTEE-style splice-site interpretation is version-dependent: splice-site rules differ between genome builds (e.g. the LOFTEE master branch does not work with GRCh38), so annotation consistency matters when intersecting splice sites

## Evidence

- [results] A larger fraction of sQTLs than non-sQTLs modifies splice site strength (63% vs 49%, OR = 1.79): [evidence span withheld — anchor status: not_found]
- [abstract] RBP vs splice site scope: "While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins."
- [discussion] pipeline implementation: "our implementation of an enhanced pipeline for sQTL mapping based on sQTLseekeR2, Nextflow and Docker"
- [methods] min_md filter: "--min_md MIN_MD             minimum effect size reported (default: 0.05)"
- [readme] sqtlseeker2-nf README scope: "A pipeline for splicing quantitative trait loci (sQTL) mapping."
