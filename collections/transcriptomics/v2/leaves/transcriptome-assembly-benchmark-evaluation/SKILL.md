---
name: transcriptome-assembly-benchmark-evaluation
description: 'Use when comparing transcriptome assembler output on the same RNA-seq
  reads: you have multiple GTF assemblies (e.g., from single-aligner runs and merged-alignment
  runs) and need correct-transcript counts, precision, and F-score against a known
  reference.'
when_to_use_negative:
- The current Tiglon version has no annotation-guided assembly option, so this benchmark
  design cannot measure annotation-guided assembly performance
- 'Data are long-read RNA-seq (PacBio or Oxford Nanopore): the current Tiglon version
  is not compatible with long-read datasets'
- You need a single-assembler sensitivity analysis only; integrating alignments of
  different aligners is the point of this benchmark
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: Tiglon
  role: Integrates multiple aligner BAMs into labeled splice graphs and assembles
    transcripts; run on Hisat2, Star, and merged alignments
  repo: github.com/yutingsdu/Tiglon
- name: StringTie2
  role: Baseline assembler run separately on Hisat2 and Star alignments; StringTie2-Merge
    provides the merged-assembly baseline
  repo: https://github.com/gpertea/stringtie
- name: Scallop
  role: Baseline assembler run separately on Hisat2 and Star alignments and on merged
    alignments
  repo: https://github.com/Kingsford-Group/scallop
- name: Hisat2
  role: Splice-aware aligner producing one of the two alignment conditions
  repo: https://github.com/DaehwanKimLab/hisat2
- name: STAR
  role: Splice-aware aligner producing the second alignment condition
  repo: https://github.com/alexdobin/STAR
- name: Cuffcompare
  role: Evaluates each assembly GTF against the RefSeq transcriptome to count correct
    transcripts, precision, and F-score
- name: BamTools
  role: Prerequisite dependency for building Tiglon
  repo: https://github.com/pezmaster31/bamtools
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2022.104067
    title: Tiglon enables accurate transcriptome assembly via integrating mappings
      of different aligners
  source_content_hash: sha256:d88bfa87a21eedaa2a273eeb0493aa634b47b970c4f108c4581d452695257c93
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BamTools
  - Cuffcompare
  - Hisat2
  - STAR
  - Scallop
  - Star
  - StringTie2
  - Tiglon
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# transcriptome-assembly-benchmark-evaluation

## Summary

Benchmark genome-guided transcriptome assemblers (Tiglon, StringTie2, Scallop, StringTie2-Merge) under different alignment conditions (Hisat2, Star, merged) by evaluating assembled GTFs against a reference transcriptome (NCBI RefSeq hg38) with Cuffcompare, counting correct transcripts, precision, and F-score.

## When to use

Use when comparing transcriptome assembler output on the same RNA-seq reads: you have multiple GTF assemblies (e.g., from single-aligner runs and merged-alignment runs) and need correct-transcript counts, precision, and F-score against a known reference.

## When NOT to use

- The current Tiglon version has no annotation-guided assembly option, so this benchmark design cannot measure annotation-guided assembly performance
- Data are long-read RNA-seq (PacBio or Oxford Nanopore): the current Tiglon version is not compatible with long-read datasets
- You need a single-assembler sensitivity analysis only; integrating alignments of different aligners is the point of this benchmark

## Inputs

- RSEM-simulated RNA-seq reads (~52 million 100-bp paired-end, parameters learned from SRR7536920)
- Reference genome GRCh38/hg38 and NCBI RefSeq transcriptome annotation
- BAM alignment files from Hisat2 and Star
- GTF transcript assemblies from Tiglon, StringTie2, Scallop, and StringTie2-Merge

## Outputs

- One GTF assembly per assembler-condition pair (7 conditions)
- simulated_benchmark.csv with one row per condition containing correct transcripts, precision, and F-score

## How to apply

Generate a common evaluation set: for the Tiglon paper, ~52 million 100-bp paired-end reads were simulated with RSEM with model parameters learned from the real human RNA-seq sample SRR7536920 over the hg38 NCBI RefSeq transcriptome. Map reads with two splice-aware aligners (Hisat2 and Star), then assemble under each condition: each assembler on Hisat2 alignments, each on Star alignments, Tiglon and merged-alignment runs on both (e.g., 'Tiglon -B bamFile_list -s first -o Tiglon_outdir'), plus StringTie2-Merge for the merged condition. Evaluate all assembler-condition assemblies against the RefSeq transcriptome with Cuffcompare, recording correct transcripts, precision, and F-score per condition (7 rows in simulated_benchmark.csv). A correct result must reproduce Tiglon's reported simulated-dataset numbers: 22,928 correct transcripts, 64.01% precision, F-score 0.480, exceeding StringTie2-Merge (20,630), StringTie2+MergedAlignments (20,654), and Scallop+MergedAlignments (20,277).

## Related tools

- **Tiglon** (Integrates multiple aligner BAMs into labeled splice graphs and assembles transcripts; run on Hisat2, Star, and merged alignments) — github.com/yutingsdu/Tiglon
- **StringTie2** (Baseline assembler run separately on Hisat2 and Star alignments; StringTie2-Merge provides the merged-assembly baseline) — https://github.com/gpertea/stringtie
- **Scallop** (Baseline assembler run separately on Hisat2 and Star alignments and on merged alignments) — https://github.com/Kingsford-Group/scallop
- **Hisat2** (Splice-aware aligner producing one of the two alignment conditions) — https://github.com/DaehwanKimLab/hisat2
- **STAR** (Splice-aware aligner producing the second alignment condition) — https://github.com/alexdobin/STAR
- **Cuffcompare** (Evaluates each assembly GTF against the RefSeq transcriptome to count correct transcripts, precision, and F-score)
- **BamTools** (Prerequisite dependency for building Tiglon) — https://github.com/pezmaster31/bamtools

## Examples

```
Tiglon -B bamFile_list -s first -o Tiglon_outdir
```

## Evaluation signals

- simulated_benchmark.csv contains exactly 7 rows (one per assembler-condition) with precision and F-score columns
- Tiglon row reproduces 22,928 correct transcripts, 64.01% precision, and F-score 0.480 on the simulated dataset
- Tiglon row exceeds StringTie2-Merge (20,630 correct), StringTie2+MergedAlignments (20,654), and Scallop+MergedAlignments (20,277)
- Every assembly GTF was produced under identical simulated input reads and evaluated against the same RefSeq transcriptome with Cuffcompare
- Tiglon was run with a BAM file list covering both aligners (e.g., 'Tiglon -B bamFile_list -s first -o Tiglon_outdir') for the merged condition

## Limitations

- The current Tiglon version does not support annotation-guided assemblies using known transcript annotations
- The current Tiglon version is not compatible with long-read RNA-seq datasets (PacBio or ONT)
- Combining Tiglon's integrating strategy with TransBorrow's borrowing strategy is proposed as future work, not implemented
- Tiglon accepts only BAM files generated by Hisat(2) and Star; other aligners are unsupported

## Evidence

- [results] On the simulated dataset, Tiglon correctly assembled 22,928 transcripts: [evidence span withheld — anchor status: not_found]
- [results] Simulation parameters learned from real data: "The parameters of the simulation model were learned from the real human RNA-seq dataset with the NCBI SRA accession code of SRR7536920"
- [results] Simulated reads mapped with two aligners: "we mapped the simulated reads to the reference genome by using Hisat2 and Star"
- [abstract] Benchmark scope: "We evaluate Tiglon on a simulated dataset and 12 real datasets under the Hisat2 and Star mappings"
- [other] Merged-alignment invocation: "Tiglon -B bamFile_list -s first -o Tiglon_outdir"
- [readme] Tiglon input format restriction: "The current version of Tiglon can accepts BAM files generated by Hisat(2) and Star."
