---
name: allele-specific-read-counting
description: Use when scRNA-Seq reads from a polymorphic F1 cross (e.g. CAST×B6) must
  be converted into per-allele gene counts, and you must decide between unique-read
  counting and multi-read weighted allocation before allelic classification.
when_to_use_negative:
- The data are bulk RNA-Seq or lack strain-specific polymorphism information, so no
  phase-known diploid transcriptome can be built
- Reads are already allele-resolved UMI counts (e.g. Larsson et al. 224-cell UMI data)
  and multi-read resolution is not the bottleneck
- You only need total expression per gene, not per-allele proportions, and unique
  mapping alone would suffice
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: g2gtools
  role: Incorporates known SNPs and short indels (Sanger REL-1505) into the reference
    mouse genome to reconstruct the CAST haplotype
  repo: http://churchill-lab.github.io/g2gtools/
- name: bowtie v1.0.0
  role: Aligns single-cell reads to the diploid transcriptome index with --all --best
    --strata and -v 3 so multi-reads are enumerated
- name: EMASE
  role: Resolves multi-reads via EM-based weighted allocation using the estimated
    probability of each alignment
- name: scBASE
  role: Consumes the resulting allele-specific count matrices for beta-binomial mixture
    classification of allelic expression states
  repo: https://github.com/churchill-lab/scBASE
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:4d0cde6a171c652a3d7016e3fa39500fbf74bac0afdc586f60c2926c9a20d105
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMASE
  - Stan
  - bowtie
  - bowtie v1.0.0
  - g2gtools
  - scBASE
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# allele-specific-read-counting

## Summary

Generates allele-specific read count matrices (maternal/paternal per gene per cell) from F1-hybrid single-cell RNA-Seq by aligning to a phase-known diploid transcriptome and resolving multi-mapping reads either by discarding them (unique-reads) or by EM-based weighted allocation, since counting choice drives downstream monoallelic and dynamic expression calls.

## When to use

Use when scRNA-Seq reads from a polymorphic F1 cross (e.g. CAST×B6) must be converted into per-allele gene counts, and you must decide between unique-read counting and multi-read weighted allocation before allelic classification.

## When NOT to use

- The data are bulk RNA-Seq or lack strain-specific polymorphism information, so no phase-known diploid transcriptome can be built
- Reads are already allele-resolved UMI counts (e.g. Larsson et al. 224-cell UMI data) and multi-read resolution is not the bottleneck
- You only need total expression per gene, not per-allele proportions, and unique mapping alone would suffice

## Inputs

- Single-cell RNA-Seq FASTQ reads from F1 hybrid cells (e.g. Deng et al. CAST×B6, 286 pre-implantation embryo cells)
- Reference mouse genome plus known SNP/indel annotation (Sanger REL-1505) for diploid genome reconstruction
- bowtie (v1.0.0) index of the phase-known diploid transcriptome
- Gene/transcript annotation to assign alignments to genes

## Outputs

- Unique-reads allele-specific count matrix (maternal/paternal reads per gene per cell, multi-reads discarded)
- Weighted-allocation allele-specific count matrix produced by the EMASE EM algorithm
- Per-method summary of reads retained, monoallelic genes called per cell, and bi-allelic genes missed (e.g. allelic_counts_comparison.csv)

## How to apply

Reconstruct the alternate haplotype (CAST) by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome with g2gtools, then build a bowtie v1.0.0 index of the diploid transcriptome. Align single-cell reads with bowtie using --all, --best, and --strata and allowing three mismatches (-v 3) so all best-scoring alignments, including multi-reads, are enumerated. For the unique-reads method, discard all genomic and allelic multi-reads and count only reads mapping unambiguously to one allele of one gene; this retains only ~14.9% of available reads. For weighted allocation, run the EM algorithm in EMASE, which resolves each multi-read with weights based on the estimated probability of each alignment, preserving far more information. Rationale: the paper shows discarding multi-reads inflates monoallelic calls (~66 more per cell) and misses ~1,908 bi-allelic genes versus weighted allocation, so counting method must be reported alongside any ASE classification (e.g. scBASE states where an allele is expressed only if estimated abundance exceeds one read).

## Related tools

- **g2gtools** (Incorporates known SNPs and short indels (Sanger REL-1505) into the reference mouse genome to reconstruct the CAST haplotype) — http://churchill-lab.github.io/g2gtools/
- **bowtie v1.0.0** (Aligns single-cell reads to the diploid transcriptome index with --all --best --strata and -v 3 so multi-reads are enumerated)
- **EMASE** (Resolves multi-reads via EM-based weighted allocation using the estimated probability of each alignment)
- **scBASE** (Consumes the resulting allele-specific count matrices for beta-binomial mixture classification of allelic expression states) — https://github.com/churchill-lab/scBASE

## Examples

```
bowtie -v 3 --all --best --strata diploid_transcriptome_index reads.fastq > aln.bowtie
```

## Evaluation signals

- Unique-reads counting should retain only ~14.9% of available reads; a much higher retention suggests multi-reads were not correctly discarded
- Unique-reads vs weighted-allocation contrast should reproduce monoallelic inflation (~66 more monoallelic genes per cell) and ~1,908 bi-allelic genes missed, e.g. Mtdh monoallelic in 88 (unique-reads) vs 7 (weighted-allocation) cells
- Alignment must use --all --best --strata with -v 3; restricting to a single best alignment silently removes allelic multi-reads
- Output matrices must span all cells (e.g. 286 Deng et al. cells) with maternal/paternal counts per gene and share gene/cell row-column structure across counting methods for paired comparison

## Limitations

- The unique-reads method leads to higher variability in allelic proportion estimates, more sampling zeros, and spurious findings of dynamic and monoallelic gene expression
- No software version is stated for EMASE, RSEM, kallisto, or g2gtools, so exact multi-read weight reproducibility is uncertain
- Read counting via RSEM or kallisto is suggested as an alternative but the paper's weighted-allocation results were produced specifically with the EMASE EM algorithm

## Evidence

- [intro] we exclude multi-mapping reads (multi-reads) and count only reads that map unambiguously to one allele of one gene: "we exclude multi-mapping reads (multi-reads) and count only reads that map unambiguously to one allele of one gene"
- [discussion] We aligned reads using bowtie with parameters '–all', '–best', and '–strata', allowing for three mismatches ('-v 3'): [evidence span withheld — anchor status: not_found]
- [discussion] multi-reads are resolved by computing a weighted allocation based on the estimated probability of each alignment: "multi-reads are resolved by computing a weighted allocation based on the estimated probability of each alignment"
- [discussion] We reconstructed the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome sequence: "We reconstructed the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome sequence"
