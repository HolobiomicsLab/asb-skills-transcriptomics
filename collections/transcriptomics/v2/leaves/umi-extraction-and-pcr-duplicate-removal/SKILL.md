---
name: umi-extraction-and-pcr-duplicate-removal
description: 'Use when processing paired-end FASTQ from spatially barcoded ST data
  (e.g., MOB ST, SRA: SRR3382373) to attach barcode/UMI identifiers to read 2 and
  deduplicate PCR artifacts before poly(A) site identification.'
when_to_use_negative:
- Data without UMIs or spatial barcodes (e.g., standard bulk 3'-seq) — umi_tools extract/dedup
  cannot operate as designed
- Non-spatially-barcoded imaging-based ST data such as MERFISH/seqFISH — per the article,
  APA sites can only be identified from spatially barcoded ST data using existing
  scRNA-seq tools like scAPAtrap or Sierra
- When the goal is gene expression quantification rather than poly(A) site-level analysis
  — a standard ST count pipeline suffices
edam_topics:
- http://edamontology.org/topic_3572
tools:
- name: umi_tools
  role: Extracts barcodes and UMIs from read 1, appends them to read 2 headers, and
    performs PCR duplicate removal (dedup) keeping one read per UMI per genomic coordinate
- name: STAR
  role: Aligns read 2 to the GRCm38 reference genome; only uniquely mapped reads are
    retained
- name: scAPAtrap
  role: 'Downstream consumer: identifies poly(A) sites from the deduplicated mapped
    reads'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2023.01.003
    title: 'stAPAminer: Mining Spatial Patterns of Alternative Polyadenylation for
      Spatially Resolved Transcriptomic Studies'
  source_content_hash: sha256:f556e0b4884b472c55a5b4cd51a0d17834c663265c678f798858d260129387cd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - STAR
  - scAPAtrap
  - stAPAminer
  - umi_tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3572
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# umi-extraction-and-pcr-duplicate-removal

## Summary

Pre-alignment preprocessing skill for spatially barcoded ST reads: extract spatial barcodes and UMIs from read 1, append them to read 2 sequence headers with umi_tools, align read 2 with STAR keeping only uniquely mapped reads, and remove PCR duplicates so one read per UMI per genomic coordinate remains before poly(A) site calling.

## When to use

Use when processing paired-end FASTQ from spatially barcoded ST data (e.g., MOB ST, SRA: SRR3382373) to attach barcode/UMI identifiers to read 2 and deduplicate PCR artifacts before poly(A) site identification.

## When NOT to use

- Data without UMIs or spatial barcodes (e.g., standard bulk 3'-seq) — umi_tools extract/dedup cannot operate as designed
- Non-spatially-barcoded imaging-based ST data such as MERFISH/seqFISH — per the article, APA sites can only be identified from spatially barcoded ST data using existing scRNA-seq tools like scAPAtrap or Sierra
- When the goal is gene expression quantification rather than poly(A) site-level analysis — a standard ST count pipeline suffices

## Inputs

- Raw paired-end spatially barcoded ST FASTQ reads (read 1: barcode+UMI; read 2: transcript sequence), e.g. SRA SRR3382373
- GRCm38 mouse reference genome and latest Ensembl genome annotation

## Outputs

- Read 2 FASTQ (or aligned BAM) whose headers carry spatial barcode and UMI information
- Uniquely mapped, PCR-deduplicated BAM (one read per UMI per genomic coordinate) suitable for scAPAtrap poly(A) site identification
- poly(A) site expression table (polyA_sites.tsv)

## How to apply

Start from the raw spatially barcoded MOB ST reads (SRA: SRR3382373); extract barcodes and UMIs from read 1 and use umi_tools to append the barcode and UMI information to the sequence header of read 2, since read 2 carries the transcript sequence. Align the read 2 file to the GRCm38 reference genome with STAR and retain only uniquely mapped reads, discarding multi-mappers. Run umi_tools dedup on the aligned BAM so that one read per UMI for each genomic coordinate remains — this prevents PCR-amplified molecules from inflating poly(A) site counts. Feed the deduplicated BAM into scAPAtrap (extending annotated 3' UTRs by 1000 bp to recruit intergenic sites) to identify poly(A) sites, then build the polyA_sites.tsv expression table. Correctness is judged by whether downstream site counts and correlations reproduce the paper's expectations (e.g., 47,886 sites from ST-MOB; Pearson ~0.7 against SC-MOB).

## Related tools

- **umi_tools** (Extracts barcodes and UMIs from read 1, appends them to read 2 headers, and performs PCR duplicate removal (dedup) keeping one read per UMI per genomic coordinate)
- **STAR** (Aligns read 2 to the GRCm38 reference genome; only uniquely mapped reads are retained)
- **scAPAtrap** (Downstream consumer: identifies poly(A) sites from the deduplicated mapped reads)

## Examples

```
umi_tools extract --stdin read1.fq.gz --read2-in read2.fq.gz --read2-out read2_barcode_umi.fq.gz && umi_tools dedup -I aligned.bam -S deduped.bam
```

## Evaluation signals

- Read 2 sequence headers contain the appended spatial barcode and UMI extracted from read 1
- Alignment output contains only uniquely mapped reads (multi-mappers removed)
- After umi_tools dedup, exactly one read per UMI per genomic coordinate remains
- Downstream scAPAtrap on the deduplicated BAM recovers the reported scale of sites (47,886 poly(A) sites from ST-MOB; 260 spots with ten or more read counts)
- Site-level expression correlates with independent data: expected Pearson ~0.7 vs SC-MOB and ~0.69 vs neural bulk 3'-seq

## Limitations

- No version numbers were reported for any software tool used (umi_tools, STAR, scAPAtrap), complicating exact reproduction
- Only uniquely mapped reads are retained, so multi-mapping reads are discarded entirely
- APA/poly(A) site identification downstream is restricted to spatially barcoded ST data; existing tools cannot handle MERFISH/seqFISH data

## Evidence

- [discussion] barcodes and UMIs were extracted from read 1, and umi_tools [63] was adopted to append the barcode and UMI information to the sequence header of read 2: "barcodes and UMIs were extracted from read 1, and umi_tools [63] was adopted to append the barcode and UMI information to the sequence header of read 2"
- [discussion] PCR duplicates were removed using the dedup function in umi_tools, and one read per UMI for each genomic coordinate remained: "PCR duplicates were removed using the dedup function in umi_tools, and one read per UMI for each genomic coordinate remained"
- [discussion] Reads from the read 2 file were aligned to the reference genome using STAR [64], and only uniquely mapped reads were retained: "Reads from the read 2 file were aligned to the reference genome using STAR [64], and only uniquely mapped reads were retained"
- [other] A total of 47,886 poly(A) sites were extracted from the MOB ST data (ST-MOB) using the scAPAtrap pipeline: [evidence span withheld — anchor status: not_found]
- [readme] APA sites can only be identified from the spatial barcoded ST data for now, using existing scRNA-seq tools like scAPAtrap or Sierra.: [evidence span withheld — anchor status: external_doc]
