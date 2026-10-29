---
name: phased-genotype-error-simulation
description: Use when benchmarking allele-specific CNA callers on scWGS data requires
  ground-truth simulations that preserve haplotype structure and phasing accuracy,
  e.g., testing sensitivity to 200 kb–5 Mb CNAs under controlled phase-switch error
  rates.
when_to_use_negative:
- Input is real tumor scWGS data whose clonal haplotype structure should be inferred,
  not simulated — SDX models synthetic diploid chrX only
- The caller under test does not use BAF or phasing information; the 1% phase-switch
  error injection targets phasing-aware methods specifically
- A benchmark requires realistic female XX or autosome diploid simulation — SDX pools
  male haploid X chromosomes and this construction does not generalize directly
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: samtools
  role: Extract chrX reads from oligodendrocyte BAM files and manipulate/excise/duplicate
    reads to create allele-specific copy-number states
- name: HiScanner
  role: Allele-specific CNA caller benchmarked on the SDX simulated data (combines
    read depth, BAF, and haplotype phasing)
  repo: github.com/parklab/HiScanner
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:5657571631ca946919ad8aafbf07de28c2e68268d13c1d416aff030c8897dd05
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# Phased genotype error simulation

## Summary

A simulation strategy for building synthetic diploid X (SDX) scWGS benchmark data by pooling reads from two male haploid X chromosomes (OligoA and OligoB1), inserting allele-specific CNAs, and injecting phase-switch errors at a controlled 1% rate so BAF/phasing-based CNA callers are challenged realistically.

## When to use

Use when benchmarking allele-specific CNA callers on scWGS data requires ground-truth simulations that preserve haplotype structure and phasing accuracy, e.g., testing sensitivity to 200 kb–5 Mb CNAs under controlled phase-switch error rates.

## When NOT to use

- Input is real tumor scWGS data whose clonal haplotype structure should be inferred, not simulated — SDX models synthetic diploid chrX only
- The caller under test does not use BAF or phasing information; the 1% phase-switch error injection targets phasing-aware methods specifically
- A benchmark requires realistic female XX or autosome diploid simulation — SDX pools male haploid X chromosomes and this construction does not generalize directly

## Inputs

- Single oligodendrocyte BAM files from two male donors (OligoA and OligoB1)
- Coordinates of regions harboring germline copy-number alterations on chrX (to exclude for a euploid baseline)
- Simulated germline heterozygous SNP (gHET) positions and phased genotypes along chrX
- Design grid of CNA classes (4) and interval sizes (13, 200 kb–5 Mb)

## Outputs

- One SDX BAM per simulated condition (4 x 13 x 100 conditions)
- Ground-truth manifest recording event type, event size, start coordinate, and donor/cell of origin per SDX BAM

## How to apply

Extract reads mapping to chromosome X from single oligodendrocyte BAM files of two male donors using samtools, first removing reads from regions with germline copy-number alterations so cells start from a euploid baseline. For each of 4 CNA classes (single-copy loss, single-copy gain, two-copy gain, cnLOH) and 13 interval sizes spanning 200 kb to 5 Mb, sample 100 start sites and excise/duplicate reads from selected OligoB1 chrX intervals; sampling reads prior to pooling preserves local haplotype structure. Simulate germline heterozygous SNPs (gHETs) along chrX, add Poisson sampling noise to read depth to mimic sequencing stochasticity, and introduce phase-switch errors at a 1% rate into the phased genotypes. Finally, merge modified OligoB1 reads with unmodified OligoA reads into one SDX BAM per condition and write a ground-truth manifest recording event type, size, start coordinate, and donor/cell of origin.

## Related tools

- **samtools** (Extract chrX reads from oligodendrocyte BAM files and manipulate/excise/duplicate reads to create allele-specific copy-number states)
- **HiScanner** (Allele-specific CNA caller benchmarked on the SDX simulated data (combines read depth, BAF, and haplotype phasing)) — github.com/parklab/HiScanner

## Examples

```
samtools view -b OligoB1.bam chrX > OligoB1_chrX.bam
```

## Evaluation signals

- A generated SDX BAM exists for every one of the 4 x 13 x 100 simulated conditions
- Every manifest entry matches the intended CNA type, size (200 kb–5 Mb), and start site
- Phase-switch errors are present at the specified 1% rate in the simulated phased genotypes
- Simulated cells start from a euploid baseline: no reads remain from regions harboring germline copy-number alterations
- Local haplotype structure is preserved at inserted single-allele gain/loss intervals (no unintended disruption from read pooling)

## Limitations

- The simulation pools reads from only two male haploid X chromosomes, restricting generalization to autosomes or XX genomes
- Poisson sampling noise and a fixed 1% phase-switch rate are simplifications; true scWGS amplification artifact structure and phasing error spectra may differ
- Benchmark realism depends on correctly removing germline CNA regions; incomplete removal breaks the euploid baseline assumption

## Evidence

- [methods] Phase-switch error injection at 1% rate: [evidence span withheld — anchor status: not_found]
- [other] SDX haplotype-preserving construction: [evidence span withheld — anchor status: not_found]
- [abstract] Simulation benchmark outcome: "In simulated data, HiScanner consistently outperforms state-of-the-art methods across various CNA types and sizes"
- [readme] HiScanner tool description: "HiScanner is a python package for high-resolution single-cell copy number analysis."
