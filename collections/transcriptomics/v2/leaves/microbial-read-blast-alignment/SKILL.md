---
name: microbial-read-blast-alignment
description: Use when small RNA-seq FASTQ reads have been mapped to hg19/hg38 with
  STAR and an unmapped BAM/FASTA remains, and you need microbial taxon read counts
  for the four supported taxons archaea, bacteria, fungi and viruses.
when_to_use_negative:
- Input is a mapped BAM or annotated small RNA counts — the Microbe module expects
  only qualified reads that were NOT mapped to the human genome
- The target microbes fall outside the four supported taxons (archaea, bacteria, fungi,
  viruses)
- You need whole-metagenomic assembly or strain-level profiling rather than fragmental
  microbial RNA read counting against UCSC nt
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3174
tools:
- name: COMPSRA
  role: Runs the Microbe module (-mic) that BLAST-aligns unmapped reads and aggregates
    taxon read counts
  repo: https://github.com/cougarlj/COMPSRA
- name: BLAST
  role: Sequence comparison tool aligning unmapped small RNA reads to the UCSC nt
    database
- name: STAR
  role: Upstream aligner producing the unmapped reads (saved into a FASTA file) that
    feed the Microbe module
- name: UCSC nt database
  role: Nucleotide reference database against which unmapped reads are BLAST-aligned
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:de1a4b73fd8ac1c5eca26a9ed206ed61d13fec652ec2a48d692a2a37aae382ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - BLAST
  - COMPSRA
  - STAR
  - UCSC nt database
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# microbial-read-blast-alignment

## Summary

Profiles microbial small RNA (archaea, bacteria, fungi, viruses) by BLAST-aligning reads that failed to map to the human genome (e.g. *_Aligned_UnMapped.bam from STAR) against UCSC nt prebuilt microbial databases within COMPSRA's Microbe module.

## When to use

Use when small RNA-seq FASTQ reads have been mapped to hg19/hg38 with STAR and an unmapped BAM/FASTA remains, and you need microbial taxon read counts for the four supported taxons archaea, bacteria, fungi and viruses.

## When NOT to use

- Input is a mapped BAM or annotated small RNA counts — the Microbe module expects only qualified reads that were NOT mapped to the human genome
- The target microbes fall outside the four supported taxons (archaea, bacteria, fungi, viruses)
- You need whole-metagenomic assembly or strain-level profiling rather than fragmental microbial RNA read counting against UCSC nt

## Inputs

- Unmapped reads from the Alignment module (e.g. *_Aligned_UnMapped.bam; unmapped reads are saved into a FASTA file for input into the Microbe Module)
- Prebuilt microbial BLAST databases (blast_archaea, blast_bacteria, blast_fungi, blast_viruses)
- BLAST taxonomy information (blast_taxonomy)

## Outputs

- Per-taxon microbial read counts (tabulated read count per taxon) for archaea, bacteria, fungi and viruses
- Microbe module output directory with BLAST-based taxon assignments per sample

## How to apply

First use COMPSRA ToolKit to install BLAST, taxonomy information and the prebuilt microbial databases (blast_archaea, blast_bacteria, blast_fungi, blast_viruses). Then run the Microbe module with -mic -mtool Blast -mdb archaea,bacteria,fungi,viruses, feeding the qualified reads that were not mapped to the human genome (the Alignment module's *_Aligned_UnMapped.bam) so they are aligned to the UCSC nucleotide (nt) database via BLAST. Because of interspecies homology, hits must be aggregated by listing all potential taxa with their read counts according to the phylogenetic tree, restricted to the four supported taxons, and parsed with COMPSRA's fast text access/parsing algorithm. On the 12 healthy human serum samples this approach averaged ~3.5 hours per run and reported 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses.

## Related tools

- **COMPSRA** (Runs the Microbe module (-mic) that BLAST-aligns unmapped reads and aggregates taxon read counts) — https://github.com/cougarlj/COMPSRA
- **BLAST** (Sequence comparison tool aligning unmapped small RNA reads to the UCSC nt database)
- **STAR** (Upstream aligner producing the unmapped reads (saved into a FASTA file) that feed the Microbe module)
- **UCSC nt database** (Nucleotide reference database against which unmapped reads are BLAST-aligned)

## Examples

```
java -jar COMPSRA.jar -mic -mtool Blast -mdb archaea,bacteria,fungi,viruses -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned_UnMapped.bam -out ./example_out/
```

## Evaluation signals

- Reproduce the reported serum-sample totals: 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses across the 12 healthy human serum samples
- Output counts are restricted to exactly the four supported taxons: archaea, bacteria, fungi and viruses
- Every input read in the Microbe run corresponds to a read that failed human-genome mapping in the Alignment module (no mapped reads leak in)
- Microbiome module runtime for the 12 test samples averages around the reported ~3.5 hours including viruses, archaea, bacteria and fungi
- Taxon read counts reflect aggregation across all potential taxa per read due to interspecies homology, not single best-hit assignment

## Limitations

- Interspecies homology means BLAST hits must be aggregated over all potential taxa via the phylogenetic tree; species-level precision is limited
- Only four microbial taxons are supported (archaea, bacteria, fungi, viruses); additional resource downloads (BLAST, taxonomy, prebuilt databases) are required before the module can run
- The module is optional and slow relative to the rest of the pipeline (~3.5 hours on average for the 12 test samples)

## Evidence

- [methods] Microbe module BLAST of unmapped reads to UCSC nt: "The qualified reads that were not mapped to the human genome in the Alignment Module are aligned to the nucleotide (nt) database27 from UCSC using BLAST"
- [readme] Microbe module supports four taxons: "The four major microbial taxons archaea, bacteria, fungi and viruses are supported."
- [readme] Unmapped reads saved for Microbe module: "Reads that could not be mapped to the human genome are saved into a FASTA file for input into the Microbe Module."
- [results] Detected microbial counts in serum samples: "as well as 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses"
- [results] Microbiome module runtime: "For the 12 test samples, the estimated time on average is 3.5 hours, including viruses, archaea, bacteria and fungi"
