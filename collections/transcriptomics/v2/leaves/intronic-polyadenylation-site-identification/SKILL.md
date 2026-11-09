---
name: intronic-polyadenylation-site-identification
description: Use when analyzing coordinate-sorted RNA-seq BAM files (bulk or SMART-seq2
  single-cell) to discover novel skipped or composite IPA events, reconstruct IPA
  isoforms, and estimate per-site PAU usage not present in the RefSeq annotation.
when_to_use_negative:
- The goal is only 3'UTR alternative polyadenylation quantification in annotated regions
  — tools like DaPars, Aptardi, APAtrap, TAPAS, QAPA, or LABRAT cover that scope;
  InPACT targets intronic polyadenylation and novel IPA isoform discovery
- Only 3'-end sequencing (3P-seq, polyA-seq) data are available and no conventional
  RNA-seq alignments exist, since InPACT is designed for standard RNA-seq BAM inputs
- Sequencing depth is very low — the read thresholds (>=5 spliced reads for skipped,
  >=10 spanning reads for composite) may be unattainable, mirroring the coverage-dependence
  criticism raised against IPAFinder
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3512
tools:
- name: InPACT
  role: 'Core method: identifies skipped and composite IPA sites from BAM + RefSeq
    GTF + candidate SAF, assembles novel IPA transcripts (InPACT_transcript), and
    computes PAU usage (InPACT_quantify)'
  repo: https://github.com/YY-TMU/InPACT
- name: Salmon
  role: Transcript-level abundance quantification (salmon index -k 31; salmon quant
    --validateMappings) on gffread-extracted novel IPA transcripts, feeding quant.sf
    into InPACT_quantify
  repo: https://github.com/COMBINE-lab/salmon
- name: gffread
  role: Extracts transcript sequences from merged.gtf to build the Salmon index FASTA
- name: HISAT2
  role: Aligns raw RNA-seq reads to the GRCh38 reference genome to produce the sorted
    BAM input required by InPACT
- name: SAMtools
  role: Filters secondary alignments and removes PCR duplicates (samtools rmdup) prior
    to IPA site identification
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_008
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:468f86d17ce0e44366f0bf096cc716417dcf302060250c5e1ba613832a35ebc5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - APARENT
  - ClusterProfiler
  - DRIMSeq
  - DeepPASS
  - DeepPASTA
  - HISAT2
  - InPACT
  - SAMtools
  - Salmon
  - gffread
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# intronic-polyadenylation-site-identification

## Summary

Identify and quantify novel intronic polyadenylation (IPA) sites and isoforms from conventional RNA-seq using InPACT, which combines a CNN sequence module scoring candidate polyA sites (InPACT_polyAsites.hg38.saf for GRCh38) with a random-forest read module operating on sorted BAM alignments against a RefSeq GTF annotation.

## When to use

Use when analyzing coordinate-sorted RNA-seq BAM files (bulk or SMART-seq2 single-cell) to discover novel skipped or composite IPA events, reconstruct IPA isoforms, and estimate per-site PAU usage not present in the RefSeq annotation.

## When NOT to use

- The goal is only 3'UTR alternative polyadenylation quantification in annotated regions — tools like DaPars, Aptardi, APAtrap, TAPAS, QAPA, or LABRAT cover that scope; InPACT targets intronic polyadenylation and novel IPA isoform discovery
- Only 3'-end sequencing (3P-seq, polyA-seq) data are available and no conventional RNA-seq alignments exist, since InPACT is designed for standard RNA-seq BAM inputs
- Sequencing depth is very low — the read thresholds (>=5 spliced reads for skipped, >=10 spanning reads for composite) may be unattainable, mirroring the coverage-dependence criticism raised against IPAFinder

## Inputs

- Coordinate-sorted BAM file of RNA-seq alignments to GRCh38 (bulk or per-cell scRNA-seq)
- RefSeq GTF annotation (e.g. hg38.109.20211119.ncbiRefSeq.gtf.gz from UCSC goldenPath)
- Pre-scored candidate polyA site file (InPACT_polyAsites.hg38.saf: gene, chromosome, start, end, strand)
- Reference genome FASTA (for InPACT_transcript and gffread)

## Outputs

- predict.result.txt — identified IPA sites with terminal exon, IPA type (Skipped/Composite), gene, coordinates, and PolyAsite columns
- merged.gtf — augmented GTF of annotated plus novel IPA isoforms
- ipa_usage.txt — per-IPA-site relative usage (PAU) estimates derived from Salmon transcript TPMs

## How to apply

Align reads to GRCh38 with HISAT2, filter secondary alignments and PCR duplicates with SAMtools, and coordinate-sort the BAM. Run InPACT (-i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5) using default read thresholds — minimum 5 spliced reads for skipped terminal exons and 10 boundary-crossing reads for composite terminal exons — to produce predict.result.txt per sample. Assemble novel IPA isoforms with InPACT_transcript into a merged GTF (merged.gtf), extract transcript sequences with gffread, build a Salmon index with -k 31, and quantify with salmon quant --validateMappings to obtain quant.sf. Finally compute per-isoform IPA usage (PAU) with InPACT_quantify (--transcript_tpm quant.sf --annotation_file merged.gtf --ipa_info predict.result.txt). The rationale is that the sequence module provides pre-scored candidate polyA sites while the read module uses RNA-seq alignment features (via random-forest ensembles) to call IPA sites sample-wise, enabling identification of both skipped and composite IPA, which competing tools such as APAIQ and IPAFinder cannot do reliably.

## Related tools

- **InPACT** (Core method: identifies skipped and composite IPA sites from BAM + RefSeq GTF + candidate SAF, assembles novel IPA transcripts (InPACT_transcript), and computes PAU usage (InPACT_quantify)) — https://github.com/YY-TMU/InPACT
- **Salmon** (Transcript-level abundance quantification (salmon index -k 31; salmon quant --validateMappings) on gffread-extracted novel IPA transcripts, feeding quant.sf into InPACT_quantify) — https://github.com/COMBINE-lab/salmon
- **gffread** (Extracts transcript sequences from merged.gtf to build the Salmon index FASTA)
- **HISAT2** (Aligns raw RNA-seq reads to the GRCh38 reference genome to produce the sorted BAM input required by InPACT)
- **SAMtools** (Filters secondary alignments and removes PCR duplicates (samtools rmdup) prior to IPA site identification)

## Examples

```
InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5; InPACT_transcript --predict_terminal predict.result.txt --annotated_gtf RefSeq.gtf --fa_path genome.fa --save_gtf merged.gtf
```

## Evaluation signals

- Output schema check: predict.result.txt must contain columns for Terminal exon, IPA type (Skipped or Composite), Gene, upstream coordinate, PolyAsite, and IPA usage (PAU)
- Reproducibility benchmark on HEK293 replicates: expect 471 and 393 novel IPA sites, ~218 overlapping sites between replicates, and Pearson r = 0.94 (P < 0.05) for novel IPA isoform expression
- Known counts check: HeLa U1-AMO treatment should yield ~767 novel IPA events vs 151 in control (Wilcoxon P < 2.2e-16); monocyte data should give 2977 novel sites (1105 skipped, 1872 composite)
- Threshold compliance: called skipped events must have >=5 spliced reads and composite events >=10 boundary-crossing reads (defaults confirmed in the README)
- InPACT takes about an hour to run the provided test file (SRR7647801) using five cores — substantially faster or failed runs indicate misconfiguration

## Limitations

- No version number is reported for InPACT or any compared tool, so exact software versions cannot be pinned from the paper
- IPAFinder-style read-coverage-fluctuation approaches need adequate sequencing depth; low-depth inputs (e.g. shallow scRNA-seq cells) may fall below the 5/10-read thresholds and miss events
- The human fetal bone marrow scRNA-seq dataset lacked a stated accession in the paper (task cards resolve it to E-MTAB-9801), complicating retrieval without external lookup
- InPACT's sequence-module candidate file is genome-build specific (hg38 SAF provided); other genomes require retraining or regenerating candidate polyA sites

## Evidence

- [abstract] we propose a computational method called InPACT, which allows for the precise characterization of IPA from conventional RNA-seq data: "we propose a computational method called InPACT, which allows for the precise characterization of IPA from conventional RNA-seq data"
- [methods] InPACT invocation with defaults: "InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5"
- [methods] Random-forest read module: "InPACT trains classifiers on ten randomized subsamples of the training set using random forest, and then ensemble the classifiers."
- [readme] Default read thresholds: "Minimum number of spliced reads required to consider the potential skipped terminal exon. [default=5]"
- [readme] Quantify + usage command chain: "InPACT_quantify --transcript_tpm quant.sf --annotation_file merged.gtf --ipa_info predict.result.txt --save_file ipa_usage.txt"
