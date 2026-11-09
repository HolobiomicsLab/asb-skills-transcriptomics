---
name: sra-rnaseq-data-retrieval
description: Use when reproducing the UMI mapping-shift analysis or Table 1 cluster-size
  read fractions and raw FASTQ files for any of run_171108, run_170420, SCRB, La Manno,
  Yanai1, or Yanai2 are needed.
when_to_use_negative:
- The analysis does not involve UMI-containing libraries or the UMI-analysis pipeline,
  e.g. paired-end bulk RNA-Seq differential expression projects where the 5'-UMI clipping
  and single-end assumption do not apply.
- Input is already quality-filtered, UMI-clipped, or a sorted SAM/BAM file — the downstream
  steps (umi_counterizer.pl, umi_analyzer_uthash) expect raw FASTQ with the UMI still
  at the 5' end of each read.
- The target data set is not one of the article's six, and no 5'-UMI position/length
  is known for it, since fastq_umi_clipper requires an explicit clip_length and optional
  g_length (e.g. protocol-derived trailing GGG/GGGG from the Clontech SMARTer system).
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: fastq_qual_filter
  role: First consumer of the retrieved FASTQ files; filters on a mean-quality substring
    (e.g. min_ok 30) and requires demultiplexed single-index input
  repo: https://github.com/ncgr/UMI-analysis
- name: fastq_umi_clipper
  role: Removes the 5' UMI (clip_length) and optional trailing Gs (g_length) from
    the retrieved reads before alignment
  repo: https://github.com/ncgr/UMI-analysis
- name: STAR
  role: Aligns the retrieved, clipped reads to the genomic reference (no introns)
    producing the sorted SAM needed by umi_counterizer.pl
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:a062556a64461fd32fb9535414a35307764417cdb33495c7fcf08ac35a91f5ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncgr/UMI-analysis
  tools:
  - C
  - STAR
  - Text::Levenshtein::XS
  - Try::Tiny
  - fastq_qual_filter
  - fastq_umi_clipper
  - pcregrep
  - uthash
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# sra-rnaseq-data-retrieval

## Summary

Retrieves raw single-end FASTQ reads for the six UMI-containing RNA-Seq data sets used in the Sena et al. 2018 UMI mapping-shift study (run_171108, run_170420, SCRB, La Manno, Yanai1, Yanai2) from their SRA accessions, so that UMI-read cluster distributions can be reproduced downstream.

## When to use

Use when reproducing the UMI mapping-shift analysis or Table 1 cluster-size read fractions and raw FASTQ files for any of run_171108, run_170420, SCRB, La Manno, Yanai1, or Yanai2 are needed.

## When NOT to use

- The analysis does not involve UMI-containing libraries or the UMI-analysis pipeline, e.g. paired-end bulk RNA-Seq differential expression projects where the 5'-UMI clipping and single-end assumption do not apply.
- Input is already quality-filtered, UMI-clipped, or a sorted SAM/BAM file — the downstream steps (umi_counterizer.pl, umi_analyzer_uthash) expect raw FASTQ with the UMI still at the 5' end of each read.
- The target data set is not one of the article's six, and no 5'-UMI position/length is known for it, since fastq_umi_clipper requires an explicit clip_length and optional g_length (e.g. protocol-derived trailing GGG/GGGG from the Clontech SMARTer system).

## Inputs

- SRA accession identifiers for the six data sets (run_171108, run_170420, SCRB, La Manno, Yanai1, Yanai2)
- Raw single-end FASTQ files (demultiplexed, one index per file)
- Reference genomic sequence for subsequent STAR alignment (e.g. Physcomitrella patens or the data set's matching genome)

## Outputs

- Per-data-set FASTQ files ready for fastq_qual_filter (infile argument) and fastq_umi_clipper (clip_length, g_length arguments)
- A retrieval manifest mapping each data set name to its SRA accession and file path

## How to apply

Download the FASTQ files for each of the six data sets from their SRA accessions, demultiplexing first if a file still contains more than one index, since the downstream fastq_qual_filter assumes demultiplexed input containing only one index. The retrieved reads are single-end, matching the pipeline's stated use case, and each file must retain the 5'-UMI position information so that fastq_umi_clipper can move the correct number of nucleotides (clip_length) into the FASTQ header. Retrieve all six data sets because the article's finding is comparative: only ~19% of reads in run_171108 fall in size-one UMI-read clusters versus ~75% in the La Manno data set, and the SCRB, Yanai1, and Yanai2 (CEL-Seq2) data sets span the intermediate spread. After retrieval, validate files by running them through the UMI-analysis pipeline's quality filter and confirming the reads align with STAR (no introns) into sorted SAM files that umi_counterizer.pl can consume.

## Related tools

- **fastq_qual_filter** (First consumer of the retrieved FASTQ files; filters on a mean-quality substring (e.g. min_ok 30) and requires demultiplexed single-index input) — https://github.com/ncgr/UMI-analysis
- **fastq_umi_clipper** (Removes the 5' UMI (clip_length) and optional trailing Gs (g_length) from the retrieved reads before alignment) — https://github.com/ncgr/UMI-analysis
- **STAR** (Aligns the retrieved, clipped reads to the genomic reference (no introns) producing the sorted SAM needed by umi_counterizer.pl) — https://github.com/alexdobin/STAR

## Evaluation signals

- Each retrieved FASTQ is demultiplexed — it contains only one index — so fastq_qual_filter's demultiplexing assumption holds.
- Retrieved reads are single-end with a UMI at the 5' end whose length matches the clip_length supplied to fastq_umi_clipper.
- The full pipeline run on retrieved run_171108 data reproduces approximately 19% of reads in size-one UMI-read clusters, and La Manno reproduces approximately 75%.
- Mapping-shift density distributions computed from the retrieved data peak at zero shift with a bell-shaped decay, matching the reported Figures 2–4 equivalents.
- All six data sets are retrieved and processed so the comparative cluster-size spread (SCRB greatest, then Yanai1, La Manno, Yanai2) can be reproduced.

## Limitations

- The UMI-analysis pipeline's use case involves single-ended sequencing only; retrieved paired-end data would not fit the downstream steps without modification.
- Files must be demultiplexed before the pipeline: the tools assume the FASTQ 'has been demultiplexed already, i.e. contains only one index, or that you don't care what the indices are'.
- The specific SRA accession numbers are not stated in the article text itself and must be resolved from the data availability information before download.

## Evidence

- [abstract] This behaviour occurs in all such libraries that we have produced, and in multiple other UMI-containing RNA-Seq data sets in the public domain.: "This behaviour occurs in all such libraries that we have produced, and in multiple other UMI-containing RNA-Seq data sets in the public domain."
- [other] Download the FASTQ files for each dataset (run_171108, run_170420, SCRB, La Manno, Yanai1, Yanai2) from their SRA accessions and demultiplex if needed.: [evidence span withheld — anchor status: not_found]
- [readme] This assumes your FASTQ file has been demultiplexed already, i.e. contains only one index, or that you don't care what the indices are.: "This assumes your FASTQ file has been demultiplexed already, i.e. contains only one index, or that you don't care what the indices are."
- [readme] Our use case only involves single ended sequencing at this stage.: "Our use case only involves single ended sequencing at this stage."
- [discussion] The SCRB-Seq data have the greatest spread, followed by the CEL-Seq2 data set Yanai1: "The SCRB-Seq data have the greatest spread, followed by the CEL-Seq2 data set Yanai1"
