---
name: diploid-transcriptome-alignment
description: Use when single-end scRNA-Seq reads from an F1 hybrid (e.g. CAST×B6)
  must be assigned to parental alleles without discarding multi-mapping reads, as
  a prerequisite for ASE counting and classification.
when_to_use_negative:
- Data are from a non-hybrid, single-genome sample where allelic origin of reads is
  not of interest — a standard single-genome transcriptome index suffices
- The analysis plan requires discarding multi-mapping reads (unique-reads counting
  only) — while the same alignment can be reused, this skill's purpose is to retain
  multi-reads, so alignment runs that filter them (e.g. default bowtie without --all)
  defeat the goal
- Input reads are paired-end or UMI-based protocols incompatible with the bowtie v1.0.0
  single-end '-v 3' alignment mode described here
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: g2gtools
  role: Incorporate known SNPs and short indels (Sanger REL-1505) into the reference
    mouse genome to reconstruct the CAST genome
  repo: http://churchill-lab.github.io/g2gtools/
- name: bowtie v1.0.0
  role: Index the diploid transcriptome and align single-end reads with --all --best
    --strata and -v 3 to retain multi-reads
- name: EMASE
  role: Downstream EM-based weighted allocation of the retained multi-reads to alleles
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:b254dfce6aa9a0502a7db3135cb4518c631cff6a5dd1ece84791107f50603892
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# diploid-transcriptome-alignment

## Summary

Constructs a phase-known diploid transcriptome (CAST + B6) and aligns single-cell reads to it with bowtie, retaining all best-scoring alignments so that both genomic and allelic multi-reads are preserved for downstream allele-specific counting.

## When to use

Use when single-end scRNA-Seq reads from an F1 hybrid (e.g. CAST×B6) must be assigned to parental alleles without discarding multi-mapping reads, as a prerequisite for ASE counting and classification.

## When NOT to use

- Data are from a non-hybrid, single-genome sample where allelic origin of reads is not of interest — a standard single-genome transcriptome index suffices
- The analysis plan requires discarding multi-mapping reads (unique-reads counting only) — while the same alignment can be reused, this skill's purpose is to retain multi-reads, so alignment runs that filter them (e.g. default bowtie without --all) defeat the goal
- Input reads are paired-end or UMI-based protocols incompatible with the bowtie v1.0.0 single-end '-v 3' alignment mode described here

## Inputs

- Single-end FASTQ reads from Deng et al. pre-implantation embryo cells (CAST×B6 F1 hybrid, 286 cells)
- GRCm38 reference mouse genome sequence
- Known CAST SNPs and short indels (Sanger REL-1505 variant catalog)
- Ensembl Release 78 transcript annotations for building the B6 and CAST transcriptomes

## Outputs

- Diploid transcriptome FASTA with maternal/paternal (CAST/B6) copy labels
- bowtie v1.0.0 index of the diploid transcriptome
- Allele-specific alignment files (alignments tagged by parental origin) retaining all best-scoring alignments including multi-reads

## How to apply

First reconstruct the CAST/EiJ genome by incorporating known SNPs and short indels (Sanger REL-1505) into the GRCm38 reference genome using g2gtools, then build the CAST-specific transcriptome and concatenate it with the B6 transcriptome (Ensembl Release 78) into a single diploid transcriptome FASTA with maternal/paternal copy labels. Build a bowtie (v1.0.0) index over this diploid transcriptome. Align the single-end Deng et al. embryo reads with bowtie using '--all', '--best', and '--strata' and allowing three mismatches ('-v 3') so that all best-scoring alignments, including multi-reads, are enumerated rather than discarded. Emit allele-specific alignments tagged by parental origin for downstream counting (unique-reads vs EMASE weighted allocation). Rationale: keeping multi-reads is essential because discarding them inflates monoallelic calls; whether to later resolve multi-reads by EM (EMASE) or drop them is a separate downstream decision.

## Related tools

- **g2gtools** (Incorporate known SNPs and short indels (Sanger REL-1505) into the reference mouse genome to reconstruct the CAST genome) — http://churchill-lab.github.io/g2gtools/
- **bowtie v1.0.0** (Index the diploid transcriptome and align single-end reads with --all --best --strata and -v 3 to retain multi-reads)
- **EMASE** (Downstream EM-based weighted allocation of the retained multi-reads to alleles)

## Examples

```
bowtie -v 3 --all --best --strata diploid_transcriptome reads.fastq > allelic_alignments.sam
```

## Evaluation signals

- The g2gtools CAST reconstruction completes and the diploid transcriptome FASTA contains both CAST and B6 copies from Ensembl Release 78 with maternal/paternal labels
- The bowtie v1.0.0 index builds successfully on the diploid transcriptome (file_exists check)
- Alignment output files exist for the Deng et al. cells and reads were aligned under the '-v 3 --all --best --strata' parameterization (script_runs check)
- Alignment records include genomic and allelic multi-reads (multiple alignments per read), confirming multi-reads were not discarded
- Downstream sanity check: the unique-reads count derived from these alignments retains only 14.9% of available reads, matching the published figure

## Limitations

- No software version is stated for EMASE or g2gtools, only bowtie v1.0.0 is pinned, limiting exact reproducibility
- Retaining multi-reads is only the first stage: the unique-reads downstream path discards them (retaining just 14.9% of reads), so the alignment's value depends on choosing weighted allocation
- No data accession IDs (e.g. GEO/SRA) are given for the Deng et al. dataset, only platform GPL12112

## Evidence

- [discussion] We reconstructed the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome sequence: "We reconstructed the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome sequence"
- [discussion] We constructed a bowtie (v1.0.0) index to represent the diploid transcriptome: "We constructed a bowtie (v1.0.0) index to represent the diploid transcriptome"
- [discussion] We aligned reads using bowtie with parameters '–all', '–best', and '–strata', allowing for three mismatches ('-v 3'): [evidence span withheld — anchor status: not_found]
- [discussion] unique-read method discards genomic and allelic multi-reads: "Unique-read counts are obtained directly from counting reads after discarding all genomic and allelic multi-reads"
- [results] The alignment stage works by constructing a diploid transcriptome from CAST- and B6-specific sequences of each annotated transcript (Ensembl Release 78): [evidence span withheld — anchor status: not_found]
