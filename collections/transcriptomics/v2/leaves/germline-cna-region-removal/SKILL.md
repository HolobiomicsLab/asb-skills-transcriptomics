---
name: germline-cna-region-removal
description: Use when constructing euploid-baseline single-cell inputs for simulated
  CNA benchmarking (e.g., SDX chrX pooling from male oligodendrocyte BAMs) or when
  donor cells carry germline CNAs that would confound somatic CNA calling.
when_to_use_negative:
- The target analysis is restricted to autosomes from both-sex samples (HiScanner
  excludes sex chromosomes from its cross-sample workflow; applying chrX-based removal
  there is out of scope)
- Input cells are already euploid across the region of interest — removal is unnecessary
  and discards coverage
- Germline variant genotyping rather than copy-number analysis is the goal; SCAN2-style
  workflows explicitly exclude sex chromosomes and matched-bulk-supported calls
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: samtools
  role: Extract reads mapping to chromosome X from single oligodendrocyte BAM files
    and remove reads from germline CNA regions to establish an euploid baseline
- name: HiScanner
  role: Downstream allele-specific CNA caller whose simulated benchmark (SDX) depends
    on the germline-cleaned euploid starting baseline
  repo: https://github.com/parklab/HiScanner
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:fff3b9a29255400ec588bc6cd267b956d88a6a1cf4545674e2ff4b37d90dd949
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/HiScanner
  tools:
  - HiScanner
  - samtools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Germline CNA region removal

## Summary

Excise reads falling in genomic intervals that harbor germline copy-number alterations (e.g., on donor chromosome X) from single-cell BAM files using samtools, so that downstream simulated cells or analyses start from an euploid baseline. This prevents pre-existing aneuploid/germline CN state from contaminating allele-specific CNA benchmarks or scWGS CNA detection.

## When to use

Use when constructing euploid-baseline single-cell inputs for simulated CNA benchmarking (e.g., SDX chrX pooling from male oligodendrocyte BAMs) or when donor cells carry germline CNAs that would confound somatic CNA calling.

## When NOT to use

- The target analysis is restricted to autosomes from both-sex samples (HiScanner excludes sex chromosomes from its cross-sample workflow; applying chrX-based removal there is out of scope)
- Input cells are already euploid across the region of interest — removal is unnecessary and discards coverage
- Germline variant genotyping rather than copy-number analysis is the goal; SCAN2-style workflows explicitly exclude sex chromosomes and matched-bulk-supported calls

## Inputs

- Single-cell BAM files (e.g., oligodendrocyte scWGS BAMs from two male donors)
- Coordinates of germline copy-number alteration regions to exclude
- Reference chromosome of interest (e.g., chromosome X) interval list

## Outputs

- Filtered chrX BAM with germline CNA regions removed, starting from an euploid baseline
- Cleaned reads available for merging into simulated allele-specific CNA BAMs (SDX)

## How to apply

Identify intervals harboring germline copy-number alterations in the source BAMs, then use samtools to extract and remove reads from those regions — in the SDX benchmark, reads mapping to chromosome X were pulled from single oligodendrocyte BAM files of two male donors (OligoA, OligoB1) with samtools, and reads in germline CNA regions were removed so simulated cells start from a euploid baseline. Apply standard read filters during extraction: keep only uniquely mappable reads with mapping quality >= 30, mirroring the HiScanner preprocessing. To maintain sex consistency when pooling donors (as HiScanner does for autosomal analyses), restrict the analysis appropriately — the article restricts cross-sex comparisons to autosomes, while SDX pools only male chrX. The cleaned reads can then be merged with read-level CNA edits (single-copy loss, single-copy gain, two-copy gain, cnLOH over 200 kb–5 Mb intervals) and written out as one simulated BAM per condition.

## Related tools

- **samtools** (Extract reads mapping to chromosome X from single oligodendrocyte BAM files and remove reads from germline CNA regions to establish an euploid baseline)
- **HiScanner** (Downstream allele-specific CNA caller whose simulated benchmark (SDX) depends on the germline-cleaned euploid starting baseline) — https://github.com/parklab/HiScanner

## Examples

```
samtools view -b OligoB1.bam chrX > oligoB1_chrX.bam
```

## Evaluation signals

- Read depth over removed germline CNA intervals should drop to near zero in the output BAM, while non-CNA intervals retain expected euploid coverage
- All retained reads pass the MAPQ >= 30 / uniquely mappable filter
- For SDX simulation, every generated BAM (4 event types x 13 interval sizes x 100 sites, 200 kb–5 Mb) exists and each ground-truth manifest entry matches the intended CNA type, size, and start site
- Merged SDX BAMs preserve haplotype structure and phase-switch error rates (~1%) from the source phased genotypes

## Limitations

- Intervals misidentified as germline CNA regions will remove genuine coverage and bias simulated CNA events in the same window
- Read removal per se does not restore the single-allele structure of a heterozygous deletion; per-donor sampling (as in SDX) is needed to place events on one haplotype
- Existing CNA callers like HiScanner's simulated benchmark workflow are autosome-restricted for cross-sex comparison, so germline removal on chrX applies mainly to sex-matched pooling schemes

## Evidence

- [methods] Extract chrX reads and remove germline CNA regions for euploid baseline: [evidence span withheld — anchor status: not_found]
- [discussion] Read quality filter threshold for retained reads: "Only uniquely mappable reads with a minimum mapping quality of 30 were considered"
- [discussion] Autosome-only restriction rationale for cross-sample consistency: "To maintain consistency across samples from both sexes, our analysis was restricted to autosomal chromosomes"
- [methods] Ground-truth manifest validation of simulated conditions: [evidence span withheld — anchor status: not_found]
