---
name: long-read-rna-seq-alignment
description: Use when you have raw long-read RNA FASTQ data (PacBio Iso-Seq, ONT)
  and unspliced reference sequences (FASTA) and need spliced alignments (BAM+BAI)
  as input for VIsoQLR isoform analysis.
when_to_use_negative:
- Reads are already aligned and you have a BAM file — proceed directly to VIsoQLR's
  main analysis view instead of re-aligning
- Reference annotation is a genome + GTF and you need a genome-guided pipeline like
  StringTie2/bambu; VIsoQLR's GMAP panel expects unspliced FASTA reference sequences
- Short-read data where spliced alignment to a genome with a standard spliced short-read
  aligner is more appropriate
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3512
- http://edamontology.org/topic_3308
tools:
- name: VIsoQLR
  role: Docker-based interactive tool hosting the GMAP alignment panel; builds the
    GMAP index and returns BAM+BAI for isoform detection
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: GMAP
  role: Aligner used inside VIsoQLR to map raw FASTQ long reads against the built
    reference index
- name: Minimap2
  role: Alternative aligner callable from VIsoQLR using a FASTA reference (the same
    file used to build the GMAP index)
  repo: https://github.com/lh3/minimap2
- name: Docker
  role: Container runtime for tblabfjd/visoqlr (docker pull tblabfjd/visoqlr:latest)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:bf71d9ef8ae45cbbf65b2e22bc206da7a91f2f9be3beefb08d04efa86f4fad2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - Docker
  - GMAP
  - Minimap2
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3512
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# long-read-rna-seq-alignment

## Summary

Align long-read RNA sequencing reads (e.g., PacBio UHR Iso-Seq FASTQ) to a reference using GMAP inside the VIsoQLR Docker container, producing a mapped BAM file plus BAI index suitable for downstream isoform detection and quantification.

## When to use

Use when you have raw long-read RNA FASTQ data (PacBio Iso-Seq, ONT) and unspliced reference sequences (FASTA) and need spliced alignments (BAM+BAI) as input for VIsoQLR isoform analysis.

## When NOT to use

- Reads are already aligned and you have a BAM file — proceed directly to VIsoQLR's main analysis view instead of re-aligning
- Reference annotation is a genome + GTF and you need a genome-guided pipeline like StringTie2/bambu; VIsoQLR's GMAP panel expects unspliced FASTA reference sequences
- Short-read data where spliced alignment to a genome with a standard spliced short-read aligner is more appropriate

## Inputs

- Raw long-read RNA sequencing reads in FASTQ format (e.g., public PacBio UHR Iso-Seq dataset)
- Unspliced reference sequence(s) in FASTA format (e.g., MANE Ensembl 108 sequences for PAX6 and TP53)

## Outputs

- GMAP reference index ZIP
- Mapped reads in BAM format
- BAM index (BAI) file

## How to apply

In the VIsoQLR Docker container, first upload the reference sequence(s) in FASTA format to the GMAP panel so VIsoQLR builds the GMAP reference index automatically, then download the index ZIP. Next, upload the reference index ZIP together with the raw reads FASTQ file, select the number of threads, and run GMAP mapping. Retrieve the mapped reads as BAM plus its BAI index, which can be uploaded directly to the main VIsoQLR analysis view where consensus exon coordinates (CECs) are detected at a chosen Read threshold (%) (e.g., 3% for the PAX6/TP53 PacBio UHR analysis). Minimap2 is offered as an alternative aligner, but the reference must be in FASTA format (the same file used to build the GMAP index).

## Related tools

- **VIsoQLR** (Docker-based interactive tool hosting the GMAP alignment panel; builds the GMAP index and returns BAM+BAI for isoform detection) — https://github.com/TBLabFJD/VIsoQLR
- **GMAP** (Aligner used inside VIsoQLR to map raw FASTQ long reads against the built reference index)
- **Minimap2** (Alternative aligner callable from VIsoQLR using a FASTA reference (the same file used to build the GMAP index)) — https://github.com/lh3/minimap2
- **Docker** (Container runtime for tblabfjd/visoqlr (docker pull tblabfjd/visoqlr:latest))

## Examples

```
docker pull tblabfjd/visoqlr:latest; docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- The GMAP panel successfully builds and returns a downloadable reference index ZIP from the uploaded FASTA reference sequence(s)
- The GMAP run completes and both download buttons appear: mapped reads (BAM) and its index (BAI)
- Uploading the resulting BAM to VIsoQLR's main view yields expected mapped read counts (e.g., 45 mapped reads for PAX6, 450 for TP53 in the PacBio UHR validation)
- Isoform quantification on the mapped BAM reproduces reference values (PAX6 canonical isoform 9 reads at 45%; TP53 top isoform 166 reads at 88.8%) at a 3% read threshold
- The reference file used for Minimap2 is the same FASTA file used to build the GMAP index

## Limitations

- Minimap2 has documented limitations that are advised in its GitHub (https://github.com/lh3/minimap2)
- On Windows, the VIsoQLR image is developed for the Linux kernel, so WSL2 (Linux on Windows) is required before running the container
- The Read threshold (%) trade-off: lowering it detects more isoforms but with the cost of including noise-level isoforms; the default is 2%

## Evidence

- [methods] Build GMAP reference index from reference sequence(s): "The first one is used to build the reference index used by GMAP by uploading the reference sequence(s)."
- [methods] Align raw reads with GMAP and retrieve BAM+BAI: "The second panel is dedicated to align the raw reads (in FASTQ format) using GMAP."
- [methods] BAM and BAI download outputs: "two download bottoms will appear: the mapped reads (BAM) and its index (BAI)."
- [methods] Minimap2 requires the same FASTA reference: "In this case the reference must be in FASTA format (the same file used to build the GMAP index in the first panel)."
- [readme] Docker image acquisition and run: "A docker image can be downloaded from https://hub.docker.com/r/tblabfjd/visoqlr"
