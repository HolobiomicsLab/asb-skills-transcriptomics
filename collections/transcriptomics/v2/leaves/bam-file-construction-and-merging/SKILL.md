---
name: bam-file-construction-and-merging
description: Use when benchmarking allele-specific CNA callers requires simulated
  single-cell scWGS BAMs with known ground-truth events (200 kb–5 Mb), preserved haplotype
  structure, and realistic amplification artifacts.
when_to_use_negative:
- If the goal is calling CNAs on real single-cell data rather than generating ground-truth
  simulation data for benchmarking allele-specific CNA callers
- If the input BAMs are not haploid-sex-chromosome-derived (e.g., female cells or
  autosomes), since the method relies on pooling two male haploid X chromosomes to
  synthesize a diploid baseline
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: samtools
  role: Extracts reads mapping to chromosome X from oligodendrocyte BAM files and
    merges modified OligoB1 reads with unmodified OligoA reads into per-condition
    SDX BAMs
- name: HiScanner
  role: Downstream allele-specific CNA caller that the SDX benchmark BAMs are used
    to evaluate
  repo: https://github.com/parklab/HiScanner
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60446-5
    title: High-resolution detection of copy number alterations in single cells with
      HiScanner
  source_content_hash: sha256:fb87e5555f751f18b3dc3c38aec24b929e0746c2018ed01b98c235633b6820b0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/parklab/HiScanner
  tools:
  - HiScanner
  - samtools
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# BAM file construction and merging

## Summary

Constructs synthetic diploid X (SDX) single-cell BAM files by extracting chrX reads from two male haploid oligodendrocyte BAMs with samtools, excising/duplicating reads to embed allele-specific CNAs, and merging modified OligoB1 reads with unmodified OligoA reads into one SDX BAM per simulated condition.

## When to use

Use when benchmarking allele-specific CNA callers requires simulated single-cell scWGS BAMs with known ground-truth events (200 kb–5 Mb), preserved haplotype structure, and realistic amplification artifacts.

## When NOT to use

- If the goal is calling CNAs on real single-cell data rather than generating ground-truth simulation data for benchmarking allele-specific CNA callers
- If the input BAMs are not haploid-sex-chromosome-derived (e.g., female cells or autosomes), since the method relies on pooling two male haploid X chromosomes to synthesize a diploid baseline

## Inputs

- Single oligodendrocyte BAM files from two male donors (OligoA, OligoB1)
- Germline copy-number alteration region definitions for chrX (to exclude)
- Simulated gHET genotypes along chrX with phase-switch errors at a 1% rate
- Sampling specifications: 4 CNA classes x 13 interval sizes (200 kb–5 Mb) x 100 start sites

## Outputs

- One SDX BAM per simulated condition (4 x 13 x 100 BAMs)
- Ground-truth manifest recording event type, event size, start coordinate, and donor/cell of origin for every SDX BAM

## How to apply

Extract reads mapping to chromosome X from single oligodendrocyte BAM files of the two male donors using samtools, and remove reads from regions harboring germline copy-number alterations so each simulated cell starts from a euploid baseline. For each of the 4 CNA classes (single-copy loss, single-copy gain, two-copy gain, cnLOH) across 13 interval sizes spanning 200 kb to 5 Mb, sample 100 start sites and excise or duplicate reads from the selected OligoB1 chrX intervals to create the allele-specific copy-number state, sampling reads prior to pooling so local haplotype structure is not disrupted. Introduce Poisson sampling noise into read depth, simulate germline heterozygous SNPs, and add phase-switch errors at a 1% rate to the phased genotypes. Finally, merge the modified OligoB1 reads with unmodified OligoA reads to write one SDX BAM per simulated condition, and record event type, size, start coordinate, and donor/cell of origin in a ground-truth manifest. Rationale: pooling two haploid X chromosomes reproduces a synthetic diploid genome in which single-allele copy gains or losses can be inserted while phasing accuracy, phase-switch rates, and scWGS amplification artifacts remain controllable and preserved.

## Related tools

- **samtools** (Extracts reads mapping to chromosome X from oligodendrocyte BAM files and merges modified OligoB1 reads with unmodified OligoA reads into per-condition SDX BAMs)
- **HiScanner** (Downstream allele-specific CNA caller that the SDX benchmark BAMs are used to evaluate) — https://github.com/parklab/HiScanner

## Examples

```
samtools view -b oligoB1.bam chrX > oligoB1_chrX.bam && samtools merge oligoA_chrX.bam oligoB1_chrX.modified.bam > sdx_condition.bam
```

## Evaluation signals

- Every generated SDX BAM exists for all 4 x 13 x 100 simulated conditions
- Each ground-truth manifest entry matches the intended CNA type, size (200 kb–5 Mb), and start site
- Phase-switch errors are present at the 1% rate in the simulated phased genotypes
- Reads from regions harboring germline copy-number alterations are absent from the extracted chrX baseline (euploid starting state)
- Haplotype structure around inserted events is preserved because reads were sampled prior to pooling

## Limitations

- Simulation is restricted to chromosome X of two male donors, so results may not generalize to autosomes
- Poisoning of benchmarks by misspecified ground truth is possible if excision/duplication start sites overlap unremoved germline CNA regions
- The synthetic baseline still inherits donor-specific scWGS amplification artifacts, which are realistic but not controllable in magnitude

## Evidence

- [methods] The SDX approach pools reads from two male haploid X chromosomes, and by sampling reads prior to pooling, single-allele copy gains or losses can be inserted without disrupting local haplotype structure: [evidence span withheld — anchor status: not_found]
- [methods] Extract reads mapping to chromosome X from the single oligodendrocyte BAM files of the two male donors using samtools: [evidence span withheld — anchor status: not_found]
- [methods] Merge the modified OligoB1 reads with the unmodified OligoA reads and write one SDX BAM per simulated condition: [evidence span withheld — anchor status: not_found]
- [methods] verify that each generated SDX BAM exists for all 4 x 13 x 100 simulated conditions: [evidence span withheld — anchor status: not_found]
