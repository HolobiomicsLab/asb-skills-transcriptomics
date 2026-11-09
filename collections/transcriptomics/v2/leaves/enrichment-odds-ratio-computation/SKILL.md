---
name: enrichment-odds-ratio-computation
description: Use when testing whether sQTLs alter annotated splice-site strength more
  frequently or more strongly than non-sQTL variants, via PWM scores of reference
  vs alternative alleles and a 2×2 odds ratio at FDR 0.05.
when_to_use_negative:
- The question is about variants modifying RNA-binding protein binding motifs rather
  than splice-site strength — the paper reports these outnumber splice-site sequence
  changes
- Variants were never tested by sqtlseeker2-nf, so no sQTL vs non-sQTL partition exists
  and the 2×2 enrichment is undefined
- Variants do not overlap annotated donor/acceptor splice-site intervals (e.g. they
  act through intron retention hotspots or distal mechanisms)
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: sQTLseekeR2
  role: Provides the nominal per-variant association tests (md effect size, pv) and
    the FDR-based sQTL labels used to partition splice-site variants for the odds
    ratio
  repo: guigolab/sQTLseekeR2
- name: sqtlseeker2-nf
  role: Nextflow pipeline that generates the all-tests.nominal.tsv and sqtls-${level}fdr.nominal.tsv
    files consumed by this analysis
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:808e19192260c9f104a128b13c2b3e2effa9ea79bcac36d60bf28201990a77e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sQTLseekeR2
  tools:
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# enrichment-odds-ratio-computation

## Summary

Computes the odds ratio of splice-site strength-altering variants among sQTLs versus non-sQTLs, using PWM-scored donor/acceptor alleles from sqtlseeker2-nf nominal output, to show that sQTLs preferentially modify splice site strength (63% vs 49%, OR = 1.79).

## When to use

Use when testing whether sQTLs alter annotated splice-site strength more frequently or more strongly than non-sQTL variants, via PWM scores of reference vs alternative alleles and a 2×2 odds ratio at FDR 0.05.

## When NOT to use

- The question is about variants modifying RNA-binding protein binding motifs rather than splice-site strength — the paper reports these outnumber splice-site sequence changes
- Variants were never tested by sqtlseeker2-nf, so no sQTL vs non-sQTL partition exists and the 2×2 enrichment is undefined
- Variants do not overlap annotated donor/acceptor splice-site intervals (e.g. they act through intron retention hotspots or distal mechanisms)

## Inputs

- sqtlseeker2-nf nominal output all-tests.nominal.tsv (columns: chr, start, end, snpId, md effect size, pv)
- sQTL call file sqtls-${level}fdr.nominal.tsv (default fdr threshold 0.05) from sqtlseeker2-nf
- genome annotation intervals for donor and acceptor splice sites
- position weight matrices built from human splice sites
- reference genome sequence for extracting ref/alt allele splice-site context

## Outputs

- Odds ratio (metric_splicesite_strength_or) of strength-altering variants among sQTLs vs non-sQTLs (reported 1.79)
- Per-variant absolute splice-site strength change |PWM(alt) − PWM(ref)|
- Fractions of strength-modifying sQTLs and non-sQTLs (reported 63% vs 49%)
- Stratified |ΔPWM| distributions across md effect-size bins reproducing the Fig. 4a increasing trend

## How to apply

Load the sQTL mapping results from the sqtlseeker2-nf output folder: all-tests.nominal.tsv (chr, start, end, snpId, md effect size, pv) for all tested variants, and sqtls-${level}fdr.nominal.tsv with the default fdr 0.05 threshold for sQTL labels. Intersect tested variant coordinates with annotated donor and acceptor splice-site intervals to keep variants overlapping splice junction motifs. Score the reference and alternative allele sequence of each splice-site variant with position weight matrices built from human splice sites, and compute the absolute strength change |PWM(alt) − PWM(ref)| per variant. Partition splice-site variants into sQTLs (fdr < 0.05) and non-sQTLs, then compute the odds ratio (metric_splicesite_strength_or) of strength-altering variants among sQTLs versus non-sQTLs. Finally, bin the absolute strength change by the sQTLseekeR2 md effect size to test the increasing trend of Fig. 4a. The rationale is that a PWM-based sequence-level metric isolates cis-regulatory splice-disruption, which the paper contrasts with the larger set of variants modifying RNA-binding protein binding sites.

## Related tools

- **sQTLseekeR2** (Provides the nominal per-variant association tests (md effect size, pv) and the FDR-based sQTL labels used to partition splice-site variants for the odds ratio) — guigolab/sQTLseekeR2
- **sqtlseeker2-nf** (Nextflow pipeline that generates the all-tests.nominal.tsv and sqtls-${level}fdr.nominal.tsv files consumed by this analysis) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
./nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- Odds ratio must be greater than 1, reproducing the reported 63% (sQTLs) vs 49% (non-sQTLs) enrichment, OR = 1.79
- Wilcoxon test on |PWM(alt) − PWM(ref)| between sQTLs and non-sQTLs is significant (reported p-value 1.98 × 10−7)
- Absolute splice-site strength change increases monotonically with md effect-size bins, matching the Fig. 4a pattern
- Only variants overlapping annotated donor/acceptor splice-site intervals enter the 2×2 table, and sQTL membership is assigned at fdr 0.05

## Limitations

- Non-sQTL labels can reflect limited statistical power rather than a true absence of effect, which biases the odds ratio
- LD structure means paired sQTL/eQTL-type effects cannot be attributed to the same variant versus distinct variants in LD
- Reported effect sizes are restricted by the --min_md default of 0.05, so weaker effects are excluded from the md binning

## Evidence

- [results] A larger fraction of sQTLs than non-sQTLs modifies splice site strength (63% vs 49%, OR = 1.79): [evidence span withheld — anchor status: not_found]
- [results] Absolute strength change is larger for sQTLs and scales with effect size: [evidence span withheld — anchor status: not_found]
- [abstract] Splice-site sequence changes are only part of the splicing-variant spectrum: "While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins."
- [supplementary] Input files and columns for the per-variant tests: [evidence span withheld — anchor status: not_found]
- [methods] Effect size filter on reported md values: "minimum effect size reported (default: 0.05)"
