---
name: snp-informed-genome-reconstruction
description: Use when allele-specific expression analysis of F1 hybrid (e.g., CAST×B6)
  scRNA-Seq data requires a personalized diploid reference built from known strain
  SNPs/indels, so reads can align to both parental transcriptomes and multi-reads
  can be enumerated for EM-based weighted allocation.
when_to_use_negative:
- The reference genome already matches the sample's strain (no strain-specific variants
  need incorporation) and a single-genome index suffices
- The dataset lacks phase-known parental strain variant calls, so a diploid transcriptome
  cannot be constructed
- The downstream analysis discards multi-reads entirely by design (unique-reads counting),
  for which a strain-informed diploid index is unnecessary
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0196
tools:
- name: g2gtools
  role: Incorporates known SNPs and short indels (Sanger REL-1505) into the reference
    mouse genome sequence to reconstruct the CAST genome
  repo: http://churchill-lab.github.io/g2gtools/
- name: bowtie v1.0.0
  role: Builds the index of the diploid transcriptome and aligns single-cell reads
    with --all --best --strata and -v 3 to enumerate all best alignments
- name: EMASE
  role: 'Downstream consumer of the multi-read alignments: resolves multi-reads via
    weighted allocation with an EM algorithm'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-13099-0
    title: A Bayesian mixture model for the analysis of allelic expression in single
      cells
  source_content_hash: sha256:d7d13a5a7d8606eb850cff3cac82f43756c4ef8a4d9bc4fb834f406712f3ec4f
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# snp-informed-genome-reconstruction

## Summary

Reconstructs a strain-specific (e.g., CAST) genome by incorporating known SNPs and short indels into a reference genome with g2gtools, then builds a bowtie (v1.0.0) index of the phase-known diploid transcriptome. This enables allele-aware alignment of single-cell RNA-Seq reads from F1 hybrid (CAST×B6) cells so that both unique and multi-mapping reads can be assigned to parental alleles.

## When to use

Use when allele-specific expression analysis of F1 hybrid (e.g., CAST×B6) scRNA-Seq data requires a personalized diploid reference built from known strain SNPs/indels, so reads can align to both parental transcriptomes and multi-reads can be enumerated for EM-based weighted allocation.

## When NOT to use

- The reference genome already matches the sample's strain (no strain-specific variants need incorporation) and a single-genome index suffices
- The dataset lacks phase-known parental strain variant calls, so a diploid transcriptome cannot be constructed
- The downstream analysis discards multi-reads entirely by design (unique-reads counting), for which a strain-informed diploid index is unnecessary

## Inputs

- Reference mouse genome sequence (FASTA)
- Known CAST strain SNPs and short indels (Sanger REL-1505 variant annotation)
- Single-cell RNA-Seq reads from the Deng et al. CAST×B6 F1 pre-implantation embryo dataset (286 cells, single-end Illumina HiSeq 2000, GPL12112)

## Outputs

- Reconstructed CAST genome sequence with strain-specific SNPs and indels incorporated
- bowtie (v1.0.0) index representing the phase-known diploid transcriptome
- BAM/SAM alignments of single-cell reads to the diploid transcriptome, with all best-scoring alignments (including multi-reads) enumerated

## How to apply

Take the reference mouse genome sequence and overlay known SNPs and short indels for the relevant strain — here the CAST strain using the Sanger REL-1505 variant set — using g2gtools (http://churchill-lab.github.io/g2gtools/) to produce the reconstructed strain genome. Combine the reference and reconstructed sequences to represent the diploid transcriptome and build a bowtie (v1.0.0) index over it. Align the 286 single-cell reads from the Deng et al. pre-implantation embryo dataset using bowtie with '--all', '--best', and '--strata', allowing three mismatches ('-v 3'), so that all best-scoring alignments — including genomic and allelic multi-reads — are enumerated rather than discarded. The rationale is that a naive single-genome reference collapses alleles and hides allelic multi-reads, which downstream weighted allocation (EMASE) needs as input; discarding them inflates monoallelic calls.

## Related tools

- **g2gtools** (Incorporates known SNPs and short indels (Sanger REL-1505) into the reference mouse genome sequence to reconstruct the CAST genome) — http://churchill-lab.github.io/g2gtools/
- **bowtie v1.0.0** (Builds the index of the diploid transcriptome and aligns single-cell reads with --all --best --strata and -v 3 to enumerate all best alignments)
- **EMASE** (Downstream consumer of the multi-read alignments: resolves multi-reads via weighted allocation with an EM algorithm)

## Evaluation signals

- The bowtie (v1.0.0) index builds successfully over the combined reference + reconstructed CAST transcriptome without missing contigs
- Incorporated variants match the Sanger REL-1505 SNP/indel set; spot-check CAST-specific bases at known variant positions in the reconstructed FASTA
- Alignments run with '--all', '--best', '--strata', '-v 3' return multiple best-scoring alignments for reads overlapping SNPs (multi-reads are retained, not suppressed)
- Downstream, the unique-reads pipeline (discarding all genomic and allelic multi-reads) should retain only ~14.9% of available reads, confirming multi-reads were correctly enumerated upstream

## Limitations

- No software version is stated for g2gtools in the article, which limits exact reproducibility of the reconstruction
- No data accession IDs (e.g., GEO/SRA) are given for the Deng et al. dataset, only platform GPL12112, complicating raw-read retrieval
- Unique-read counting on this pipeline retains only 14.9% of available reads, so a broken or incomplete variant overlay would silently worsen read loss

## Evidence

- [discussion] Reconstruction of CAST genome by SNPs and indels: "We reconstructed the CAST genome by incorporating known SNPs and short indels (Sanger REL-1505) into the reference mouse genome sequence"
- [discussion] bowtie index of diploid transcriptome: "We constructed a bowtie (v1.0.0) index to represent the diploid transcriptome"
- [discussion] bowtie alignment parameters enumerating multi-reads: [evidence span withheld — anchor status: not_found]
- [discussion] g2gtools tool citation: [evidence span withheld — anchor status: not_found]
