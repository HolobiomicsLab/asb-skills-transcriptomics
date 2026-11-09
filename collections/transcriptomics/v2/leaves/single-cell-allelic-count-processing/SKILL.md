---
name: single-cell-allelic-count-processing
description: Use when scRNA-Seq reads from a CAST×B6-style F1 hybrid must be converted
  into maternal/paternal allelic count matrices (e.g., 122 blastocyst cells, 8,290
  genes) that feed bursting or ASE-state models, and multi-mapping reads would otherwise
  be discarded.
when_to_use_negative:
- Input is already a per-allele count matrix or UMI table with unique allelic assignment
  (e.g., the Larsson et al. UMI counts) — the counting/allocation step is already
  done and only expression filtering applies
- Bulk RNA-Seq or non-hybrid samples where no allele-of-origin can be distinguished
  — the unique-reads/weighted-allocation dichotomy and partial pooling across allelic
  states are not applicable
- High-coverage data where multi-reads are rare — discarding them (unique-reads) introduces
  little variability and partial pooling may have negligible impact
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3520
- http://edamontology.org/topic_0203
tools:
- name: scBASE
  role: Implements the counting/classification pipeline producing allelic count matrices,
    beta-binomial mixture classification into P/B/M states, and partial pooling of
    allelic proportions
  repo: https://github.com/kbchoi-jax/scbase
- name: EMASE
  role: EM algorithm that computes weighted allocation of multi-mapping reads to alleles
    for the weighted count matrices
- name: bowtie v1.0.0
  role: Aligns reads to the diploid transcriptome index with --all --best --strata
    -v 3 (three mismatches allowed)
- name: g2gtools
  role: Reconstructs the CAST genome by incorporating known SNPs and short indels
    (Sanger REL-1505) into the reference mouse genome
- name: RSEM
  role: Alternative software for read counting via weighted allocation, mentioned
    as interchangeable with EMASE-style counting
- name: kallisto
  role: Alternative software for read counting via weighted allocation of multi-reads
- name: scphaser
  role: One possible approach for phasing/haplotyping cells in the allelic workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:600d99043031f1b7e708bdd20846c4bcb3cdb91061ab83ff28de5b59f01aeb1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kbchoi-jax/scbase
  tools:
  - EMASE
  - RSEM
  - SCALE
  - bowtie v1.0.0
  - g2gtools
  - kallisto
  - scBASE
  - scphaser
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3520
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# single-cell-allelic-count-processing

## Summary

Process allele-specific read alignments from scRNA-Seq into per-cell, per-gene allelic count matrices for F1-hybrid data by choosing between unique-reads counting and EM-based weighted allocation of multi-reads, optionally followed by partial pooling, to reduce sampling zeros and allelic-proportion variability before downstream modeling.

## When to use

Use when scRNA-Seq reads from a CAST×B6-style F1 hybrid must be converted into maternal/paternal allelic count matrices (e.g., 122 blastocyst cells, 8,290 genes) that feed bursting or ASE-state models, and multi-mapping reads would otherwise be discarded.

## When NOT to use

- Input is already a per-allele count matrix or UMI table with unique allelic assignment (e.g., the Larsson et al. UMI counts) — the counting/allocation step is already done and only expression filtering applies
- Bulk RNA-Seq or non-hybrid samples where no allele-of-origin can be distinguished — the unique-reads/weighted-allocation dichotomy and partial pooling across allelic states are not applicable
- High-coverage data where multi-reads are rare — discarding them (unique-reads) introduces little variability and partial pooling may have negligible impact

## Inputs

- scRNA-Seq FASTQ reads from F1-hybrid single cells (e.g., 122 mouse blastocyst cells from Deng et al., CAST×B6)
- Phase-known diploid transcriptome index built with bowtie v1.0.0 and a CAST genome reconstructed by g2gtools (SNPs/indels, Sanger REL-1505)
- Allele-specific alignment files mapping reads to maternal/paternal haplotypes
- Optional per-cell ASE-state classifications (P/B/M) from the scBASE beta-binomial mixture for partial pooling

## Outputs

- Allele-specific count matrices (maternal and paternal counts per gene per cell) under the unique-reads method
- Allele-specific count matrices under EM-based weighted allocation of multi-reads
- Partially pooled allelic proportion matrices (weighted averages of class-specific proportions) for each counting scheme
- Estimated abundances with alleles flagged as expressed when estimated abundance > 1 read (or 1 UMI)

## How to apply

First reconstruct the alternate haplotype (e.g., CAST genome from SNPs/indels via g2gtools) and align reads to a phase-known diploid transcriptome with bowtie v1.0.0 using '--all --best --strata -v 3'. Then count allele-specific alignments under one of two schemes: the unique-reads method, which discards all genomic and allelic multi-reads, or weighted allocation via the EM algorithm in EMASE, which probabilistically splits multi-reads. If coverage is low and cell number is large (e.g., 122 blastocyst cells), apply partial pooling by replacing each cell's allelic proportions with a weighted average of class-specific proportions across cells in the same ASE state. Treat an allele as expressed only if its estimated abundance exceeds one read (or one UMI for UMI-based data like Larsson et al.). Feed the resulting four scBASE-estimated matrices (unique, weighted, unique+partial-pooling, weighted+partial-pooling) into downstream models such as SCALE or the beta-binomial mixture classification into P/B/M states.

## Related tools

- **scBASE** (Implements the counting/classification pipeline producing allelic count matrices, beta-binomial mixture classification into P/B/M states, and partial pooling of allelic proportions) — https://github.com/kbchoi-jax/scbase
- **EMASE** (EM algorithm that computes weighted allocation of multi-mapping reads to alleles for the weighted count matrices)
- **bowtie v1.0.0** (Aligns reads to the diploid transcriptome index with --all --best --strata -v 3 (three mismatches allowed))
- **g2gtools** (Reconstructs the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome)
- **RSEM** (Alternative software for read counting via weighted allocation, mentioned as interchangeable with EMASE-style counting)
- **kallisto** (Alternative software for read counting via weighted allocation of multi-reads)
- **scphaser** (One possible approach for phasing/haplotyping cells in the allelic workflow)

## Examples

```
git clone git://github.com/kbchoi-jax/scbase
```

## Evaluation signals

- Discarding multi-reads (unique-reads method) should visibly increase variability of allelic proportion estimates and frequency of sampling zeros compared with weighted allocation — if not, the allocation step likely failed
- Down-sampling check: randomly sampling 1% of reads in each of the 122 blastocyst cells should show higher mean squared error for unique-reads than weighted-allocation estimates
- An allele should be counted as expressed only when estimated abundance > 1 read (or > 1 UMI); genes with reads but no allelic reads should be treated as missing rather than zero
- For the downstream SCALE analysis on 122 blastocyst cells, the four matrices over 8,290 genes should reproduce 3,381 (unique), 4,815 (weighted), 6,068 (unique+PP), and 6,761 (weighted+PP) significant genes at 5% FDR, and |logOR|>2 counts of 2,845/3,763 before pooling and 5,622/6,209 after pooling
- Partial pooling should have the biggest impact (largest shift in significant gene counts) when read coverage is low and the number of cells is large

## Limitations

- Discarding multi-mapping reads leads to higher variability in allelic proportions, more sampling zeros, and can produce spurious findings of dynamic and monoallelic expression
- Multi-reads assigned only probabilistically under weighted allocation carry allocation uncertainty that is not propagated as sampling variance in the count matrices
- Expression threshold (abundance > 1 read or 1 UMI) and ASE class boundaries (e.g., πg⋅s > 0.7) are arbitrary, limiting reproducibility of gene classification
- Software versions for EMASE, RSEM, kallisto, and g2gtools are not stated in the article; MCMC details are deferred to Supplementary Methods
- Weighted allocation was not applied to the Larsson et al. UMI data (txburst pipeline), so the two counting schemes are not universally applicable to all input types

## Evidence

- [intro] We first count the allele-specific read alignments using one of two methods. In the unique-reads method, we exclude multi-mapping reads (multi-reads): "We first count the allele-specific read alignments using one of two methods. In the unique-reads method, we exclude multi-mapping reads (multi-reads)"
- [discussion] multi-reads are resolved by computing a weighted allocation based on the estimated probability of each alignment: "multi-reads are resolved by computing a weighted allocation based on the estimated probability of each alignment"
- [intro] Lastly, we can apply partial pooling to improve the individual cell-level estimation by combining information across cells that are in the same allelic expression state.: "Lastly, we can apply partial pooling to improve the individual cell-level estimation by combining information across cells that are in the same allelic expression state."
- [discussion] We reconstructed the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome sequence: "We reconstructed the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome sequence"
- [discussion] We aligned reads using bowtie with parameters '–all', '–best', and '–strata', allowing for three mismatches ('-v 3'): [evidence span withheld — anchor status: not_found]
- [discussion] We randomly sampled 1% of reads in each of 122 cells at the early, mid, and late blastocyst stages: "We randomly sampled 1% of reads in each of 122 cells at the early, mid, and late blastocyst stages"
- [discussion] we regarded it as expressed if its estimated abundance is greater than one reads (or one UMI as in Larsson et al.22): "we regarded it as expressed if its estimated abundance is greater than one reads (or one UMI as in Larsson et al.22)"
