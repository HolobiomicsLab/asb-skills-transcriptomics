---
name: splice-aware-rna-seq-read-alignment
description: Use when starting from RNA-seq FASTQ files (e.g., SRA accessions like
  SRR307911) and you need genome-aligned BAMs that preserve splice junctions for transcriptome
  assembly, especially when integrating multiple aligner mappings.
when_to_use_negative:
- Input is already assembled transcript GTF output from an assembler — no re-alignment
  is needed; go straight to evaluation with Cuffcompare.
- 'The reads are long-read RNA-seq (PacBio or ONT): the article notes the current
  Tiglon version is not compatible with long-read datasets, and this skill targets
  short-read spliced alignment.'
- Only quantification at the transcriptome level is required without genome-guided
  assembly — a pseudoaligner workflow would avoid genome alignment entirely.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: HISAT2
  role: Splice-aware aligner run with --dta to produce junction-preserving BAMs of
    the RNA-seq reads
  repo: https://github.com/DaehwanKimLab/hisat2
- name: STAR
  role: Second splice-aware aligner producing an independent BAM so junctions missed
    by HISAT2 can be recovered by integration
  repo: https://github.com/alexdobin/STAR
- name: samtools
  role: Sorting per-aligner BAMs and merging HISAT2 and STAR alignments into the combined-alignment
    condition
- name: Tiglon
  role: Downstream consumer of the per-aligner/merged BAMs via a bamFile_list to build
    labeled splice graphs
  repo: https://github.com/yutingsdu/Tiglon
merged_aliases:
- splice-aware-rna-seq-alignment-with-star
merged_alias_records:
- alias: rna-seq-splice-aware-read-alignment
  slug: rna-seq-splice-aware-read-alignment
  jaccard_score: 1.0
  method: token-set-jaccard
  decision: auto
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_003
  - task_005
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2022.104067
    title: Tiglon enables accurate transcriptome assembly via integrating mappings
      of different aligners
  source_content_hash: sha256:7053b7b465583daf1cdcd8c7039103f50ac20bcc570b5309b12e80af870748ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaehwanKimLab/hisat2
  tools:
  - HISAT2
  - Hisat2
  - PAIRADISE
  - STAR
  - Scallop
  - Star
  - StringTie2
  - Tiglon
  - pybedtools
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# splice-aware-rna-seq-read-alignment

## Summary

Generate splice-aware RNA-seq alignments by mapping reads to a reference genome (e.g., GRCh38 or GRCm38) with two complementary splice-aware aligners, HISAT2 (with --dta) and STAR, producing coordinate-sorted BAM files that downstream assemblers such as Tiglon, StringTie2, and Scallop can consume.

## When to use

Use when starting from RNA-seq FASTQ files (e.g., SRA accessions like SRR307911) and you need genome-aligned BAMs that preserve splice junctions for transcriptome assembly, especially when integrating multiple aligner mappings.

## When NOT to use

- Input is already assembled transcript GTF output from an assembler — no re-alignment is needed; go straight to evaluation with Cuffcompare.
- The reads are long-read RNA-seq (PacBio or ONT): the article notes the current Tiglon version is not compatible with long-read datasets, and this skill targets short-read spliced alignment.
- Only quantification at the transcriptome level is required without genome-guided assembly — a pseudoaligner workflow would avoid genome alignment entirely.

## Inputs

- RNA-seq paired-end FASTQ files (e.g., SRA accessions SRR307911, SRR7536920, DRR205674)
- Reference genome FASTA (e.g., GRCh38/hg38 for human, GRCm38/mm10 for mouse)
- Prebuilt aligner indices (HISAT2 graph FM index; STAR genome index)

## Outputs

- Coordinate-sorted BAM per aligner per sample (e.g., Hisat.bam, Star.bam)
- Merged BAM combining HISAT2 and STAR alignments (via samtools merge)
- bamFile_list text file listing one BAM path per line for Tiglon input

## How to apply

Download the RNA-seq FASTQ files and align them to the reference genome twice: once with HISAT2 using the --dta option (so the BAM records junction coverage compatible with StringTie/Tiglon) and once with STAR, then sort the resulting BAMs with samtools. Retain both aligner-specific BAM sets rather than only one, because the Tiglon paper's motivating observation is that HISAT2 and STAR alignments each cover whole exons of a transcript but neither alone covers all its splice junctions. Optionally merge the HISAT2 and STAR BAMs with samtools merge into a combined-alignment file, and write a bamFile_list (one BAM path per line) so Tiglon can build its labeled splice graph, labeling junctions as captured by all aligners (label 2) or only some (label 1). For robustness checks, HISAT2 default mode (without --dta) can also be run, since the paper reports a similar performance trend without the --dta flag. Correct application yields one sorted BAM per aligner per sample, ready for assembler evaluation with Cuffcompare against RefSeq annotations such as hg38.ncbiRefSeq.gtf or mm10.ncbiRefSeq.gtf.

## Related tools

- **HISAT2** (Splice-aware aligner run with --dta to produce junction-preserving BAMs of the RNA-seq reads) — https://github.com/DaehwanKimLab/hisat2
- **STAR** (Second splice-aware aligner producing an independent BAM so junctions missed by HISAT2 can be recovered by integration) — https://github.com/alexdobin/STAR
- **samtools** (Sorting per-aligner BAMs and merging HISAT2 and STAR alignments into the combined-alignment condition)
- **Tiglon** (Downstream consumer of the per-aligner/merged BAMs via a bamFile_list to build labeled splice graphs) — https://github.com/yutingsdu/Tiglon

## Examples

```
hisat2 --dta -x GRCh38 -1 reads_1.fq.gz -2 reads_2.fq.gz | samtools sort -o Hisat.bam; STAR --genomeDir GRCh38_index --readFilesIn reads_1.fq.gz reads_2.fq.gz --outSAMtype BAM SortedByCoordinate
```

## Evaluation signals

- Each sample yields two coordinate-sorted BAM files, one from HISAT2 (--dta) and one from STAR, that are accepted by StringTie2/Scallop/Tiglon without format errors.
- For a known transcript (e.g., XR_929,880.3 in SRR307911), both HISAT2 and STAR BAMs should cover all its exons while neither alone covers all its splice junctions — this is the expected motivating pattern.
- samtools merge succeeds on the HISAT2 and STAR BAMs and the merged BAM plus bamFile_list are accepted by Tiglon ('Tiglon -B bamFile_list -s first -o Tiglon_outdir').
- Downstream Cuffcompare evaluation against hg38.ncbiRefSeq.gtf or mm10.ncbiRefSeq.gtf produces per-assembler correct-transcript counts consistent with the paper's reported ranges (e.g., Tiglon recovering ~9.51% more correct transcripts than StringTie2-Merge on H1–H8).

## Limitations

- Neither aligner alone covers all splice junctions of a transcript, which is exactly why single-aligner assemblies miss transcripts that the combined mappings recover.
- HISAT2 run without --dta changes junction-read bookkeeping; the paper shows the performance trend holds, but the --dta mode is the default configuration in the benchmark workflows.
- Splice-aware alignment is genome-guided only; the article notes current Tiglon has no annotation-guided assembly option, so annotation information is not used at this step.

## Evidence

- [abstract] We evaluate Tiglon on a simulated dataset and 12 real datasets under the Hisat2 and Star mappings: "We evaluate Tiglon on a simulated dataset and 12 real datasets under the Hisat2 and Star mappings"
- [intro] both aligners cover exons but not all junctions: "both the Hisat2 and Star (two of the best current aligners) alignments cover the whole exons of this transcript, but neither of them covers the entire junctions"
- [results] simulated reads mapped with both aligners: "we mapped the simulated reads to the reference genome by using Hisat2 and Star"
- [readme] Tiglon input is a list of alignment files from different aligners: "The software takes a file that lists alignment files generated by different aligners as input, and outputs all assembled candidate transcripts in GTF format."
