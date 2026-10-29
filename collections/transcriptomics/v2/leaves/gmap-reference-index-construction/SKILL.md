---
name: gmap-reference-index-construction
description: Use when raw long-read FASTQ data must be aligned to unspliced reference
  sequences (e.g. MANE Ensembl 108) with GMAP as the first step of a VIsoQLR isoform
  analysis workflow.
when_to_use_negative:
- Input is already aligned (BAM, GFF3, or BED6) — in that case skip alignment and
  upload directly to the main VIsoQLR analysis view.
- Reference is a full genome and the goal is whole-transcriptome discovery — VIsoQLR
  is tailored to low-throughput, gene-at-a-time splicing assays rather than genome-scale
  pipelines.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: VIsoQLR
  role: Interactive tool whose GMAP panel automates reference index construction from
    uploaded FASTA reference sequence(s) and then runs GMAP alignment of FASTQ reads
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: GMAP
  role: Spliced aligner used to align long cDNA reads to the unspliced reference index,
    producing mapped reads (BAM/BAI or GFF3 exon coordinates)
- name: Docker
  role: Container runtime used to run the VIsoQLR image containing the GMAP index-building
    and alignment panels
- name: Minimap2
  role: Alternative aligner offered by VIsoQLR when a FASTA reference is used directly
    instead of a pre-built GMAP index
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:2df5a41df4aca983ab98d851c52900e6722b59913c74b0758b6a2a5e78ad748c
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
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gmap-reference-index-construction

## Summary

Build a GMAP reference index from unspliced reference sequences (FASTA) inside the VIsoQLR Docker container so that long cDNA reads (FASTQ) can be spliced-aligned to a gene locus prior to isoform detection and quantification.

## When to use

Use when raw long-read FASTQ data must be aligned to unspliced reference sequences (e.g. MANE Ensembl 108) with GMAP as the first step of a VIsoQLR isoform analysis workflow.

## When NOT to use

- Input is already aligned (BAM, GFF3, or BED6) — in that case skip alignment and upload directly to the main VIsoQLR analysis view.
- Reference is a full genome and the goal is whole-transcriptome discovery — VIsoQLR is tailored to low-throughput, gene-at-a-time splicing assays rather than genome-scale pipelines.

## Inputs

- Unspliced reference sequence(s) in FASTA format (e.g. MANE Ensembl 108 gene sequences such as PAX6 and TP53)
- Raw long-read sequencing data in FASTQ format (e.g. PacBio UHR Iso-Seq reads)
- VIsoQLR Docker image (tblabfjd/visoqlr:latest)

## Outputs

- GMAP reference index downloadable as a ZIP file
- Mapped reads as a BAM file plus BAI index
- Per-read exon coordinates (GFF3 when GMAP is run with --format=2 in the Mini-IsoQLR variant)

## How to apply

Inside the VIsoQLR Docker container (pulled as tblabfjd/visoqlr:latest and run with port 8888 mapped), navigate to the GMAP panel and upload the reference sequence(s) in FASTA format; VIsoQLR then builds the GMAP reference index automatically. Download the resulting reference index ZIP, then supply it together with the FASTQ reads file and a chosen number of threads to the alignment panel to run GMAP mapping. The same FASTA file can also serve as the reference if aligning with Minimap2 instead of GMAP. Rationale: GMAP with --format=2 outputs a GFF3 containing exon coordinates of all reads, which downstream VIsoQLR analysis (consensus exon coordinate detection at a Read threshold such as 2–3%) requires; using unspliced reference sequences avoids dependence on a prior annotation.

## Related tools

- **VIsoQLR** (Interactive tool whose GMAP panel automates reference index construction from uploaded FASTA reference sequence(s) and then runs GMAP alignment of FASTQ reads) — https://github.com/TBLabFJD/VIsoQLR
- **GMAP** (Spliced aligner used to align long cDNA reads to the unspliced reference index, producing mapped reads (BAM/BAI or GFF3 exon coordinates))
- **Docker** (Container runtime used to run the VIsoQLR image containing the GMAP index-building and alignment panels)
- **Minimap2** (Alternative aligner offered by VIsoQLR when a FASTA reference is used directly instead of a pre-built GMAP index) — https://github.com/lh3/minimap2

## Examples

```
docker pull tblabfjd/visoqlr:latest && docker run -it -p 8888:8888 tblabfjd/visoqlr:latest  # then upload FASTA reference in the GMAP panel; CLI variant: gmap --format=2 -d ref_index reads.fastq > reads.gff3
```

## Evaluation signals

- GMAP panel completes index building automatically after FASTA upload and yields a downloadable reference index ZIP.
- Alignment panel returns two download buttons: mapped reads (BAM) and its index (BAI).
- Mapped BAM contains the expected read counts for the target locus (e.g. 45 mapped reads for PAX6 and 450 for TP53 in the PacBio UHR validation).
- Downstream VIsoQLR analysis with a 3% read threshold reproduces the reference isoform counts (PAX6 canonical isoform 9 reads, 45%; TP53 top isoform 166 reads, 88.8%).

## Limitations

- The article does not report software version numbers for VIsoQLR or the aligners; reproducibility depends on the tblabfjd/visoqlr:latest image tag.
- minimap2 has documented limitations that the VIsoQLR authors say are advised on the minimap2 GitHub.
- Building an index per unspliced reference means novel/aberrant splice junctions must be inferred from reads themselves, as VIsoQLR works without prior knowledge of splice sites.

## Evidence

- [methods] Build GMAP reference index from reference sequence(s): "The first one is used to build the reference index used by GMAP by uploading the reference sequence(s)."
- [methods] Align raw reads with GMAP using the downloaded index: "The second panel is dedicated to align the raw reads (in FASTQ format) using GMAP."
- [methods] Retrieve mapped sequences: "two download bottoms will appear: the mapped reads (BAM) and its index (BAI)."
- [readme] GMAP GFF3 exon coordinates in Mini-IsoQLR: [evidence span withheld — anchor status: external_doc]
- [readme] Benchmark data acquisition and GMAP index generation: [evidence span withheld — anchor status: external_doc]
