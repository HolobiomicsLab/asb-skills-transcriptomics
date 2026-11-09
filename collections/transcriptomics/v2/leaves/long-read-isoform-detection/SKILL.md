---
name: long-read-isoform-detection
description: Use when BAM/GFF3/BED6 alignments of long reads (PacBio Iso-Seq or ONT
  cDNA/direct RNA) from a targeted gene, minigene, or SIRV spike-in must be resolved
  into isoforms and quantified without prior splice-site annotation.
when_to_use_negative:
- Input is raw short-read RNA-seq or Sanger/capillary electropherograms — the tool
  is designed for long-read sequencing data
- Whole-transcriptome high-throughput annotation where a batch pipeline such as wf-transcriptomes
  (bambu/SQANTI3) is more appropriate; VIsoQLR is tailored to low-throughput, one-gene-at-a-time
  inspection and manual fine-tuning
- The alignment already exists but Minimap2 was used on data with small exons — re-align
  with GMAP first
edam_operation: http://edamontology.org/operation_0250
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0108
tools:
- name: VIsoQLR
  role: Interactive analyzer/viewer/editor that defines consensus splice sites and
    quantifies isoforms from aligned long reads; manual tuning of read threshold,
    padding, and merge-close-splice-sites parameters
  repo: https://github.com/TBLabFJD/VIsoQLR
- name: GMAP
  role: Aligns FASTQ long reads to the reference (preferred over Minimap2 for small
    exons), producing BAM plus BAI for VIsoQLR upload
- name: StringTie2
  role: Comparator run in long-read mode ('-L -f 0') on the same BAM to benchmark
    isoform detection
- name: FLAIR
  role: Comparator run with default parameters and a supplied GTF for exon-coordinate
    correction
- name: Mini-IsoQLR
  role: Command-line pipeline variant for ONT minigene/RT-PCR data using GMAP --format=2
    GFF3 output
  repo: https://github.com/TBLabFJD/Mini-IsoQLR
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_008
  - task_006
  source_papers:
  - doi: 10.1007/s00439-023-02539-z
    title: 'VIsoQLR: an interactive tool for the detection, quantification and fine-tuning
      of isoforms in selected genes using long-read sequencing'
  source_content_hash: sha256:a3d5e711b77d29663ccaf06fe7f127a3ef9c5c08221aa69384388157340b7a67
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/TBLabFJD/VIsoQLR
  tools:
  - FLAIR
  - GMAP
  - Mini-IsoQLR
  - StringTie2
  - VIsoQLR
  edam_operation: http://edamontology.org/operation_0250
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0108
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# long-read-isoform-detection

## Summary

Detect and quantify isoforms from long-read sequencing (LRS) alignments of a single locus or spike-in set using VIsoQLR's consensus exon coordinate (CEC) approach, benchmarked against StringTie2 and FLAIR on SIRV and PAX6 minigene data.

## When to use

Use when BAM/GFF3/BED6 alignments of long reads (PacBio Iso-Seq or ONT cDNA/direct RNA) from a targeted gene, minigene, or SIRV spike-in must be resolved into isoforms and quantified without prior splice-site annotation.

## When NOT to use

- Input is raw short-read RNA-seq or Sanger/capillary electropherograms — the tool is designed for long-read sequencing data
- Whole-transcriptome high-throughput annotation where a batch pipeline such as wf-transcriptomes (bambu/SQANTI3) is more appropriate; VIsoQLR is tailored to low-throughput, one-gene-at-a-time inspection and manual fine-tuning
- The alignment already exists but Minimap2 was used on data with small exons — re-align with GMAP first

## Inputs

- Aligned long reads in BAM format with BAI index (e.g., GMAP-aligned PacBio UHR Iso-Seq BAM, cond_VIsoQLR_GMAP)
- Unaligned FASTQ long reads plus reference sequence(s) in FASTA for GMAP index building and alignment
- Alignments in GFF3 or BED6 format as alternative to BAM
- Known isoform reference (GTF) for comparison, e.g., Lexogen SIRV reference transcripts with gold-standard abundances

## Outputs

- Detected and quantified isoform table (coordinates, lengths, ranked abundances)
- Exonic starting/ending points export
- Cosine similarity of predicted isoform abundance profile versus reference abundances
- Benchmark detection counts versus known isoform set (e.g., X of 68 SIRV isoforms)

## How to apply

Launch VIsoQLR via Docker (tblabfjd/visoqlr:latest, port 8888) and upload the aligned reads in GFF3, BED6, or BAM format; if starting from FASTQ, first build a GMAP reference index and align with GMAP (preferred over Minimap2, which has documented problems with small exons). VIsoQLR defines consensus splice sites per gene and quantifies isoforms, with the 'Read threshold (%)' (default 2%) selecting candidate CECs by exon-coordinate frequency among all reads, plus 'Padding (# of bases)' and 'Merge close splice sites (# of bases)' parameters for merging close candidates. Import known transcripts (e.g., Lexogen SIRV reference GTF) via the 'Defined isoforms for comparison' panel for visual comparison, and export the isoform information panel and exonic starting/ending points. For benchmarking, run FLAIR with default parameters plus a supplied GTF and StringTie2 in long-read mode with '-L -f 0' on the same BAM, then count exact isoform matches against the known set and compute cosine similarity of abundance profiles versus the gold standard.

## Related tools

- **VIsoQLR** (Interactive analyzer/viewer/editor that defines consensus splice sites and quantifies isoforms from aligned long reads; manual tuning of read threshold, padding, and merge-close-splice-sites parameters) — https://github.com/TBLabFJD/VIsoQLR
- **GMAP** (Aligns FASTQ long reads to the reference (preferred over Minimap2 for small exons), producing BAM plus BAI for VIsoQLR upload)
- **StringTie2** (Comparator run in long-read mode ('-L -f 0') on the same BAM to benchmark isoform detection)
- **FLAIR** (Comparator run with default parameters and a supplied GTF for exon-coordinate correction)
- **Mini-IsoQLR** (Command-line pipeline variant for ONT minigene/RT-PCR data using GMAP --format=2 GFF3 output) — https://github.com/TBLabFJD/Mini-IsoQLR

## Examples

```
docker pull tblabfjd/visoqlr:latest && docker run -it -p 8888:8888 tblabfjd/visoqlr:latest
```

## Evaluation signals

- SIRV benchmark: 49 of 68 SIRV isoforms (72%) detected with default parameters, 99% exon coordinate accuracy, and overall cosine similarity 0.78 to Lexogen gold standard (highest partial similarity SIRV7 = 0.97); FLAIR expect 37/68 (cosine 0.68), StringTie2 12/68 (cosine 0.42)
- PAX6 minigene check: VIsoQLR yields 40 isoforms (9 above 1% abundance) with canonical isoform at 53.1% and 776 bp; StringTie2 yields 58 isoforms (8 above 1%) with canonical at 60.5% and 816 bp
- Every isoform reported should be checked against the 2% default read threshold; most automatically missed isoforms fall below it, and lowering the threshold trades sensitivity for more calls
- Boundary-extension audit: StringTie2 boundary extensions (e.g., 816 vs 776 bp canonical) must be flagged as artifacts by cross-checking they are absent from the shared BAM alignments
- BAM/BAI pair integrity: GMAP panel must return both mapped reads (BAM) and its index (BAI) before upload

## Limitations

- Missing isoforms in the automatic analysis were mostly below the default 2% read threshold; lowering the threshold detects more isoforms but risks noise, and per-gene manual parameter tuning is often required
- No software version numbers reported for VIsoQLR, StringTie2, or FLAIR; no data accession identifiers given for the underlying LRS datasets
- Minimap2 has documented problems with small exons, so GMAP should be used for alignment
- Small isoform abundances (e.g., Iso5 at 4% and Iso8 at 1.7%) may be overrepresented by LRS and hard to distinguish from noise in electropherograms
- StringTie2 extends isoform outer boundaries and may not call isoforms lacking certain exons (e.g., Iso4 and Iso9 with missed exons 5 and 6)

## Evidence

- [discussion] VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs: "VIsoQLR detects with 99% accuracy of exon coordinates 72% of the known isoforms using SIRVs, outperforming the other tools."
- [discussion] Read threshold default 2%: "most missing isoforms were below 2% (default value). At the same time, lowering this threshold has the cost of detecting more isoforms"
- [methods] Input formats: "navigate to your file system to upload your aligner reads in `GFF3`, `BED6`, or `BAM` file format."
- [methods] Alignment choice: [evidence span withheld — anchor status: not_found]
- [discussion] StringTie2 artifact: "This extension (also seen in the SIRVs analysis) seems to be an artifact as it is not present in BAM files used by StringTie2 and VIsoQLR"
- [readme] Tool scope: "VIsoQLR is tailored to thoroughly analyze mRNA expression and maturation in low-throughput splicing assays."
