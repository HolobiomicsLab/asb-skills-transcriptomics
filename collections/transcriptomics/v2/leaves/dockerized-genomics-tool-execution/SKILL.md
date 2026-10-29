---
name: dockerized-genomics-tool-execution
description: Use when you need to execute VIsoQLR on long-read sequencing alignments
  (BAM/GFF3/BED6) for isoform identification and quantification of a single gene or
  minigene, and want a reproducible containerized run on any OS including Windows/WSL2.
when_to_use_negative:
- Input is not long-read sequencing data for a single gene/locus or minigene splicing
  assay (VIsoQLR is tailored to low-throughput, one-gene-at-a-time analysis)
- Splice sites are already known and only a genome-annotation-based transcriptome-level
  analysis is needed (e.g., full transcriptome discovery with wf-transcriptomes/bambu)
- No Docker support is available and containerized execution is not possible on the
  target system
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: VIsoQLR
  role: Interactive analyzer, viewer and editor executed in Docker for isoform detection,
    quantification and fine-tuning from long-read data
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: GMAP
  role: Aligner used inside the container to map raw FASTQ long reads against a user-built
    reference index, producing BAM output
- name: Minimap2
  role: Alternative aligner to GMAP for FASTA references; used with caution because
    it has documented problems with small exons
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:05d8f55a1876d96d80e8ecdd8520c45b28a4bed8912d8a9689203acc00a06646
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - GMAP
  - Minimap2
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# dockerized-genomics-tool-execution

## Summary

Run VIsoQLR inside its official Docker container (tblabfjd/visoqlr) so the interactive isoform detection, viewing and editing web app executes in a reproducible Linux environment with port 8888 mapped, bypassing local dependency installation.

## When to use

Use when you need to execute VIsoQLR on long-read sequencing alignments (BAM/GFF3/BED6) for isoform identification and quantification of a single gene or minigene, and want a reproducible containerized run on any OS including Windows/WSL2.

## When NOT to use

- Input is not long-read sequencing data for a single gene/locus or minigene splicing assay (VIsoQLR is tailored to low-throughput, one-gene-at-a-time analysis)
- Splice sites are already known and only a genome-annotation-based transcriptome-level analysis is needed (e.g., full transcriptome discovery with wf-transcriptomes/bambu)
- No Docker support is available and containerized execution is not possible on the target system

## Inputs

- Aligned long reads in BAM format with BAI index (or GFF3 / BED6 alignments)
- Raw FASTQ reads plus reference sequence(s) in FASTA for GMAP alignment inside the container
- Docker image tblabfjd/visoqlr:latest pulled from Docker Hub

## Outputs

- Identified and quantified isoform tables (ranked isoform abundances and lengths) via the VIsoQLR web interface at port 8888
- GMAP-produced mapped reads (BAM) and index (BAI) when aligning raw FASTQ inside the container
- Editable consensus splice site graphics and tables for manual curation

## How to apply

Install Docker (on Windows, install Docker Desktop with WSL2 since the image targets the Linux kernel), then pull the image with 'docker pull tblabfjd/visoqlr:latest' and launch it with 'docker run -it -p 8888:8888 tblabfjd/visoqlr:latest'. Open the web interface at port 8888 and upload the aligned reads (BAM plus BAI index, or GFF3/BED6) so VIsoQLR automatically runs its default isoform identification and quantification pipeline, including the default 2% read threshold for consensus exon coordinate (CEC) detection. If starting from raw FASTQ, use the container's GMAP panel: first upload the reference sequence(s) to build the GMAP index, then align the FASTQ reads with GMAP (preferred over Minimap2 for small exons) and download the resulting BAM and BAI. Fine-tune parameters such as 'Read threshold (%)', 'Padding (# of bases)', and 'Merge close splice sites (# of bases)' through the interface when isoforms are missing or spurious.

## Related tools

- **VIsoQLR** (Interactive analyzer, viewer and editor executed in Docker for isoform detection, quantification and fine-tuning from long-read data) — https://github.com/TBLabFJD/VIsoQLR
- **GMAP** (Aligner used inside the container to map raw FASTQ long reads against a user-built reference index, producing BAM output)
- **Minimap2** (Alternative aligner to GMAP for FASTA references; used with caution because it has documented problems with small exons) — https://github.com/lh3/minimap2

## Examples

```
docker pull tblabfjd/visoqlr:latest; docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- Docker image pull and container start succeed and the VIsoQLR web interface is reachable at localhost:8888
- After uploading a BAM, VIsoQLR produces consensus splice sites and a ranked isoform abundance table without errors
- On the SIRV control dataset, detected isoform counts and cosine similarities match published values (e.g., 65 isoforms at 0.25% threshold, cosine similarity 0.81)
- On the PAX6 minigene assay with defaults, VIsoQLR reports 40 isoforms (9 above 1%) with the canonical isoform at 53.1% abundance and 776 bp
- GMAP alignment step yields both a BAM and its BAI index downloadable from the container interface

## Limitations

- The Docker image is developed for the Linux kernel; Windows users must install Linux via WSL2 before running the container
- Automatic analysis with the default 2% read threshold can miss isoforms below that abundance, requiring manual parameter tuning of thresholds, padding, and splice-site merge distance
- Minimap2 has documented problems with small exons, so GMAP is the preferred aligner inside the workflow

## Evidence

- [methods] Pull the VIsoQLR Docker image: "docker pull tblabfjd/visoqlr:latest"
- [methods] Run container with port mapping: "docker run -it -p 8888:8888 tblabfjd/visoqlr:latest"
- [methods] Windows WSL2 requirement: "For windows users, as the image has been developed for Linux kernel, it is necessary to install Linux on Windows with WSL2"
- [methods] Upload alignment formats: "navigate to your file system to upload your aligner reads in `GFF3`, `BED6`, or `BAM` file format."
- [methods] GMAP index and alignment panels in the container: "The first one is used to build the reference index used by GMAP by uploading the reference sequence(s)."
- [readme] Tool purpose (readme): "VIsoQLR is an interactive analyzer, viewer and editor for the semi-automated identification and quantification of known and novel isoforms using long-read sequencing data."
