---
name: sequencing-noise-modeling
description: Use when building simulated single-cell sequencing benchmarks that must
  preserve haplotype structure and amplification artifacts while challenging BAF/phasing-based
  CNA callers with realistic noise.
when_to_use_negative:
- The target is detecting real somatic CNAs in experimental scWGS data rather than
  benchmarking caller sensitivity on known ground truth
- The simulated cells should reflect clonal CNAs shared across a population of cells
  — SDX embeds non-clonal, single-cell allele-specific CNAs in haploid-pooled reads
- 'Sex chromosomes are out of scope: SCAN2, a dependency in the HiScanner pipeline,
  explicitly does not support sex chromosomes, and the simulation strategy relies
  on male haploid X pooling'
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: samtools
  role: Extract reads mapping to chromosome X from single oligodendrocyte BAM files
    and manipulate read intervals for CNA insertion
- name: HiScanner
  role: Downstream allele-specific CNA caller whose benchmarking motivates the noise
    model (Poisson depth noise, 1% phase-switch errors)
  repo: github.com/parklab/HiScanner
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:a119b42cedff8545223f9dfac09114cc31aef546fca12bcb0f906f7046d6c898
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HiScanner
  - samtools
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# sequencing-noise-modeling

## Summary

Construct realistic simulated scWGS benchmarks by embedding stochastic sequencing noise (Poisson read-depth sampling) and phasing imperfections (phase-switch errors at a controlled 1% rate) into pooled haploid read data, so that allele-specific CNA callers like HiScanner are evaluated under realistic error conditions.

## When to use

Use when building simulated single-cell sequencing benchmarks that must preserve haplotype structure and amplification artifacts while challenging BAF/phasing-based CNA callers with realistic noise.

## When NOT to use

- The target is detecting real somatic CNAs in experimental scWGS data rather than benchmarking caller sensitivity on known ground truth
- The simulated cells should reflect clonal CNAs shared across a population of cells — SDX embeds non-clonal, single-cell allele-specific CNAs in haploid-pooled reads
- Sex chromosomes are out of scope: SCAN2, a dependency in the HiScanner pipeline, explicitly does not support sex chromosomes, and the simulation strategy relies on male haploid X pooling

## Inputs

- Single oligodendrocyte BAM files from two male donors (OligoA and OligoB1), chrX reads extracted with samtools
- Regions harboring germline copy-number alterations to exclude from the euploid baseline
- Simulated phased gHET genotypes along chrX

## Outputs

- One SDX BAM per simulated condition (4 CNA classes x 13 interval sizes x 100 start sites)
- Ground-truth manifest recording event type, event size (200 kb–5 Mb), start coordinate, and donor/cell of origin per BAM
- Simulated phased genotypes with 1% phase-switch error rate

## How to apply

Pool reads from two male haploid X chromosomes (OligoA and OligoB1 oligodendrocyte BAMs extracted with samtools, excluding germline CNA regions to start from a euploid baseline), then sample reads prior to pooling so single-allele copy gains or losses can be inserted without disrupting local haplotype structure. Simulate germline heterozygous SNPs (gHETs) along chrX and introduce Poisson sampling noise into the read depth to mimic sequencing stochasticity. Introduce phase-switch errors at a 1% error rate into the simulated phased genotypes so that BAF/phasing-based callers are realistically challenged. Merge modified OligoB1 reads with unmodified OligoA reads per simulated condition and record event type, size, and start coordinate in a ground-truth manifest.

## Related tools

- **samtools** (Extract reads mapping to chromosome X from single oligodendrocyte BAM files and manipulate read intervals for CNA insertion)
- **HiScanner** (Downstream allele-specific CNA caller whose benchmarking motivates the noise model (Poisson depth noise, 1% phase-switch errors)) — github.com/parklab/HiScanner

## Examples

```
samtools view -b oligoB1.bam chrX | samtools view -b -e 'not in_germline_cna_region' - | python insert_cna_and_noise.py --event single_copy_loss --size 1Mb --phase_switch_rate 0.01 > SDX_event.bam
```

## Evaluation signals

- Every generated SDX BAM exists for all 4 x 13 x 100 simulated conditions (CNA class x interval size x start site)
- Each ground-truth manifest entry matches the intended CNA type, size within 200 kb–5 Mb, and start site
- Phase-switch errors are present at the 1% error rate in the simulated phased genotypes
- Local haplotype structure is preserved around inserted allele-specific copy-number events (read sampling done prior to pooling)
- Poisson sampling noise in read depth reproduces scWGS amplification-artifact-like fluctuations without altering the intended allele-specific copy state

## Limitations

- Amplification artifacts in single-cell whole-genome amplification can confound CNA detection even at high sequencing depth, so simulated noise may not capture all real WGA artifacts despite Poisson modeling
- The simulation is restricted to chromosome X of male donors to exploit haploid pooling; extending to autosomes requires a different haplotype-pooling strategy
- FDR estimates from such simulations are conservative because FDR increases as CNA size decreases

## Evidence

- [methods] Poisson read-depth noise: [evidence span withheld — anchor status: not_found]
- [methods] phase-switch error rate: [evidence span withheld — anchor status: not_found]
- [results] read sampling before pooling preserves haplotype: [evidence span withheld — anchor status: not_found]
- [intro] WGA artifacts confound detection: "Even if the cell is sequenced at high depth, these amplification artifacts can confound CNA detection by causing large fluctuations in the read depth ratio"
