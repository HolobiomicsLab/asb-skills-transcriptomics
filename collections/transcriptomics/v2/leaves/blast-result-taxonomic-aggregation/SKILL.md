---
name: blast-result-taxonomic-aggregation
description: Use when small RNA-seq reads unmapped to the human genome (e.g. *_Aligned_UnMapped.bam)
  have been BLAST-aligned to the UCSC nt database and per-taxon microbial read counts
  must be tabulated.
when_to_use_negative:
- The reads of interest were mapped to the human genome — those are handled by the
  Annotation module against miRBase, piRNABank/piRBase/piRNACluster, gtRNAdb, GENCODE
  release 27 and circBase, not by Microbe.
- You need species-level or strain-level taxon resolution beyond the four supported
  taxons (archaea, bacteria, fungi, viruses).
- Input is not FASTQ/SAM/BAM small RNA-seq data processed through COMPSRA's QC and
  Alignment modules.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0659
tools:
- name: COMPSRA
  role: Runs the Microbe module (-mic) that drives BLAST alignment and aggregates
    BLAST hits into per-taxon counts; installed via COMPSRA ToolKit (-tk -dr).
  repo: https://github.com/cougarlj/COMPSRA
- name: BLAST
  role: Aligns unmapped reads to the UCSC nt nucleotide database; installed via 'java
    -jar COMPSRA.jar -tk -dr -ck blast'.
- name: STAR
  role: Produces the upstream unmapped-read BAM/FASTA by mapping qualified reads to
    hg19/hg38 in the Alignment module.
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:6d73143e7f1e4bfa6612ee508f2766a6a56a2aed66dd97cd6be0a2956bd8c785
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - BLAST
  - COMPSRA
  - STAR
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# blast-result-taxonomic-aggregation

## Summary

Aggregate BLAST alignments of unmapped small RNA-seq reads against the UCSC nt database into per-taxon read counts for the four COMPSRA microbial taxons (archaea, bacteria, fungi, viruses), accounting for interspecies homology.

## When to use

Use when small RNA-seq reads unmapped to the human genome (e.g. *_Aligned_UnMapped.bam) have been BLAST-aligned to the UCSC nt database and per-taxon microbial read counts must be tabulated.

## When NOT to use

- The reads of interest were mapped to the human genome — those are handled by the Annotation module against miRBase, piRNABank/piRBase/piRNACluster, gtRNAdb, GENCODE release 27 and circBase, not by Microbe.
- You need species-level or strain-level taxon resolution beyond the four supported taxons (archaea, bacteria, fungi, viruses).
- Input is not FASTQ/SAM/BAM small RNA-seq data processed through COMPSRA's QC and Alignment modules.

## Inputs

- Unmapped read output of the COMPSRA Alignment module (e.g. sample01_17to50_FitRead_STAR_Aligned_UnMapped.bam / FASTA of reads not mapped to hg19/hg38)
- Prebuilt COMPSRA microbial BLAST databases (blast_archaea, blast_bacteria, blast_fungi, blast_viruses)
- BLAST taxonomy information (blast_taxonomy)

## Outputs

- Per-taxon read counts for archaea, bacteria, fungi and viruses in each sample
- Tabulation of all potential taxa with read counts resolved via the phylogenetic tree

## How to apply

First install BLAST plus taxonomy data and the prebuilt microbial databases (blast_archaea, blast_bacteria, blast_fungi, blast_viruses) via the COMPSRA ToolKit download commands (java -jar COMPSRA.jar -tk -dr -ck blast, blast_taxonomy, blast_archaea). Run the Microbe module with -mic -mtool Blast -mdb archaea,bacteria,fungi,viruses, taking the qualified reads that were not mapped to the human genome in the Alignment Module as input. Because of interspecies homology, do not assign each read to a single species: instead, list all potential taxa with their read counts according to the phylogenetic tree, restricted to the four supported taxons. Parse the BLAST output with COMPSRA's fast text access/parsing algorithm and tabulate read counts per taxon. On the 12 healthy human serum samples this runtime averaged about 3.5 hours covering all four taxons.

## Related tools

- **COMPSRA** (Runs the Microbe module (-mic) that drives BLAST alignment and aggregates BLAST hits into per-taxon counts; installed via COMPSRA ToolKit (-tk -dr).) — https://github.com/cougarlj/COMPSRA
- **BLAST** (Aligns unmapped reads to the UCSC nt nucleotide database; installed via 'java -jar COMPSRA.jar -tk -dr -ck blast'.)
- **STAR** (Produces the upstream unmapped-read BAM/FASTA by mapping qualified reads to hg19/hg38 in the Alignment module.)

## Examples

```
java -jar COMPSRA.jar -mic -mtool Blast -mdb archaea,bacteria,fungi,viruses -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned_UnMapped.bam -out ./example_out/
```

## Evaluation signals

- Reproduce the reported serum-sample totals: 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses across the 12 healthy human serum samples.
- Only the four supported taxons (archaea, bacteria, fungi, viruses) appear in the output counts.
- Read counts are aggregated across all homologous potential taxa (phylogenetic-tree listing) rather than a single best hit per read.
- For the 12 test samples, total runtime averages ~3.5 hours when all four taxons are included; large deviations suggest misconfiguration.
- Output is a tab-delimited per-taxon read count table per sample, consistent with COMPSRA's count-profile conventions.

## Limitations

- Only four microbial taxons are supported (archaea, bacteria, fungi, viruses).
- Interspecies homology means reads cannot be uniquely assigned to a single species; counts are reported for all potential taxa.
- The Microbe module is optional and requires additional large resource downloads (BLAST, taxonomy, prebuilt nt-derived databases); estimated runtime is ~3.5 hours average for 12 samples.
- Reads that map to the human genome are excluded, so only fragmental microbial RNA outside human mapping is captured.

## Evidence

- [methods] Microbe module: BLAST alignment of unmapped reads to UCSC nt database: "The qualified reads that were not mapped to the human genome in the Alignment Module are aligned to the nucleotide (nt) database27 from UCSC using BLAST"
- [results] Serum microbe counts finding: "COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs, as well as 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses"
- [readme] Microbe module description: "The four major microbial taxons archaea, bacteria, fungi and viruses are supported."
- [readme] Microbe module invocation example: "java -jar COMPSRA.jar -mic -mtool Blast -mdb archaea -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned_UnMapped.bam -out ./example_out/"
