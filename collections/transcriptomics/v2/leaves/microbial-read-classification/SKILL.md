---
name: microbial-read-classification
description: Use when you have unmapped reads (e.g., a STAR UnMapped.bam from human
  hg19/hg38 alignment of small RNA-seq FASTQ data) and need to profile fragmental
  microbial RNA content.
when_to_use_negative:
- Reads already mapped to the human genome — these belong to the Annotation module
  (miRBase, piRNABank, gtRNAdb, circBase etc.), not the Microbe module
- Metagenomic shotgun datasets where a dedicated taxonomic classifier is more appropriate
  than BLAST of small-RNA-derived unmapped reads against nt
- When only miRNA/piRNA/tRNA annotation of human-mapped small RNAs is needed and microbial
  content is out of scope
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0659
tools:
- name: COMPSRA
  role: Stand-alone platform whose Microbe module orchestrates BLAST alignment of
    unmapped reads against the UCSC nt database for the four microbial taxons
  repo: https://github.com/cougarlj/COMPSRA
- name: BLAST
  role: Sequence comparison tool that aligns human-unmappable qualified reads to the
    UCSC nt database in the Microbe module
- name: STAR
  role: Default aligner that maps qualified reads to hg38 and produces the UnMapped.bam
    consumed by the Microbe module
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:fcfbf4d4068841fb091884b0df9ad3c96b40a6a0f450352a1f64c16eda977dd3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - BLAST
  - COMPSRA
  - Java
  - STAR
  - exceRpt
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# microbial-read-classification

## Summary

Classify small RNA-seq reads that failed to map to the human genome into microbial taxons (viruses, archaea, bacteria, fungi) by BLASTing them against the UCSC nt database, as implemented in the COMPSRA Microbe module.

## When to use

Use when you have unmapped reads (e.g., a STAR UnMapped.bam from human hg19/hg38 alignment of small RNA-seq FASTQ data) and need to profile fragmental microbial RNA content.

## When NOT to use

- Reads already mapped to the human genome — these belong to the Annotation module (miRBase, piRNABank, gtRNAdb, circBase etc.), not the Microbe module
- Metagenomic shotgun datasets where a dedicated taxonomic classifier is more appropriate than BLAST of small-RNA-derived unmapped reads against nt
- When only miRNA/piRNA/tRNA annotation of human-mapped small RNAs is needed and microbial content is out of scope

## Inputs

- Unmapped reads from the Alignment Module (e.g., sample01_17to50_FitRead_STAR_Aligned_UnMapped.bam)
- Quality-controlled small RNA-seq FASTQ files (50 nt reads, trimmed adapters and 4 random bases at both 5' and 3' ends)
- Prebuilt BLAST microbial databases: blast_archaea, blast_bacteria, blast_fungi, blast_viruses
- BLAST taxonomy information (blast_taxonomy)

## Outputs

- Per-sample microbial species predictions grouped by taxon (archaea, bacteria, fungi, viruses)
- Microbial read counts per sample (e.g., 608 archaea, 103825 bacteria, 45343 fungi, 208 viruses in the 12 serum samples)
- Runtime benchmark entries for the Microbe module (~3.5 hours average across 12 samples)

## How to apply

First run QC and Alignment so that qualified reads are mapped to hg38 with STAR; collect the reads that could not be mapped to the human genome, which COMPSRA saves as a FASTA/UnMapped.bam input for the Microbe module. Download BLAST, taxonomy information, and the prebuilt microbial databases (blast_archaea, blast_bacteria, blast_fungi, blast_viruses) via the COMPSRA ToolKit. Then run the Microbe module with -mic -mtool Blast and one -mdb taxon at a time against the UCSC nucleotide (nt) database, supporting the four major microbial taxons archaea, bacteria, fungi and viruses. For the 12 serum test samples (50 nt reads trimmed to 0–42 nt), the average Microbe module runtime was ~3.5 hours per sample; COMPSRA detected e.g. 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses at a read count threshold of 5.

## Related tools

- **COMPSRA** (Stand-alone platform whose Microbe module orchestrates BLAST alignment of unmapped reads against the UCSC nt database for the four microbial taxons) — https://github.com/cougarlj/COMPSRA
- **BLAST** (Sequence comparison tool that aligns human-unmappable qualified reads to the UCSC nt database in the Microbe module)
- **STAR** (Default aligner that maps qualified reads to hg38 and produces the UnMapped.bam consumed by the Microbe module)

## Examples

```
java -jar COMPSRA.jar -mic -mtool Blast -mdb archaea -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned_UnMapped.bam -out ./example_out/
```

## Evaluation signals

- Per-sample Microbe module runtime approximates the reported ~3.5 hours average across the 12 serum samples (see Supplementary Table S1)
- Output counts are partitioned only into the four supported taxons: archaea, bacteria, fungi and viruses
- Input to the module is exactly the unmapped-read file produced by the Alignment Module (STAR UnMapped output)
- Detected microbial counts at a read count threshold of 5 should be in a comparable range to the published 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses on the 12 serum samples

## Limitations

- The Microbe module is optional and requires downloading additional resources (BLAST, blast_taxonomy, and each prebuilt taxon database) before first use
- Runtime is heavy: ~3.5 hours on average per sample over the 12 test samples because each read is BLASTed against the large UCSC nt database
- Only four major microbial taxons (archaea, bacteria, fungi, viruses) are supported; other taxons are not classified

## Evidence

- [methods] Microbe module: BLAST alignment of unmapped reads to UCSC nt database: "The qualified reads that were not mapped to the human genome in the Alignment Module are aligned to the nucleotide (nt) database27 from UCSC using BLAST"
- [results] Microbe module runtime on test samples: "For the 12 test samples, the estimated time on average is 3.5 hours, including viruses, archaea, bacteria and fungi (see Supplementary Table S1)"
- [results] Microbial detection results: "as well as 608 archaea, 103825 bacteria, 45343 fungi and 208 viruses"
- [readme] Microbe module invocation from README: "java -jar COMPSRA.jar -mic -mtool Blast -mdb archaea -in ./example_out/sample01/sample01_17to50_FitRead_STAR_Aligned_UnMapped.bam -out ./example_out/"
- [readme] Prebuilt microbial databases: "In COMPSRA, we have prebuilt four microbial databases: blast_archaea, blast_bacteria, blast_fungi, blast_viruses."
