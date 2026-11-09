---
name: multi-read-preserving-alignment-parameterization
description: Use when assigning single-cell RNA-Seq reads to parental alleles in a
  CAST×B6 F1 hybrid and you must preserve multi-mapping reads, since discarding them
  inflates allelic-proportion variability and produces spurious monoallelic calls.
when_to_use_negative:
- 'The unique-reads analysis mode is intended as the contrast: if you deliberately
  want to exclude multi-mapping reads and count only reads mapping unambiguously to
  one allele of one gene, do not apply this multi-read-preserving parameterization'
- Data are not from a phase-known cross with strain-specific variants (e.g., no CAST/B6
  SNP panel), so a diploid strain-specific transcriptome cannot be constructed
- Reads are paired-end UMI counts needing transcript-level quantification with tools
  like RSEM or kallisto rather than allele-tagged bowtie alignment
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: g2gtools
  role: Incorporates known SNPs and short indels (Sanger REL-1505) into the GRCm38
    reference to reconstruct the CAST/EiJ genome from which the CAST-specific transcriptome
    is built
  repo: http://churchill-lab.github.io/g2gtools/
- name: bowtie v1.0.0
  role: Builds the diploid transcriptome index and aligns single-end reads with '-v
    3 --all --best --strata' to retain genomic and allelic multi-reads
- name: EMASE
  role: Downstream EM-based weighted allocation that resolves the retained multi-reads
    into allele-specific counts
- name: scBASE
  role: Consumes the resulting allele-specific counts for mixture-model classification
    of allelic expression states (P/B/M)
  repo: https://github.com/kbchoi-jax/scbase
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:77d0b4473e8863bb78cf1f76cca9b9d5ce5b6c8d9ee7284abdcf2273cfdfebb4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - EMASE
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
yes_rate: 0.8823529411764706
n_claims: 17
---

# multi-read-preserving-alignment-parameterization

## Summary

An allele-specific read-alignment strategy for scRNA-Seq ASE analysis in which single-end reads are aligned to a phase-known diploid (CAST/B6) transcriptome with bowtie v1.0.0 using '-v 3 --all --best --strata', deliberately retaining genomic and allelic multi-reads for downstream weighted allocation instead of discarding them.

## When to use

Use when assigning single-cell RNA-Seq reads to parental alleles in a CAST×B6 F1 hybrid and you must preserve multi-mapping reads, since discarding them inflates allelic-proportion variability and produces spurious monoallelic calls.

## When NOT to use

- The unique-reads analysis mode is intended as the contrast: if you deliberately want to exclude multi-mapping reads and count only reads mapping unambiguously to one allele of one gene, do not apply this multi-read-preserving parameterization
- Data are not from a phase-known cross with strain-specific variants (e.g., no CAST/B6 SNP panel), so a diploid strain-specific transcriptome cannot be constructed
- Reads are paired-end UMI counts needing transcript-level quantification with tools like RSEM or kallisto rather than allele-tagged bowtie alignment

## Inputs

- Single-end scRNA-Seq FASTQ reads from CAST×B6 F1 hybrid cells (e.g., Deng et al. pre-implantation embryo cells, Illumina HiSeq 2000, platform GPL12112)
- GRCm38 reference mouse genome sequence
- CAST-specific SNP and short indel calls (Sanger REL-1505)
- Annotated transcript sequences from Ensembl Release 78

## Outputs

- Diploid CAST/B6 transcriptome FASTA with maternal/paternal copy labels
- bowtie v1.0.0 index representing the diploid transcriptome
- Allele-specific alignment files (parental-origin tagged) retaining genomic and allelic multi-reads, for downstream allele-specific counting

## How to apply

First reconstruct the CAST/EiJ genome by incorporating known SNPs and short indels (Sanger REL-1505) into the GRCm38 reference with g2gtools, then build a CAST-specific transcriptome and concatenate it with the B6 transcriptome (Ensembl Release 78) into one diploid FASTA carrying maternal/paternal copy labels. Construct a bowtie (v1.0.0) index over this diploid transcriptome and align the single-end Deng et al. embryo reads with '-v 3' (up to three mismatches) plus '--all --best --strata' so that all best-scoring alignments, including genomic and allelic multi-reads, are reported. The rationale is that discarding multi-reads leads to higher variability in allelic proportions, more sampling zeros, and spurious dynamic/monoallelic findings; the retained allele-tagged alignments can later be resolved by the EM-based weighted allocation in EMASE (or a unique-reads baseline for comparison). Validation is by confirming the bowtie index builds successfully and that alignment files exist for each Deng et al. cell with reads aligned under the '-v 3 --all --best --strata' parameterization.

## Related tools

- **g2gtools** (Incorporates known SNPs and short indels (Sanger REL-1505) into the GRCm38 reference to reconstruct the CAST/EiJ genome from which the CAST-specific transcriptome is built) — http://churchill-lab.github.io/g2gtools/
- **bowtie v1.0.0** (Builds the diploid transcriptome index and aligns single-end reads with '-v 3 --all --best --strata' to retain genomic and allelic multi-reads)
- **EMASE** (Downstream EM-based weighted allocation that resolves the retained multi-reads into allele-specific counts)
- **scBASE** (Consumes the resulting allele-specific counts for mixture-model classification of allelic expression states (P/B/M)) — https://github.com/kbchoi-jax/scbase

## Examples

```
bowtie diploid_transcriptome_index cell_reads.fq -v 3 --all --best --strata -S allele_specific_alignments.sam
```

## Evaluation signals

- The bowtie index over the diploid transcriptome builds successfully without errors
- Alignment output files exist for every Deng et al. cell with reads aligned under '-v 3 --all --best --strata'
- Alignments carry parental-origin allele tags suitable for downstream allele-specific counting
- Multi-reads are present in the output (not filtered), since the parameterization must report all best-scoring alignments rather than a single hit
- The diploid FASTA contains both maternal and paternal copy labels for each annotated transcript

## Limitations

- The article states no software version for g2gtools (or EMASE, RSEM, kallisto, scphaser), which may hinder exact reproduction of the genome reconstruction step
- Allowing up to three mismatches ('-v 3') is specific to these single-end Deng et al. reads; the parameterization may need re-tuning for other read lengths or error profiles
- No data accession IDs (e.g., GEO/SRA) are given for the scRNA-Seq datasets, complicating acquisition of identical input FASTQs

## Evidence

- [discussion] Reconstruction of CAST genome by incorporating SNPs and indels into reference genome with g2gtools: "We reconstructed the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome sequence"
- [discussion] bowtie alignment parameterization retaining multi-reads: [evidence span withheld — anchor status: not_found]
- [discussion] bowtie v1.0.0 index over diploid transcriptome: "We constructed a bowtie (v1.0.0) index to represent the diploid transcriptome"
- [abstract] Rationale for retaining multi-reads: "discarding multi-mapping reads leads to higher variability in estimates of allelic proportions, an increased frequency of sampling zeros"
- [intro] Unique-reads contrast excludes multi-reads: "In the unique-reads method, we exclude multi-mapping reads (multi-reads)"
