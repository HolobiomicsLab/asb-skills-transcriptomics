---
name: splice-site-strength-pwm-scoring
description: Use when testing whether cis genetic variants overlapping splice junction
  motifs alter donor/acceptor site strength, and whether the strength change scales
  with sQTL effect size (md).
when_to_use_negative:
- The analysis goal is RNA-binding protein binding site disruption rather than splice
  site strength — the article notes many more variants modify RBP binding sites than
  splice site sequences
- Variants do not overlap annotated donor or acceptor splice site intervals after
  intersection, so no PWM scores can be computed for them
- Splicing phenotypes were quantified without sQTLseekeR2 output (e.g. annotation-free
  LeafCutter intron excision ratios), since the skill depends on all-tests.nominal.tsv
  and sqtls-${level}fdr.nominal.tsv files
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: sQTLseekeR2
  role: R package whose sqtlseeker2-nf output (nominal per-variant tests and FDR-labelled
    sQTLs) supplies the variants, effect sizes (md), and sQTL/non-sQTL labels scored
    with splice site PWMs
  repo: https://github.com/guigolab/sqtlseeker2-nf
- name: sqtlseeker2-nf
  role: Nextflow pipeline that produced the sQTL mapping results consumed by this
    splice site strength analysis
  repo: https://github.com/guigolab/sqtlseeker2-nf
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-20578-2
    title: Identification and analysis of splicing quantitative trait loci across
      multiple tissues in the human genome
  source_content_hash: sha256:b2e6193b7a6b00aa505af83da00e5a46d9a60d1562642973437ada2857e43bf3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/guigolab/sqtlseeker2-nf
  tools:
  - sQTLseekeR2
  - sqtlseeker2-nf
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# splice-site-strength-pwm-scoring

## Summary

Score sQTL and non-sQTL variants overlapping annotated donor/acceptor splice sites with position weight matrices to quantify allele-specific splice site strength, testing whether sQTLs alter splice site strength more frequently (63% vs 49%, OR = 1.79) and more strongly than non-sQTL variants.

## When to use

Use when testing whether cis genetic variants overlapping splice junction motifs alter donor/acceptor site strength, and whether the strength change scales with sQTL effect size (md).

## When NOT to use

- The analysis goal is RNA-binding protein binding site disruption rather than splice site strength — the article notes many more variants modify RBP binding sites than splice site sequences
- Variants do not overlap annotated donor or acceptor splice site intervals after intersection, so no PWM scores can be computed for them
- Splicing phenotypes were quantified without sQTLseekeR2 output (e.g. annotation-free LeafCutter intron excision ratios), since the skill depends on all-tests.nominal.tsv and sqtls-${level}fdr.nominal.tsv files

## Inputs

- sQTLseekeR2 nominal test results (all-tests.nominal.tsv with chr, start, end, snpId, md, pv columns)
- sQTL labels file (sqtls-${level}fdr.nominal.tsv, default FDR 0.05)
- Annotated splice site intervals (donor and acceptor sites)
- Reference genome sequence for extracting reference and alternative allele splice site context

## Outputs

- Per-variant absolute splice site strength change |PWM(alt) - PWM(ref)|
- Odds ratio of strength-altering variants among sQTLs vs non-sQTLs (metric_splicesite_strength_or)
- Stratified absolute strength change by sQTL effect size (md) bins reproducing the Fig. 4a trend

## How to apply

Load the sQTL mapping results from the sqtlseeker2-nf output folder: per-variant tests from all-tests.nominal.tsv (columns chr, start, end, snpId, md effect size, pv) and sQTL labels from sqtls-${level}fdr.nominal.tsv with FDR threshold 0.05. Intersect tested variant coordinates with annotated splice site intervals (donor and acceptor sites) to retain variants overlapping splice junction motifs. Score the reference and alternative allele sequence of each splice-site variant with position weight matrices built from human splice sites to obtain donor/acceptor strength scores, then compute the absolute strength change |PWM(alt) - PWM(ref)| per variant. Partition variants into sQTLs (FDR below 0.05) and non-sQTLs, compute the odds ratio (metric_splicesite_strength_or) of strength-altering variants among sQTLs versus non-sQTLs, and stratify absolute strength change by md effect size bins to test the increasing trend of Fig. 4a.

## Related tools

- **sQTLseekeR2** (R package whose sqtlseeker2-nf output (nominal per-variant tests and FDR-labelled sQTLs) supplies the variants, effect sizes (md), and sQTL/non-sQTL labels scored with splice site PWMs) — https://github.com/guigolab/sqtlseeker2-nf
- **sqtlseeker2-nf** (Nextflow pipeline that produced the sQTL mapping results consumed by this splice site strength analysis) — https://github.com/guigolab/sqtlseeker2-nf

## Examples

```
./nextflow run guigolab/sqtlseeker2-nf -with-docker
```

## Evaluation signals

- The odds ratio of strength-altering variants among sQTLs versus non-sQTLs (metric_splicesite_strength_or) must be greater than 1, consistent with sQTL enrichment (reported 63% vs 49%, OR = 1.79)
- The absolute difference in splice site strength between sQTLs and non-sQTLs must be significant (reported Wilcoxon p-value 1.98 × 10−7)
- Absolute strength change must increase monotonically with sQTL effect size md bins, reproducing the Fig. 4a pattern
- Every scored variant must overlap an annotated donor or acceptor splice site interval after coordinate intersection
- Per-variant rows must carry both a PWM(ref) and PWM(alt) score from the human splice site position weight matrices

## Limitations

- The article notes that while many variants affect splicing patterns by altering splice site sequences, many more modify RNA-binding protein binding sites, so splice site strength captures only part of the splicing disruption mechanism
- LD structure cannot be fully resolved: paired effects may be caused by the same variant or by two different variants in LD (the --ld threshold for LD-based variant clustering defaults to 0, no clustering)
- The minimum effect size filter (--min_md, default 0.05) restricts which variants appear in the sQTLseekeR2 output before PWM scoring

## Evidence

- [abstract] Splice site PWM motivation: "While many variants affect splicing patterns by altering the sequence of splice sites, many more modify the binding sites of RNA-binding proteins."
- [discussion] sQTL mapping pipeline basis: "our implementation of an enhanced pipeline for sQTL mapping based on sQTLseekeR2, Nextflow and Docker"
- [readme] sQTLseekeR2 test structure: "Test for association between splicing ratios and genetic variants in *cis* (nominal pass)"
- [methods] LD clustering default: "--ld LD                     threshold for LD-based variant clustering (default: 0, no clustering)"
- [methods] Effect size filter: "--min_md MIN_MD             minimum effect size reported (default: 0.05)"
