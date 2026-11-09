---
name: spliceu-augmented-transcriptome-alignment
description: Use when scRNA-seq reads must be aligned to an augmented spliced+unspliced
  transcriptome so per-read splicing status can be predicted or evaluated, or when
  uniquely genome-mapped and fully exonic (ambiguous) read sets are needed.
when_to_use_negative:
- Standard gene-level UMI quantification on the native transcriptome (e.g., plain
  Cell Ranger-style counting) where the doubled spliced+unspliced reference adds cost
  without benefit
- When no matching genome FASTA and GTF are available to augment into a spliceu reference
- When the goal is only measuring gene/intron UMI counts via STAR --soloFeatures GeneFull
  without per-read splicing-status prediction
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0622
tools:
- name: R (build_spliceu.R)
  role: Builds the spliceu augmented transcriptome reference (spliceu.gtf, spliceu.fa,
    t2g_3col.tsv) from genome.fa and genes.gtf
  repo: https://github.com/COMBINE-lab/forseti
- name: STAR
  role: Generates genome indices over the spliceu reference and aligns reads (--runThreadN
    12, --quantMode TranscriptomeSAM, --outSAMtype BAM SortedByCoordinate)
- name: samtools
  role: Filters reads that map uniquely to the genome using the -d NH:1 flag
- name: bedtools
  role: Finds reads 100% mapped in an exonic region (ambiguous reads) via bedtools
    intersect -f 1.0
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae207
    title: '<tt>Forseti</tt> : a mechanistic and predictive model of the splicing
      status of scRNA-seq reads'
  source_content_hash: sha256:d5dd47859596af7b56f6495f4ab2f9795840daf07f1acf50c7529bd06a3ff99f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/COMBINE-lab/forseti
  tools:
  - Forseti
  - Python
  - R
  - R (build_spliceu.R)
  - STAR
  - bedtools
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# spliceu-augmented-transcriptome-alignment

## Summary

Builds and aligns against a 'spliceu' augmented reference — where each transcript gets a spliced and an unspliced version — so scRNA-seq reads can be mapped in a way that supports per-read splicing-status prediction (e.g., by Forseti) and downstream AUC evaluation.

## When to use

Use when scRNA-seq reads must be aligned to an augmented spliced+unspliced transcriptome so per-read splicing status can be predicted or evaluated, or when uniquely genome-mapped and fully exonic (ambiguous) read sets are needed.

## When NOT to use

- Standard gene-level UMI quantification on the native transcriptome (e.g., plain Cell Ranger-style counting) where the doubled spliced+unspliced reference adds cost without benefit
- When no matching genome FASTA and GTF are available to augment into a spliceu reference
- When the goal is only measuring gene/intron UMI counts via STAR --soloFeatures GeneFull without per-read splicing-status prediction

## Inputs

- 10x scRNA-seq FASTQ files (e.g., PBMC1k v3 healthy donor R1/R2 FASTQs)
- Reference genome FASTA and gene annotation GTF (e.g., refdata-gex-GRCh38-2020-A)
- 10x cell barcode whitelist (e.g., 3M-february-2018.txt)
- Pre-built STAR spliceu genome index

## Outputs

- spliceu augmented reference: spliceu.gtf, spliceu.fa, t2g_3col.tsv
- STAR genome indices generated from the spliceu reference
- Coordinate-sorted genome BAM (uniquely mapped reads after samtools -d NH:1 filtering)
- toTranscriptome BAM (alignments translated into transcript coordinates)
- List of fully exonic (ambiguous) reads identified via bedtools intersect -f 1.0

## How to apply

First build the spliceu augmented reference by running `build_spliceu.R` on a standard genome FASTA and gene GTF (e.g., refdata-gex-GRCh38-2020-A), which yields spliceu.gtf, spliceu.fa, and t2g_3col.tsv; move genome.fa and genes.gtf into the index directory. Generate STAR genome indices over these files (--runMode genomeGenerate, --sjdbOverhang 100) in a separate conda env (build_spliceu_r) from the one used for prediction (forseti_test). Align reads with STAR (--runThreadN 12, --outSAMtype BAM SortedByCoordinate, --quantMode TranscriptomeSAM) to obtain both coordinate-sorted genome BAMs and transcript-coordinate toTranscriptome BAMs; note the tutorial passes the R2 FASTQs in --readFilesIn before R1 for 10x chemistry. Then filter uniquely genome-mapped reads with samtools using the -d NH:1 flag and identify fully exonic reads (ambiguous reads) with bedtools intersect using -f 1.0. These filtered BAMs and the toTranscriptome alignments are the direct inputs to Forseti's forseti_predictor, and discarding reads whose final splicing_status remains 'A' is part of the downstream evaluation.

## Related tools

- **R (build_spliceu.R)** (Builds the spliceu augmented transcriptome reference (spliceu.gtf, spliceu.fa, t2g_3col.tsv) from genome.fa and genes.gtf) — https://github.com/COMBINE-lab/forseti
- **STAR** (Generates genome indices over the spliceu reference and aligns reads (--runThreadN 12, --quantMode TranscriptomeSAM, --outSAMtype BAM SortedByCoordinate))
- **samtools** (Filters reads that map uniquely to the genome using the -d NH:1 flag)
- **bedtools** (Finds reads 100% mapped in an exonic region (ambiguous reads) via bedtools intersect -f 1.0)

## Examples

```
STAR --runThreadN 12 --readFilesCommand zcat --genomeDir $STAR_IDX_DIR --outFileNamePrefix $STAR_out --outSAMtype BAM SortedByCoordinate --quantMode TranscriptomeSAM --soloFeatures GeneFull
```

## Evaluation signals

- The spliceu build produces all three expected artifacts: spliceu.gtf, spliceu.fa, and t2g_3col.tsv
- STAR output contains a toTranscriptome BAM in addition to the coordinate-sorted genome BAM (confirming --quantMode TranscriptomeSAM was applied)
- After samtools -d NH:1 filtering, every retained alignment carries a single NH value (uniquely mapped to the genome)
- bedtools intersect with -f 1.0 flags reads fully contained in exonic regions as ambiguous, matching the article's ambiguous-read ('A') definition
- Downstream Forseti prediction on these alignments yields AUC values in the reported simulated-data range 0.85-0.93 (mean ~0.92) after discarding reads with final splicing_status 'A'

## Limitations

- Fundamentally ambiguous reads (polyA-tail priming with read2 contained in the terminal exon, or cDNA fragments arising entirely from an exon) cannot be distinguished even by the mechanistic Forseti model
- The 10x 1kPBMC sample produces ~200 million transcriptome alignments; extracting the top 200 cells reduces this to 65,784,347 alignments and still takes ~3.3 hours on 32 threads for prediction
- The forseti-experiments pipeline requires ~3 TB of disk space, and Nextflow cannot guarantee fetched FASTQs are uncorrupted (failed prepare_data runs require editing input_files/sample_url_sheet.csv and re-running)

## Evidence

- [readme] The R script `build_spliceu.R` will build a _spliceu_ (spliced + unspliced), an augmented transcriptome reference: "The R script `build_spliceu.R` will build a _spliceu_ (spliced + unspliced), an augmented transcriptome reference"
- [readme] With `--quantMode TranscriptomeSAM`, STAR will output alignments translated into transcript coordinates: "With `--quantMode TranscriptomeSAM`, STAR will output alignments translated into transcript coordinates"
- [readme] we use samtools to filter the reads have unique map to the genome.: "we use samtools to filter the reads have unique map to the genome."
- [methods] find the reads 100% mapped in a exonic region (i.e. ambiguous reads): "# find the reads 100% mapped in a exonic region (i.e. ambiguous reads)"
- [readme] we simulate two sets of sequencing reads with and without fundamental ambiguity: [evidence span withheld — anchor status: external_doc]
