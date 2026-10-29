---
name: cuffcompare-based-assembly-metrics
description: Use when you have assembled transcript GTF files from one or more RNA-seq
  assemblers plus a reference GTF (e.g. NCBI RefSeq hg38 or mm10.ncbiRefSeq.gtf) and
  need transcript-level correctness, precision, and F-score metrics.
when_to_use_negative:
- The assembly is annotation-guided or already matched to the reference, so a reference-free
  comparison would be redundant
- Input alignments are long-read RNA-seq data (PacBio/ONT), which Tiglon and this
  benchmark design do not support
- You need quantification or differential expression rather than transcript-structure
  accuracy
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: Cuffcompare
  role: Compares each assembler's GTF against the RefSeq reference GTF to count correct
    transcripts, precision, and F-score
- name: Tiglon
  role: Produces the evaluated assemblies from Hisat2/Star/merged alignments
  repo: github.com/yutingsdu/Tiglon
- name: StringTie2
  role: Baseline assembler whose per-sample and merged (StringTie2-Merge) assemblies
    are evaluated
  repo: gpertea/stringtie
- name: Scallop
  role: Baseline assembler run on Star (or Hisat2) alignments whose assemblies are
    evaluated
  repo: Kingsford-Group/scallop
- name: Hisat2
  role: Splice-aware aligner producing one of the BAM alignment sets feeding the evaluated
    assemblies
  repo: DaehwanKimLab/hisat2
- name: STAR
  role: Splice-aware aligner producing the second BAM alignment set feeding the evaluated
    assemblies
  repo: alexdobin/STAR
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2022.104067
    title: Tiglon enables accurate transcriptome assembly via integrating mappings
      of different aligners
  source_content_hash: sha256:aeccd94ad533faaa913d9f18be1b37ad9490c29eaa3452801eb5ca89d13a5044
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cuffcompare
  - Hisat2
  - STAR
  - Scallop
  - Star
  - StringTie2
  - Tiglon
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# cuffcompare-based-assembly-metrics

## Summary

Evaluate transcriptome assemblies (GTF) against a reference annotation using Cuffcompare to count correct transcripts, precision, and F-score, as done when benchmarking Tiglon against StringTie2, Scallop, and StringTie2-Merge on simulated and real RNA-seq datasets.

## When to use

Use when you have assembled transcript GTF files from one or more RNA-seq assemblers plus a reference GTF (e.g. NCBI RefSeq hg38 or mm10.ncbiRefSeq.gtf) and need transcript-level correctness, precision, and F-score metrics.

## When NOT to use

- The assembly is annotation-guided or already matched to the reference, so a reference-free comparison would be redundant
- Input alignments are long-read RNA-seq data (PacBio/ONT), which Tiglon and this benchmark design do not support
- You need quantification or differential expression rather than transcript-structure accuracy

## Inputs

- Assembled transcript GTF files (e.g. Tiglon output GTF, StringTie2/Scallop/StringTie2-Merge output GTF)
- Reference annotation GTF (e.g. hg38 NCBI RefSeq transcriptome, mm10.ncbiRefSeq.gtf)

## Outputs

- Cuffcompare comparison output with counts of correct transcripts per assembly
- Computed precision and F-score per assembler-condition
- Benchmark table (e.g. simulated_benchmark.csv with 7 rows; mouse_benchmark.csv with aggregated metrics)

## How to apply

Run Cuffcompare with the reference annotation (e.g. the NCBI RefSeq GTF for hg38 GRCh38 or GRCm38/mm10) passed via -r and each assembler's output GTF as the query. Assemblies compared in the Tiglon benchmark include Tiglon run on Hisat2 alignments, Star alignments, or merged alignments of both, StringTie2 and Scallop under Hisat2/Star alignments, and StringTie2-Merge. Cuffcompare outputs counts of correctly assembled transcripts (matching the reference transcriptome), from which precision and F-score are computed; per-dataset results are compiled into a benchmark table such as simulated_benchmark.csv (7 rows, one per assembler-condition) or mouse_benchmark.csv (aggregated over the four M. musculus samples M1–M4). For expression-stratified evaluation, stratify reference transcripts by expression level before scoring to quantify recovery of lowly expressed transcripts.

## Related tools

- **Cuffcompare** (Compares each assembler's GTF against the RefSeq reference GTF to count correct transcripts, precision, and F-score)
- **Tiglon** (Produces the evaluated assemblies from Hisat2/Star/merged alignments) — github.com/yutingsdu/Tiglon
- **StringTie2** (Baseline assembler whose per-sample and merged (StringTie2-Merge) assemblies are evaluated) — gpertea/stringtie
- **Scallop** (Baseline assembler run on Star (or Hisat2) alignments whose assemblies are evaluated) — Kingsford-Group/scallop
- **Hisat2** (Splice-aware aligner producing one of the BAM alignment sets feeding the evaluated assemblies) — DaehwanKimLab/hisat2
- **STAR** (Splice-aware aligner producing the second BAM alignment set feeding the evaluated assemblies) — alexdobin/STAR

## Examples

```
cuffcompare -r mm10.ncbiRefSeq.gtf -o tiglon_vs_refseq Tiglon_hisat2_star.gtf
```

## Evaluation signals

- simulated_benchmark.csv contains precision and F-score columns and one row per assembler-condition (7 rows total)
- Simulated-dataset row for Tiglon reproduces 22,928 correct transcripts, 64.01% precision, and F-score 0.480
- Mouse benchmark (M1–M4) shows Tiglon average precision ~51.71% and average F-score ~0.2341, i.e. ~7.45% higher than StringTie-Merge
- Correct-transcript counts for baselines match reported values (StringTie2-Merge 20,630; StringTie2+MergedAlignments 20,654; Scallop+MergedAlignments 20,277)
- All compared assemblies use the same reference GTF (RefSeq) so counts are directly comparable across assemblers

## Limitations

- Cuffcompare-based correct-transcript counting depends entirely on the chosen reference annotation (NCBI RefSeq); assemblies containing novel transcripts outside RefSeq will score as incorrect
- Precision and F-score reflect reference matching only and do not measure quantification accuracy
- Metrics are conditional on upstream assembly quality; Cuffcompare does not fix assembly errors such as missing junctions from a single aligner

## Evidence

- [abstract] the state-of-the-art assemblers, including StringTie2 and Scallop: "the state-of-the-art assemblers, including StringTie2 and Scallop"
- [results] mapped simulated reads with Hisat2 and Star: "we mapped the simulated reads to the reference genome by using Hisat2 and Star"
- [other] Cuffcompare evaluation against RefSeq: [evidence span withheld — anchor status: not_found]
- [readme] Tiglon outputs GTF transcripts: "The software takes a file that lists alignment files generated by different aligners as input, and outputs all assembled candidate transcripts in GTF format."
