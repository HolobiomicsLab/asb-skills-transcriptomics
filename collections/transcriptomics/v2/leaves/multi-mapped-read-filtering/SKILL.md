---
name: multi-mapped-read-filtering
description: Use when assigning small RNA-seq FASTQ reads to tRNA-derived loci (tRF-5,
  tRF-3, tRF-1, tRH, tRNA halves) and ambiguous reads from repetitive tRNA-flanking
  regions or paralogous tRNA genes must be excluded before subtype classification.
when_to_use_negative:
- Reads are long paired-end or whole-transcript RNA-seq data rather than 14–50 nt
  small RNAs; the '--norc -v 0 -m 50' realignment to the plus strand is calibrated
  for short sRNA reads
- The goal is quantification of tncRNAs expressed from multi-copy tRNA families where
  keeping multi-mapped reads (scaled or probabilistic assignment) is the desired behavior
  instead of discarding them
- Existing human tncRNA tools (tDRmapper, MINTmap, tRF2Cancer) are adequate — they
  are trained and tested on human datasets, not plant genomes
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Bowtie1
  role: Performs the error-tolerant unique alignment ('-v 2 --best') to the artificial
    genome index and the plus-strand realignment with '-m 50' multi-map suppression
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: samtools
  role: Sorts SAM output and converts to BAM; extracts uniquely mapped reads via SAM
    flags 0 and 16
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: tncRNA Toolkit
  role: Wraps the mapping-and-filtering workflow; exposes '-m' (default 50) and '-v'
    mismatch options for the aligner
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: bedtools
  role: Masks tRNA regions with 50 bp flanks (bedtools maskfasta) when constructing
    the artificial genome used for the filtering alignment
  repo: https://github.com/skbinfo/tncRNA-Toolkit
- name: Trim Galore
  role: Produces the quality-filtered, adapter-trimmed FASTQ input that feeds the
    mapping and multi-map filtering step
  repo: https://github.com/FelixKrueger/TrimGalore
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.09.021
    title: 'Transfer RNA-derived non-coding RNAs (tncRNAs): Hidden regulation of plants''
      transcriptional regulatory circuits'
  source_content_hash: sha256:7149873c7cd81ac326fbd312ea020e9c11d07f80c8ae44a079493dcc3a6d3434
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/skbinfo/tncRNA-Toolkit
  repo_url: https://github.com/skbinfo/tncRNA-Toolkit
  tools:
  - Bowtie1
  - NCBI SRA Toolkit (v2.10)
  - Trim Galore
  - bedtools
  - biopython
  - pandas
  - samtools
  - tRNAscan-SE (v2.0.6)
  - tncRNA Toolkit
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# multi-mapped-read-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A read-mapping filter strategy for plant sRNA-seq tncRNA detection that keeps only error-tolerant, uniquely mapping reads: alignment to an artificial genome with bowtie '-v 2 --best', followed by realignment of uniquely mapped reads (SAM flags 0/16) to the plus strand with '-m 50' suppression of multi-mappers.

## When to use

Use when assigning small RNA-seq FASTQ reads to tRNA-derived loci (tRF-5, tRF-3, tRF-1, tRH, tRNA halves) and ambiguous reads from repetitive tRNA-flanking regions or paralogous tRNA genes must be excluded before subtype classification.

## When NOT to use

- Reads are long paired-end or whole-transcript RNA-seq data rather than 14–50 nt small RNAs; the '--norc -v 0 -m 50' realignment to the plus strand is calibrated for short sRNA reads
- The goal is quantification of tncRNAs expressed from multi-copy tRNA families where keeping multi-mapped reads (scaled or probabilistic assignment) is the desired behavior instead of discarding them
- Existing human tncRNA tools (tDRmapper, MINTmap, tRF2Cancer) are adequate — they are trained and tested on human datasets, not plant genomes

## Inputs

- Quality-filtered, adapter-trimmed single-end sRNA-seq FASTQ file
- Bowtie index of the masked reference genome with artificial chromosomes (mature tRNA + 50 nt leader + 50 nt trailer)
- BAM/SAM alignments of uniquely mapped reads (SAM flags 0 and 16)

## Outputs

- Set of uniquely mapped reads passing the -m 50 multi-map ceiling
- Per-read tncRNA subtype labels (tRF-5, tRF-3, tRF-1, 5'tRH, 3'tRH, tRNA halves) with RPM values
- Sorted/converted BAM files produced with samtools v1.10

## How to apply

First align quality-filtered, adapter-trimmed sRNA reads (processed by Trim Galore) to a Bowtie index built on the masked genome plus artificial chromosomes (tRNA regions with 50 nt leader/trailer flanks) using bowtie1 '-v 2 --best' to tolerate sequencing errors while enforcing unique mapping; rationale: allowing only a single mismatch inflates false negatives via base misidentification. Extract uniquely mapped reads from the BAM with samtools using SAM flags 0 and 16, then realign them to the plus strand only with bowtie1 '--norc -v 0 -m 50 --best --strata' and discard any read whose alignment occurs more than 50 times. Only reads that map exclusively to the artificial chromosomes (tRNA plus 50 nt flanks) are then classified as tncRNAs, which prevents ambiguous non-tRNA assignments; the tncRNA Toolkit exposes the multi-map ceiling as the '-m' option (default 50).

## Related tools

- **Bowtie1** (Performs the error-tolerant unique alignment ('-v 2 --best') to the artificial genome index and the plus-strand realignment with '-m 50' multi-map suppression) — https://github.com/skbinfo/tncRNA-Toolkit
- **samtools** (Sorts SAM output and converts to BAM; extracts uniquely mapped reads via SAM flags 0 and 16) — https://github.com/skbinfo/tncRNA-Toolkit
- **tncRNA Toolkit** (Wraps the mapping-and-filtering workflow; exposes '-m' (default 50) and '-v' mismatch options for the aligner) — https://github.com/skbinfo/tncRNA-Toolkit
- **bedtools** (Masks tRNA regions with 50 bp flanks (bedtools maskfasta) when constructing the artificial genome used for the filtering alignment) — https://github.com/skbinfo/tncRNA-Toolkit
- **Trim Galore** (Produces the quality-filtered, adapter-trimmed FASTQ input that feeds the mapping and multi-map filtering step) — https://github.com/FelixKrueger/TrimGalore

## Examples

```
python3 tncRNAs.py -s <species name> -i <processed small RNA reads>.fastq -o <output dir>
```

## Evaluation signals

- Output catalog contains only reads mapping exclusively to the artificial chromosomes (masked tRNA plus 50 nt flanks); no reads map to non-tRNA genomic regions
- No retained read has more than 50 alignments; reads aligning >50 times are discarded per the '-m 50' setting
- Every retained read appears at a single genomic locus (SAM flags 0/16 only, unique mapping) and aligns to the plus strand after '--norc' realignment
- Each classified tncRNA carries a subtype label and an RPM value computed as reads mapped to progenitor tRNA divided by PMF (total mapped reads / 10^6)
- tRNA half classifications respect the 7 nt anticodon-loop cleavage rule (2 nt + 3 nt anticodon + 2 nt)

## Limitations

- Allowing just a single mismatch to overcome sequencing errors may lead to base misidentification and inflation of false negatives; this skill uses '-v 2' instead
- The '-m 50' ceiling arbitrarily discards reads mapping to very highly repeated tRNA families (>50 loci), potentially undercounting tncRNAs from those loci
- Plus-strand-only realignment ('--norc') removes reverse-strand signal, so antisense tRNA-derived reads are not recovered

## Evidence
