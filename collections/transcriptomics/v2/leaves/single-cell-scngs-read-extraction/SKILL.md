---
name: single-cell-scngs-read-extraction
description: Use when constructing simulated single-cell scWGS benchmark data that
  must preserve allele-specific haplotype structure, controlled phase-switch error
  rates, and real WGA artifacts — e.g., before evaluating BAF/phasing-based CNA callers
  on ground-truth events spanning 200 kb to 5 Mb.
when_to_use_negative:
- The goal is calling allele-specific CNAs on real single-cell data rather than generating
  simulated ground truth — use HiScanner directly instead
- Input BAMs are from a female donor or from autosomes without haploid allele separation,
  since the SDX trick relies on pooling two male haploid X chromosomes
- The caller under test operates on pre-binned read-depth tracks only, since read-level
  pooling cannot affect its allele-specific behavior
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: samtools
  role: Extracts chromosome X reads from single oligodendrocyte BAM files and removes
    reads from germline CNA regions to establish the euploid baseline
- name: HiScanner
  role: Downstream BAF/phasing-based CNA caller benchmarked against the SDX simulated
    data with embedded non-clonal CNAs
  repo: github.com/parklab/HiScanner
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:a7c5a9d7973e97f5979c41d0bd037a66f497ae0e277800f4d5aba1c3aed3bf24
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HiScanner
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# single-cell-scngs-read-extraction

## Summary

A read-level simulation technique (SDX) that extracts chromosome X reads from single-cell scWGS BAM files of two male haploid donors using samtools, then pools, excises, or duplicates reads to embed allele-specific CNAs while preserving haplotype structure and scWGS amplification artifacts for benchmarking callers like HiScanner.

## When to use

Use when constructing simulated single-cell scWGS benchmark data that must preserve allele-specific haplotype structure, controlled phase-switch error rates, and real WGA artifacts — e.g., before evaluating BAF/phasing-based CNA callers on ground-truth events spanning 200 kb to 5 Mb.

## When NOT to use

- The goal is calling allele-specific CNAs on real single-cell data rather than generating simulated ground truth — use HiScanner directly instead
- Input BAMs are from a female donor or from autosomes without haploid allele separation, since the SDX trick relies on pooling two male haploid X chromosomes
- The caller under test operates on pre-binned read-depth tracks only, since read-level pooling cannot affect its allele-specific behavior

## Inputs

- Single oligodendrocyte BAM files from two male donors (OligoA, OligoB1)
- List of germline copy-number alteration regions to exclude for a euploid chrX baseline
- Simulated phased gHET genotypes with 1% phase-switch errors
- Poisson read-depth noise parameters

## Outputs

- One SDX BAM per simulated condition (4 CNA classes × 13 sizes × 100 start sites)
- Ground-truth manifest recording event type, event size, start coordinate, and donor/cell of origin per SDX BAM

## How to apply

Extract reads mapping to chromosome X from the single oligodendrocyte BAM files of two male donors with samtools, excluding reads from regions harboring germline copy-number alterations so simulated cells start from a euploid baseline (apply the MAPQ ≥ 30, uniquely mappable read filter). For each of 4 CNA classes (single-copy loss, single-copy gain, two-copy gain, cnLOH) and 13 interval sizes from 200 kb to 5 Mb, sample 100 start sites and excise or duplicate reads from the selected OligoB1 chrX intervals before pooling with unmodified OligoA reads — sampling prior to pooling is what allows single-allele gains/losses without disrupting local haplotype structure. Simulate germline heterozygous SNPs along chrX, add Poisson read-depth noise to mimic sequencing stochasticity, and inject phase-switch errors at a 1% rate into the phased genotypes. Merge modified OligoB1 reads with OligoA reads into one SDX BAM per condition, and record event type, size, start coordinate, and donor/cell of origin in a ground-truth manifest.

## Related tools

- **samtools** (Extracts chromosome X reads from single oligodendrocyte BAM files and removes reads from germline CNA regions to establish the euploid baseline)
- **HiScanner** (Downstream BAF/phasing-based CNA caller benchmarked against the SDX simulated data with embedded non-clonal CNAs) — github.com/parklab/HiScanner

## Examples

```
samtools view -h oligoB1.bam chrX | samtools view -b - -F 0x04 -q 30 -o oligoB1_chrX_reads.bam
```

## Evaluation signals

- Every generated SDX BAM exists for all 4 × 13 × 100 simulated conditions
- Each ground-truth manifest entry matches the intended CNA type, size (200 kb–5 Mb), and start site
- Phase-switch errors are present at the 1% rate in the simulated phased genotypes
- Germline CNA regions are confirmed removed, so simulated cells start from a euploid chrX baseline
- Inserted single-allele gains/losses do not disrupt local haplotype structure (phase blocks remain intact after sampling prior to pooling)

## Limitations

- Restricted to chromosome X from male haploid donors; cannot be trivially generalized to diploid autosomes where allele-of-origin per read is ambiguous
- Event sizes span only 200 kb to 5 Mb, so behavior outside this range is untested
- The 1% phase-switch error rate is a single fixed assumption rather than a sweep across realistic error regimes
- HiScanner's model assumes independence between single-cell genomes, which may be violated for cells sharing clonal origins and could affect benchmarking of clonal scenarios

## Evidence

- [methods] Sampling prior to pooling enables allele-specific CNA insertion: [evidence span withheld — anchor status: not_found]
- [methods] Phase-switch errors injected at 1% to challenge BAF/phasing callers: [evidence span withheld — anchor status: not_found]
- [methods] Excise/duplicate OligoB1 chrX reads across 4 classes, 13 sizes, 100 start sites: [evidence span withheld — anchor status: not_found]
- [abstract] HiScanner consumes read depth, BAF, and haplotype phasing: "combines read depth, B-allele frequency, and haplotype phasing to identify CNAs with high resolution"
- [readme] HiScanner is the simulated-data benchmark target: "HiScanner is a python package for high-resolution single-cell copy number analysis. It supports two modes of operation:"
