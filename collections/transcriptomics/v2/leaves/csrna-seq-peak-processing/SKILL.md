---
name: csrna-seq-peak-processing
description: Use when raw csRNA-seq reads (e.g., GEO datasets such as GSE221852 or
  GSE135498) must be turned into called TSS peaks and ±100 bp promoter sequences for
  downstream core promoter element detection.
when_to_use_negative:
- Input is already a FASTA of promoter sequences — skip peak calling and run the ElemeNT
  2023 command-line tool directly on the sequences
- Data are not TSS-enriched nascent transcripts (e.g., ordinary RNA-seq without 5'
  capture), since positional motif enrichment around the TSS is not interpretable
  without TSS resolution
- The goal is de novo motif discovery rather than detecting known core promoter elements
  with ElemeNT 2023's predefined PWMs
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0092
tools:
- name: HOMER
  role: findPeaks is used to call high-confidence TSS peaks from csRNA-seq data for
    each sample
- name: bedtools
  role: getfasta extracts ±100 bp core promoter sequences centered on each called
    TSS into FASTA files
- name: ElemeNT 2023
  role: Perl command-line tool run on the extracted FASTA files to detect TATA, dInr,
    PB, DPE, GAGA, Motif 1 and dTCT elements
  repo: https://github.com/OritAdato/ElemeNT
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae110
    title: 'ElemeNT 2023: an enhanced tool for detection and curation of core promoter
      elements'
  source_content_hash: sha256:7fd9c36babab49850b2c914b7d0cb5248c6fc3b82428410bad71493575372dab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/OritAdato/ElemeNT
  tools:
  - ElemeNT 2023
  - HOMER
  - Perl
  - bedtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# csrna-seq-peak-processing

## Summary

Converts csRNA-seq data into TSS peak coordinates (HOMER findPeaks) and TSS-centered ±100 bp core promoter FASTA sequences, ready for ElemeNT 2023 core promoter element detection and positional motif-quality assessment.

## When to use

Use when raw csRNA-seq reads (e.g., GEO datasets such as GSE221852 or GSE135498) must be turned into called TSS peaks and ±100 bp promoter sequences for downstream core promoter element detection.

## When NOT to use

- Input is already a FASTA of promoter sequences — skip peak calling and run the ElemeNT 2023 command-line tool directly on the sequences
- Data are not TSS-enriched nascent transcripts (e.g., ordinary RNA-seq without 5' capture), since positional motif enrichment around the TSS is not interpretable without TSS resolution
- The goal is de novo motif discovery rather than detecting known core promoter elements with ElemeNT 2023's predefined PWMs

## Inputs

- csRNA-seq FASTQ/SRA data from GEO (e.g., GSE221852 D. melanogaster 0–8 h embryos; GSE135498 human K562 and mouse BMDM)
- Reference genome assembly FASTA (e.g., D. melanogaster)
- TSS peak coordinate BED files produced by HOMER findPeaks

## Outputs

- TSS peak coordinate table (BED) per sample
- FASTA of ±100 bp sequences centered on each annotated TSS
- Per-element position, fraction, and median PWM score distributions (element_distributions.csv)
- Catalog of detected core promoter elements (TATA, dInr, PB, DPE, GAGA, Motif 1, dTCT) per transcript

## How to apply

Download the csRNA-seq dataset from GEO/SRA (e.g., GSE221852 for the first 8 h of D. melanogaster embryonic development, or GSE135498 for human K562 cells and mouse bone marrow-derived macrophages) and call TSS peaks with HOMER findPeaks to obtain high-confidence TSS coordinates per sample. Extract ±100 bp genomic sequences centered on each called TSS into FASTA files with bedtools getfasta from the appropriate genome assembly. Run the ElemeNT 2023 command-line implementation (Perl, github.com/OritAdato/ElemeNT) on each FASTA to detect TATA, dInr, PB, DPE, GAGA, Motif 1 and dTCT elements. Aggregate detections into per-element position, fraction, and median PWM score distributions relative to the TSS (e.g., tabulated as element_distributions.csv per species). The rationale is that preferential motif positioning — e.g., dInr at −2 and DPE at +28 — is only meaningful if peaks are TSS-centered and sequences are symmetric around the TSS.

## Related tools

- **HOMER** (findPeaks is used to call high-confidence TSS peaks from csRNA-seq data for each sample)
- **bedtools** (getfasta extracts ±100 bp core promoter sequences centered on each called TSS into FASTA files)
- **ElemeNT 2023** (Perl command-line tool run on the extracted FASTA files to detect TATA, dInr, PB, DPE, GAGA, Motif 1 and dTCT elements) — https://github.com/OritAdato/ElemeNT

## Examples

```
bedtools getfasta -fi dmel_genome.fa -bed TSS_peaks.bed -fo promoters.fasta
```

## Evaluation signals

- In D. melanogaster embryo csRNA-seq, the majority of identified dInr and DPE motifs fall at exactly −2 and +28, respectively, relative to the TSS
- For each element, the positional frequency peak and the median PWM score maximum coincide with the reported positions: dInr −2, DPE +28, Motif 1 −5, dTCT −2, PB +26
- Per-species element distributions (human K562, mouse BMDM) show preferential positional enrichment around the TSS rather than a flat positional profile
- Output is tabulated as element_distributions.csv with position, fraction, and median-score distributions per element per species

## Limitations

- Valid positional enrichment is expected only for TSS-centered, nascent-transcript data; peak-calling quality from HOMER directly determines motif positioning quality
- Element calls are PWM-based predictions from ElemeNT 2023, not experimentally validated binding sites
- ElemeNT 2023 is focused on core promoter elements, though it can also search any user-defined motif including sequence-specific DNA binding sites

## Evidence

- [intro] ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences.: "ElemeNT is an interactive tool (implemented in Perl) for rapid and convenient detection of core promoter elements within any given sequences."
- [abstract] for readily evaluating the quality of high-throughput transcription start site (TSS) datasets: "for readily evaluating the quality of high-throughput transcription start site (TSS) datasets"
- [abstract] ElemeNT's CORE database, which contains predicted core promoter elements around annotated TSSs, is now expanded to cover 10 species: "ElemeNT's CORE database, which contains predicted core promoter elements around annotated TSSs, is now expanded to cover 10 species"
- [readme] the source code used for the analysis included in the manuscript: "ElemeNT/ElemeNT2023/src/Manuscript - the source code used for the analysis included in the manuscript"
