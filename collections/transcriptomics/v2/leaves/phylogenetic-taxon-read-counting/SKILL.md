---
name: phylogenetic-taxon-read-counting
description: Use when you have small RNA-seq reads that failed to map to the human
  genome (e.g. STAR *_Aligned_UnMapped.bam) and need microbial species-level read
  counts per taxon.
when_to_use_negative:
- Reads already mapped to the human genome — those go to the Annotation module, not
  the Microbe module
- You need shotgun metagenomic abundance profiling with per-sample relative abundance
  normalization rather than small RNA fragment microbial read counts
- Your target taxon is outside the four supported taxons (archaea, bacteria, fungi,
  viruses)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0659
tools:
- name: COMPSRA
  role: Runs the Microbe module (-mic) that aligns unmapped reads via BLAST and parses
    results into per-taxon read counts
  repo: https://github.com/cougarlj/COMPSRA
- name: BLAST
  role: Alignment tool used to align unmapped qualified reads to the UCSC nt database
- name: STAR
  role: Produces the unmapped reads output consumed by the Microbe module via the
    Alignment module
- name: UCSC nt database
  role: Nucleotide reference database that unmapped reads are aligned against for
    microbial identification
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:3c8d197ded94287bed19bdc7c56710109f95ddaf52d655d0af37191b76ea3de9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - BLAST
  - COMPSRA
  - STAR
  - UCSC nt database
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# phylogenetic-taxon-read-counting

## Summary

Aggregate BLAST hits of human-unmapped small RNA-seq reads against the UCSC nt database into per-taxon read counts across the four supported microbial taxons (archaea, bacteria, fungi, viruses), using the phylogenetic tree to list all potential taxa because of interspecies homology.

## When to use

Use when you have small RNA-seq reads that failed to map to the human genome (e.g. STAR *_Aligned_UnMapped.bam) and need microbial species-level read counts per taxon.

## When NOT to use

- Reads already mapped to the human genome — those go to the Annotation module, not the Microbe module
- You need shotgun metagenomic abundance profiling with per-sample relative abundance normalization rather than small RNA fragment microbial read counts
- Your target taxon is outside the four supported taxons (archaea, bacteria, fungi, viruses)

## Inputs

- Unmapped reads BAM/FASTA from the Alignment module (e.g. *_Aligned_UnMapped.bam) for each sample
- BLAST executable and blast_taxonomy files from COMPSRA ToolKit
- Prebuilt microbial BLAST databases: blast_archaea, blast_bacteria, blast_fungi, blast_viruses

## Outputs

- Per-taxon read count table for archaea, bacteria, fungi and viruses in each sample
- BLAST alignment results of unmapped reads against the UCSC nt database

## How to apply

Take the qualified reads that were not mapped to the human genome, saved by the Alignment module as an unmapped BAM/FASTA (e.g. *_Aligned_UnMapped.bam). Install BLAST, the taxonomy information, and the four prebuilt microbial databases (blast_archaea, blast_bacteria, blast_fungi, blast_viruses) via the COMPSRA ToolKit download commands. Run the Microbe module with -mic -mtool Blast -mdb archaea,bacteria,fungi,viruses so the unmapped reads are aligned to the UCSC nt database with BLAST. Because of interspecies homology, aggregate the BLAST hits by listing all potential taxa with their read counts according to the phylogenetic tree rather than assigning each read to a single best hit. Parse the BLAST output with COMPSRA's fast text-parsing algorithm and tabulate read count per taxon; validate against the reported 12-serum-sample totals of 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses.

## Related tools

- **COMPSRA** (Runs the Microbe module (-mic) that aligns unmapped reads via BLAST and parses results into per-taxon read counts) — https://github.com/cougarlj/COMPSRA
- **BLAST** (Alignment tool used to align unmapped qualified reads to the UCSC nt database)
- **STAR** (Produces the unmapped reads output consumed by the Microbe module via the Alignment module)
- **UCSC nt database** (Nucleotide reference database that unmapped reads are aligned against for microbial identification)

## Examples

```
java -jar COMPSRA.jar -mic -mtool Blast -mdb archaea,bacteria,fungi,viruses -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned_UnMapped.bam -out ./example_out/
```

## Evaluation signals

- Reproduce the reported serum-sample totals: 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses across the 12 samples
- Only reads that failed human genome mapping appear as BLAST input (counts match the Alignment module's unmapped-read FASTA)
- Every reported taxon falls within the four supported taxons archaea, bacteria, fungi, viruses
- Multiple homologous taxa may share reads for a single read — the output lists all potential taxa per the phylogenetic tree, not a single best hit

## Limitations

- Only the four taxons archaea, bacteria, fungi and viruses are supported
- Interspecies homology means read counts are distributed over all potential taxa in the phylogenetic tree, so counts are not exclusive per species
- The microbiome module is slow: ~3.5 hours on average across the 12 test samples including viruses, archaea, bacteria and fungi
- Requires downloading additional resources (BLAST, taxonomy, prebuilt microbial databases) beyond the base COMPSRA install

## Evidence

- [methods] Microbe module: BLAST alignment of unmapped reads to UCSC nt database for microbial profiling: "The qualified reads that were not mapped to the human genome in the Alignment Module are aligned to the nucleotide (nt) database27 from UCSC using BLAST"
- [results] Reported microbial counts for 12 serum samples: "COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs, as well as 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses"
- [readme] Microbe module description and supported taxons: "The qualified reads that could not be mapped to the human genome in the Alignment Module are aligned to the nucleotide (nt) database from UCSC using BLAST. The four major microbial taxons archaea,"
- [readme] Microbe module command example: "java -jar COMPSRA.jar -mic -mtool Blast -mdb archaea -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned_UnMapped.bam -out ./example_out/"
- [results] Microbiome module runtime: "For the 12 test samples, the estimated time on average is 3.5 hours, including viruses, archaea, bacteria and fungi"
