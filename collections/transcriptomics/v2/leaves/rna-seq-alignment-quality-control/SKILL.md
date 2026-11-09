---
name: rna-seq-alignment-quality-control
description: Use when raw RNA-seq reads (e.g. the six untreated/LPS-activated monocyte
  samples from GSE118165) must be aligned to GRCh38 and cleaned into coordinate-sorted
  BAMs before InPACT can predict novel IPA sites.
when_to_use_negative:
- Input is already a coordinate-sorted BAM that has passed rRNA/secondary/duplicate
  filtering — go straight to InPACT IPA identification
- Input data are 3'-end sequencing, long-read Iso-seq, or single-cell SMART-seq2 datasets
  that require a different alignment/processing route
- The goal is transcript-level abundance quantification alone — Salmon operates on
  reads or an unsorted transcriptome BAM, not this genomic BAM QC chain
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: InPACT
  role: Downstream consumer of the QC'd coordinate-sorted BAM; its -i input_file argument
    defines the required sorted-BAM contract and default read thresholds (5 spliced
    reads skipped, 10 spanning reads composite)
  repo: https://github.com/YY-TMU/InPACT
- name: Salmon
  role: Subsequent quantification step that consumes gffread-extracted transcripts
    from InPACT-assembled isoforms (salmon index -k 31, salmon quant --validateMappings)
  repo: https://github.com/COMBINE-lab/salmon
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:fbf02e035680fb001b7027869866cef8112ba5b398e60356d8d2a66fe25c0268
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - ClusterProfiler
  - DRIMSeq
  - InPACT
  - Salmon
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# rna-seq-alignment-quality-control

## Summary

Align bulk RNA-seq FASTQ reads to the human GRCh38 reference with HISAT2 and clean the resulting alignments (rRNA removal with Bowtie, secondary-alignment filtering with SAMtools, PCR-duplicate removal with samtools rmdup) to yield coordinate-sorted BAMs suitable for InPACT IPA-site identification and downstream differential usage analysis.

## When to use

Use when raw RNA-seq reads (e.g. the six untreated/LPS-activated monocyte samples from GSE118165) must be aligned to GRCh38 and cleaned into coordinate-sorted BAMs before InPACT can predict novel IPA sites.

## When NOT to use

- Input is already a coordinate-sorted BAM that has passed rRNA/secondary/duplicate filtering — go straight to InPACT IPA identification
- Input data are 3'-end sequencing, long-read Iso-seq, or single-cell SMART-seq2 datasets that require a different alignment/processing route
- The goal is transcript-level abundance quantification alone — Salmon operates on reads or an unsorted transcriptome BAM, not this genomic BAM QC chain

## Inputs

- Raw RNA-seq FASTQ reads (paired-end; e.g. 3 control + 3 LPS-stimulated human monocyte samples from GEO GSE118165)
- Human GRCh38 reference genome (FASTA)
- RefSeq GRCh38 gene annotation (hg38.109.20211119.ncbiRefSeq.gtf.gz)
- rRNA reference sequences for Bowtie-based depletion

## Outputs

- Coordinate-sorted, deduplicated BAM files (primary alignments only) per sample, ready for InPACT (-i sample.bam)
- QC-confirmed alignments that support downstream InPACT_transcript, Salmon quantification and DRIMSeq differential transcript usage

## How to apply

Align raw reads to the human GRCh38 reference genome with HISAT2, then sort the BAM by coordinates, since InPACT's IPA-identification step explicitly requires 'A BAM file that would be used to predict IPA sites and it should be sorted by coordinates'. Remove reads mapped to rRNA sequences using Bowtie, filter out reads flagged as secondary alignment with SAMtools, and remove PCR duplicates with samtools rmdup so that only primary, deduplicated alignments feed the read module. Use the RefSeq GRCh38 annotation (hg38.109.20211119.ncbiRefSeq.gtf.gz from UCSC goldenPath) as the GTF paired with the sorted BAM, and run InPACT with the pre-scored polyA site SAF and default thresholds (minimum 5 spliced reads for skipped terminal exons, 10 spanning reads for composite terminal exons, -P 5 threads). Alignment quality is judged indirectly but decisively by the downstream outputs: the monocyte analysis must reproduce 2977 novel IPA sites (1105 skipped, 1872 composite) and 204 DRIMSeq-significant differential usage events at |log2fold_change| > 1 and FDR < 0.05.

## Related tools

- **InPACT** (Downstream consumer of the QC'd coordinate-sorted BAM; its -i input_file argument defines the required sorted-BAM contract and default read thresholds (5 spliced reads skipped, 10 spanning reads composite)) — https://github.com/YY-TMU/InPACT
- **Salmon** (Subsequent quantification step that consumes gffread-extracted transcripts from InPACT-assembled isoforms (salmon index -k 31, salmon quant --validateMappings)) — https://github.com/COMBINE-lab/salmon

## Examples

```
InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5
```

## Evaluation signals

- Each sample's BAM is coordinate-sorted (verified with samtools quickcheck / sort order), as required by InPACT's -i input_file specification
- No reads flagged as secondary alignment remain (SAMtools filtering) and PCR duplicates are removed via samtools rmdup
- rRNA-mapped reads are depleted from the BAM prior to downstream IPA analysis
- Applying the default thresholds (minimum_spliced_reads_for_skipped=5, minimum_span_reads_for_composite=10) to the six monocyte BAMs reproduces 2977 novel IPA sites: 1105 skipped and 1872 composite events
- The DRIMSeq table built on the aligned data contains 204 significant events at |log2fold_change| > 1 and FDR < 0.05, and the three output tables have the expected structure

## Limitations

- InPACT requires the input BAM sorted by coordinates; unsorted or name-sorted BAMs must be re-sorted before use
- No version number is given for InPACT or any compared tool, so environment reproducibility depends on the provided conda environment.yml
- Read-coverage-based IPA detection in general depends on adequate sequencing depth; poorly aligned or shallow samples may miss novel IPA sites

## Evidence

- [readme] InPACT requires sorted BAM input: "--input_file/-i: A BAM file that would be used to predict IPA sites and it should be sorted by coordinates."
- [readme] InPACT identification command with defaults: "InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5"
- [methods] rRNA depletion step: "We first removed those reads mapped to rRNA sequences using Bowtie"
- [methods] Secondary alignment filtering: "Reads flagged as secondary alignment were filtered out using SAMtools"
- [methods] PCR duplicate removal: "followed by the removal of PCR duplicates using samtools rmdup"
