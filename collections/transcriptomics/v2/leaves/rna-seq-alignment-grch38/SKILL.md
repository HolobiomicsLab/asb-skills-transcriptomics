---
name: rna-seq-alignment-grch38
description: Use when the input is human (GRCh38/hg38) RNA-seq FASTQ reads that must
  be aligned to a RefSeq-annotated genome before IPA site prediction with InPACT or
  downstream transcript quantification with Salmon.
when_to_use_negative:
- Input data is already aligned (BAM/CRAM) — skip directly to InPACT with the sorted
  BAM rather than re-aligning
- The genome/organism is not human GRCh38 — the provided InPACT_polyAsites.hg38.saf
  candidate file and RefSeq hg38 GTF are hg38-specific
- The goal is 3'UTR-level alternative polyadenylation quantification only, where tools
  like DaPars, QAPA, or LABRAT on existing alignments suffice
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3320
tools:
- name: InPACT
  role: Downstream IPA site identification and quantification tool consuming the coordinate-sorted
    BAM; also assembles novel IPA isoforms (InPACT_transcript) and computes IPA usage
    (InPACT_quantify)
  repo: https://github.com/YY-TMU/InPACT
- name: HISAT2
  role: Aligns RNA-seq reads to the GRCh38 reference genome index to produce the BAM
    input required by InPACT
- name: SAMtools
  role: Sorts BAMs by coordinate, filters secondary-alignment flags, and removes PCR
    duplicates (rmdup) before InPACT
- name: Salmon
  role: Indexes (k-mer size 31) and quantifies transcript abundances from gffread-extracted
    novel IPA transcript FASTAs; quant.sf feeds InPACT_quantify
  repo: https://github.com/COMBINE-lab/salmon
- name: gffread
  role: Extracts transcript sequences from the merged.gtf of novel IPA isoforms for
    Salmon indexing
- name: Bowtie
  role: Removes reads mapped to rRNA sequences prior to IPA prediction
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:17d57007b47e0e72fa9c7967d069609d20561f65a03fd103448b08f04ded3548
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YY-TMU/InPACT
  tools:
  - Bowtie
  - HISAT2
  - InPACT
  - SAMtools
  - Salmon
  - gffread
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# rna-seq-alignment-grch38

## Summary

Preprocessing skill for aligning RNA-seq reads to the human GRCh38 reference genome with HISAT2 and producing coordinate-sorted, filtered BAMs suitable as input for InPACT-based intronic polyadenylation (IPA) identification.

## When to use

Use when the input is human (GRCh38/hg38) RNA-seq FASTQ reads that must be aligned to a RefSeq-annotated genome before IPA site prediction with InPACT or downstream transcript quantification with Salmon.

## When NOT to use

- Input data is already aligned (BAM/CRAM) — skip directly to InPACT with the sorted BAM rather than re-aligning
- The genome/organism is not human GRCh38 — the provided InPACT_polyAsites.hg38.saf candidate file and RefSeq hg38 GTF are hg38-specific
- The goal is 3'UTR-level alternative polyadenylation quantification only, where tools like DaPars, QAPA, or LABRAT on existing alignments suffice

## Inputs

- Human RNA-seq FASTQ read files (paired-end or single-end) from conditions such as HeLa U1-AMO/control or HEK293 replicates
- GRCh38 (hg38) reference genome FASTA
- RefSeq gene annotation GTF (hg38.109.20211119.ncbiRefSeq.gtf, UCSC goldenPath)
- InPACT_polyAsites.hg38.saf candidate polyA-site file (20 nt regions upstream of potential polyA sites)

## Outputs

- Coordinate-sorted BAM files per sample with secondary alignments, rRNA-mapped reads, and PCR duplicates removed
- InPACT predict.result.txt per sample listing predicted IPA sites (terminal exon, skipped/composite type, gene, coordinates, PolyAsite)
- merged.gtf of assembled novel IPA isoforms (via InPACT_transcript)
- ipa_usage.txt with per-isoform PAU (IPA usage) estimates from InPACT_quantify

## How to apply

Build or use an existing HISAT2 genome index for GRCh38 and align the RNA-seq reads (paired-end or single-end) to the reference. Post-process with SAMtools: remove reads flagged as secondary alignments, remove rRNA-mapped reads with Bowtie and PCR duplicates with samtools rmdup, then sort BAMs by coordinate — InPACT requires --input_file/-i to be a coordinate-sorted BAM. Run InPACT against the RefSeq GTF (hg38.109.20211119.ncbiRefSeq.gtf from UCSC goldenPath) with the precomputed InPACT_polyAsites.hg38.saf candidate-site file and 5 threads (-P 5), keeping default thresholds of >=5 spliced reads for skipped terminal exons and >=10 boundary-crossing reads for composite terminal exons. The rationale is that InPACT's read module relies on splice-in and boundary-crossing read evidence in the alignment, so alignment quality and filtering directly control which candidate IPA sites pass these count-based filters.

## Related tools

- **InPACT** (Downstream IPA site identification and quantification tool consuming the coordinate-sorted BAM; also assembles novel IPA isoforms (InPACT_transcript) and computes IPA usage (InPACT_quantify)) — https://github.com/YY-TMU/InPACT
- **HISAT2** (Aligns RNA-seq reads to the GRCh38 reference genome index to produce the BAM input required by InPACT)
- **SAMtools** (Sorts BAMs by coordinate, filters secondary-alignment flags, and removes PCR duplicates (rmdup) before InPACT)
- **Salmon** (Indexes (k-mer size 31) and quantifies transcript abundances from gffread-extracted novel IPA transcript FASTAs; quant.sf feeds InPACT_quantify) — https://github.com/COMBINE-lab/salmon
- **gffread** (Extracts transcript sequences from the merged.gtf of novel IPA isoforms for Salmon indexing)
- **Bowtie** (Removes reads mapped to rRNA sequences prior to IPA prediction)

## Examples

```
hisat2 -x GRCh38_index -1 reads1.fq -2 reads2.fq | samtools view -b -F 256 - | samtools sort -o sample.sorted.bam -; InPACT -i sample.sorted.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5
```

## Evaluation signals

- Output table contains the IPA usage (PAU) column and per-condition novel IPA event counts, reproducible end-to-end from the input BAMs via the InPACT pipeline
- Replicate concordance reproduces published benchmarks, e.g. 471 and 393 novel IPA sites in HEK293 replicates with ~218 overlapping sites and Pearson r = 0.94 for novel isoform expression
- Differential condition test reproduces e.g. 767 novel IPA events in U1-AMO-treated HeLa vs 151 in control, with significantly increased IPA usage (Wilcoxon rank sum test, P < 2.2e-16)
- BAMs are coordinate-sorted with secondary alignments, rRNA reads, and duplicates removed before InPACT is invoked
- Predicted IPA events are classified into skipped and composite types consistent with the >=5 spliced-read and >=10 boundary-crossing-read thresholds

## Limitations

- InPACT's read module depends on alignment-derived read counts; the precision of competing tools like IPAFinder 'relies on read coverage fluctuations, which necessitates adequate sequencing depth', a caveat that applies to coverage-sensitive IPA detection generally
- No version numbers are specified for InPACT or compared tools, so exact reproducibility of alignment/tool versions is unverified
- The candidate polyA-site file (InPACT_polyAsites.hg38.saf) is specific to GRCh38/RefSeq; switching annotation sources (GENCODE, PolyA_DB 3, PolyASite 2.0) requires regeneration
- InPACT test runs take about an hour with five cores, so large cohorts require substantial compute

## Evidence

- [methods] Align the U1-AMO-treated and control HeLa RNA-seq reads to the human reference genome (GRCh38) using HISAT2 and coordinate-sort the resulting BAM files.: [evidence span withheld — anchor status: not_found]
- [methods] Reads flagged as secondary alignment were filtered out using SAMtools: "Reads flagged as secondary alignment were filtered out using SAMtools"
- [methods] InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5: "InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5"
- [readme] A BAM file that would be used to predict IPA sites and it should be sorted by coordinates.: "--input_file/-i: A BAM file that would be used to predict IPA sites and it should be sorted by coordinates."
- [readme] Minimum number of spliced reads required to consider the potential skipped terminal exon. [default=5]: "Minimum number of spliced reads required to consider the potential skipped terminal exon. [default=5]"
