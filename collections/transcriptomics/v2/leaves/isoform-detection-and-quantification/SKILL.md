---
name: isoform-detection-and-quantification
description: Use when low-throughput long-read (PacBio/ONT) data mapped to a single
  gene or locus needs isoform detection and quantification without prior splice-site
  knowledge, e.g. PAX6/TP53 PacBio UHR analysis at a 3% read threshold.
when_to_use_negative:
- When the goal is genome-wide, multi-sample transcript discovery/differential expression
  rather than interactive, one-gene-at-a-time analysis — wf-transcriptomes (bambu/SQANTI3)
  or StringTie2/FLAIR/TALON pipelines are the appropriate batch tools
- When data are short reads from Sanger/capillary electropherograms or short-read
  RNA-seq — VIsoQLR is designed for long-read sequencing (LRS) data only
- When an existing genome annotation with exon coordinates should drive analysis directly,
  since VIsoQLR defines splice sites de novo from reads without prior splice-site
  knowledge
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: VIsoQLR
  role: Interactive analyzer/viewer/editor that defines consensus splice sites, detects
    and quantifies isoforms, and allows manual fine-tuning of CEC parameters
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: GMAP
  role: Aligner used within VIsoQLR to map raw FASTQ reads to the unspliced FASTA
    reference index, producing BAM+BAI
- name: Docker
  role: Containerization/runtime for VIsoQLR (docker pull tblabfjd/visoqlr:latest;
    docker run -it -p 8888:8888)
- name: Mini-IsoQLR
  role: Command-line pipeline alternative for isoform quantification of minigene ONT
    data using GMAP --format=2 GFF3 output
  repo: https://github.com/TBLabFJD/Mini-IsoQLR
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:da873eec94fbfcc781c470990ad4f3b8f4a3e1f8c8910216e6c5646a3630fbb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - Docker
  - GMAP
  - Mini-IsoQLR
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# isoform-detection-and-quantification

## Summary

Identify and quantify known and novel isoforms in selected genes from long-read sequencing data using VIsoQLR, which defines consensus splice sites/consensus exon coordinates (CECs) per gene and quantifies isoforms with interactive fine-tuning (e.g. Read threshold set to 3%).

## When to use

Use when low-throughput long-read (PacBio/ONT) data mapped to a single gene or locus needs isoform detection and quantification without prior splice-site knowledge, e.g. PAX6/TP53 PacBio UHR analysis at a 3% read threshold.

## When NOT to use

- When the goal is genome-wide, multi-sample transcript discovery/differential expression rather than interactive, one-gene-at-a-time analysis — wf-transcriptomes (bambu/SQANTI3) or StringTie2/FLAIR/TALON pipelines are the appropriate batch tools
- When data are short reads from Sanger/capillary electropherograms or short-read RNA-seq — VIsoQLR is designed for long-read sequencing (LRS) data only
- When an existing genome annotation with exon coordinates should drive analysis directly, since VIsoQLR defines splice sites de novo from reads without prior splice-site knowledge

## Inputs

- Raw long-read sequences in FASTQ format (e.g. PacBio UHR long-read data)
- Unspliced reference sequences in FASTA format (e.g. MANE Ensembl 108) for GMAP index building
- Aligned reads in BAM (with BAI), GFF3, or BED6 format
- Optionally, known isoforms/references for comparison

## Outputs

- Mapped reads as BAM plus BAI index from GMAP alignment
- Per-gene isoform detection and quantification: isoform read counts and abundance percentages (e.g. PAX6: 45 mapped reads, canonical isoform 9 reads/45%; TP53: 450 mapped reads, top isoform 166 reads/88.8%)
- Combined per-gene isoform count/abundance table (e.g. visoqlr_uhr_pax6_tp53_isoforms.tsv)

## How to apply

Pull and run the VIsoQLR Docker image (tblabfjd/visoqlr:latest, port 8888). In the GMAP panel, upload the reference sequence(s) in FASTA format (e.g. MANE Ensembl 108 unspliced sequences for PAX6 and TP53) to auto-build the GMAP reference index, then in the second panel upload that index ZIP together with the raw FASTQ reads, choose thread count, run GMAP, and download the mapped reads as BAM plus its BAI index. Upload the aligned BAM (GFF3 or BED6 also accepted) to the main analysis view, which automatically defines consensus splice sites and candidate CECs and quantifies isoforms per gene. Set the `Read threshold (%)` parameter (default 2%; 3% in the PAX6/TP53 PacBio UHR task) to select candidate CECs by exon-coordinate frequency among all reads, adjusting `Padding (# of bases)` and `Merge close splice sites (# of bases)` as needed for manual curation. Extract per-gene mapped-read totals and isoform read counts/abundances from the isoform information panel, optionally importing known isoforms as references for comparison.

## Related tools

- **VIsoQLR** (Interactive analyzer/viewer/editor that defines consensus splice sites, detects and quantifies isoforms, and allows manual fine-tuning of CEC parameters) — https://github.com/TBLabFJD/VIsoQLR
- **GMAP** (Aligner used within VIsoQLR to map raw FASTQ reads to the unspliced FASTA reference index, producing BAM+BAI)
- **Docker** (Containerization/runtime for VIsoQLR (docker pull tblabfjd/visoqlr:latest; docker run -it -p 8888:8888))
- **Mini-IsoQLR** (Command-line pipeline alternative for isoform quantification of minigene ONT data using GMAP --format=2 GFF3 output) — https://github.com/TBLabFJD/Mini-IsoQLR

## Examples

```
docker pull tblabfjd/visoqlr:latest; docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- Mapped read totals per gene match reference values (PAX6: 45 mapped reads; TP53: 450 mapped reads on PacBio UHR at 3% threshold)
- Top-isoform counts and abundances match reported values (PAX6 canonical: 9 reads/45%; TP53 top isoform: 166 reads/88.8%)
- BAM output is accompanied by a valid BAI index file from GMAP mapping
- Changing the Read threshold (%) (e.g. default 2% vs 3%) recovers expected additional low-abundance isoforms without excessive noise calling
- Detected isoform set is consistent with imported known reference isoforms where available

## Limitations

- Most missing isoforms fall below the default 2% read threshold; lowering the threshold detects more isoforms at the cost of noise, so single-gene collections may need close inspection and manual parameter configuration
- Small isoform abundances (e.g. 4% and 1.7%) may be overrepresented by long-read sequencing and hard to distinguish from noise in electropherograms
- Minimap2-based alignment has limitations for this use case (advised on its GitHub); GMAP is the recommended aligner inside VIsoQLR
- VIsoQLR is tailored to low-throughput splicing assays and one-gene-at-a-time inspection, not high-throughput transcriptome-wide discovery

## Evidence

- [abstract] Here we describe VIsoQLR, an interactive analyzer, viewer and editor for the semi-automated identification and quantification of known and novel isoforms using long-read sequencing data.: "Here we describe VIsoQLR, an interactive analyzer, viewer and editor for the semi-automated identification and quantification of known and novel isoforms using long-read sequencing data."
- [abstract] Our tool takes sequences aligned to a reference, and for each gene, it defines consensus splice sites and quantifies isoforms.: "Our tool takes sequences aligned to a reference, and for each gene, it defines consensus splice sites and quantifies isoforms."
- [methods] The first one is used to build the reference index used by GMAP by uploading the reference sequence(s).: "The first one is used to build the reference index used by GMAP by uploading the reference sequence(s)."
- [methods] The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads.: "The `Read threshold (%)` is used to select candidate CECs above this value based on the proportion exon coordinates among all reads."
- [discussion] most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms: "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
- [readme] This pipeline uses GMAP aligner, which aligns cDNA sequences to a genome, using the parameter `--format=2` which generates a GFF3 file: [evidence span withheld — anchor status: not_found]
