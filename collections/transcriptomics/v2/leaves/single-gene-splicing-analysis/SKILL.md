---
name: single-gene-splicing-analysis
description: Use when you need to detect, inspect and quantify known or novel isoforms
  for one gene at a time from long-read RNA-seq data (PacBio or Nanopore BAM), without
  prior knowledge of splice sites, and reproduce reported isoform counts.
when_to_use_negative:
- The goal is whole-transcriptome de novo discovery across many samples rather than
  close inspection of one gene at a time — wf-transcriptomes (bambu/SQANTI3) is designed
  for that reference-guided multi-sample workflow.
- Input reads are short-read sequencing data or you already have an annotation with
  exon coordinates and splice sites, which VIsoQLR is not designed to require or exploit.
- Only abundant isoforms matter and manual curation is unnecessary — the default 2%
  threshold may suffice and lowering the threshold trades sensitivity for more (possibly
  noisy) isoforms.
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: VIsoQLR
  role: Interactive analyzer, viewer and editor that identifies consensus splice sites
    and quantifies isoforms from mapped long reads; run in Docker with a Read threshold
    (%) parameter (3% here)
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: GMAP
  role: Aligner used inside VIsoQLR to map raw FASTQ long reads against the built
    reference index, producing BAM/BAI output
- name: Docker
  role: Container runtime used to pull and serve the VIsoQLR image (docker pull tblabfjd/visoqlr:latest;
    docker run -it -p 8888:8888)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:0b416cf95aedde7ce3e30a7bba2c531c7fdf5aeeb2bee0147f6179aff64ae7d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - Docker
  - GMAP
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-gene-splicing-analysis

## Summary

Reproduce per-gene isoform detection and quantification with VIsoQLR on long-read sequencing data by mapping reads with GMAP, defining consensus exon coordinates (CECs) at a chosen read threshold (e.g. 3%), and extracting isoform read counts/abundances for single loci such as PAX6 and TP53.

## When to use

Use when you need to detect, inspect and quantify known or novel isoforms for one gene at a time from long-read RNA-seq data (PacBio or Nanopore BAM), without prior knowledge of splice sites, and reproduce reported isoform counts.

## When NOT to use

- The goal is whole-transcriptome de novo discovery across many samples rather than close inspection of one gene at a time — wf-transcriptomes (bambu/SQANTI3) is designed for that reference-guided multi-sample workflow.
- Input reads are short-read sequencing data or you already have an annotation with exon coordinates and splice sites, which VIsoQLR is not designed to require or exploit.
- Only abundant isoforms matter and manual curation is unnecessary — the default 2% threshold may suffice and lowering the threshold trades sensitivity for more (possibly noisy) isoforms.

## Inputs

- PacBio UHR long-read sequencing data in FASTQ format
- MANE (Ensembl 108) unspliced reference sequences in FASTA format
- GMAP reference index ZIP (built inside VIsoQLR from the FASTA)
- Mapped reads as BAM file with BAI index

## Outputs

- GMAP reference index ZIP
- Mapped reads as BAM with BAI index
- Per-gene isoform detection and quantification (consensus splice sites, isoform read counts and percentage abundances) for PAX6 and TP53
- Combined per-gene isoform count and abundance table (visoqlr_uhr_pax6_tp53_isoforms.tsv)

## How to apply

Obtain the public PacBio UHR long-read FASTQ and the MANE (Ensembl 108) unspliced reference sequences in FASTA. In the VIsoQLR Docker container (tblabfjd/visoqlr:latest, run with port 8888), upload the reference FASTA to the GMAP panel to build the GMAP reference index automatically. Upload the index ZIP together with the FASTQ reads in the alignment panel, select the thread count, run GMAP, and download the mapped reads as BAM plus its BAI index. Upload the BAM to the main VIsoQLR view, set the Read threshold (%) parameter — 3% for this analysis — which selects candidate CECs by the proportion of exon coordinates among all reads, and inspect the PAX6 and TP53 loci in the isoform information panel. Extract per-isoform read counts and abundances, validating against the reference values: PAX6 45 mapped reads with the canonical isoform at 9 reads (45%), TP53 450 mapped reads with the top isoform at 166 reads (88.8%); save the table as visoqlr_uhr_pax6_tp53_isoforms.tsv. Rationale: VIsoQLR was designed for close, per-gene inspection and manual fine-tuning of parameters, since low-abundance isoforms are often missed by the default 2% threshold.

## Related tools

- **VIsoQLR** (Interactive analyzer, viewer and editor that identifies consensus splice sites and quantifies isoforms from mapped long reads; run in Docker with a Read threshold (%) parameter (3% here)) — https://github.com/TBLabFJD/VIsoQLR
- **GMAP** (Aligner used inside VIsoQLR to map raw FASTQ long reads against the built reference index, producing BAM/BAI output)
- **Docker** (Container runtime used to pull and serve the VIsoQLR image (docker pull tblabfjd/visoqlr:latest; docker run -it -p 8888:8888))

## Examples

```
docker pull tblabfjd/visoqlr:latest; docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- PAX6 locus yields 45 mapped reads with the canonical transcript as the most abundant isoform at 9 reads (45%).
- TP53 locus yields 450 mapped reads with the most abundant isoform at 166 reads (88.8%).
- Mapped read totals and top-isoform counts/abundances for both genes match the reported reference values exactly.
- The GMAP panel returns both a BAM file and its BAI index after alignment, confirming mapping completed.
- The combined per-gene table is saved as visoqlr_uhr_pax6_tp53_isoforms.tsv with read counts and percentage abundances per isoform.

## Limitations

- Missing isoforms in the automatic analysis are often below the default 2% read threshold; lowering the threshold detects more isoforms but individual genes may still require close inspection and manual configuration of parameters.
- Small isoform abundances (e.g. 4% and 1.7%) may be overrepresented by long-read sequencing and hard to distinguish from noise in electropherograms.
- Known isoform comparisons require importing references from other methods; VIsoQLR does not use prior exon coordinate annotations by default.

## Evidence

- [methods] The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads.: "The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads."
- [methods] The second panel is dedicated to align the raw reads (in FASTQ format) using GMAP.: "The second panel is dedicated to align the raw reads (in FASTQ format) using GMAP."
- [methods] docker run -it -p 8888:8888 tblabfjd/visoqlr:latest: "docker run -it -p 8888:8888 tblabfjd/visoqlr:latest"
- [intro] To the best of our knowledge, there is no interactive tool that allows a close inspection and fine-tuning analysis of one gene at a time on data from long-reads RNA sequencing.: "To the best of our knowledge, there is no interactive tool that allows a close inspection and fine-tuning analysis of one gene at a time on data from long-reads RNA sequencing."
- [discussion] the description of the isoforms collection of individual genes seems to require a close inspection and sometimes a manual configuration of certain parameters: "the description of the isoforms collection of individual genes seems to require a close inspection and sometimes a manual configuration of certain parameters"
