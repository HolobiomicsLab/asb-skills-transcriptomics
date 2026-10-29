---
name: replicate-reproducibility-correlation-analysis
description: Use when two or more biological RNA-seq replicates are available and
  you need to verify that novel IPA sites and assembled IPA isoforms detected per-replicate
  are consistent (overlapping sites, correlated expression).
when_to_use_negative:
- Only a single RNA-seq sample is available — replicate overlap and correlation cannot
  be computed
- The goal is differential IPA usage between conditions (e.g. control vs LPS-stimulated)
  rather than within-condition replicate concordance
- Data are 3'-end sequencing (3P-seq, polyA-seq) benchmarks rather than conventional
  RNA-seq from biological replicates
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0203
tools:
- name: InPACT
  role: Per-replicate IPA site prediction (InPACT), novel IPA isoform assembly (InPACT_transcript),
    and IPA usage quantification (InPACT_quantify)
  repo: github.com/YY-TMU/InPACT
- name: HISAT2
  role: Aligns each replicate's reads to the GRCh38 RefSeq-based genome index before
    IPA detection
- name: SAMtools
  role: Filters secondary alignments and coordinate-sorts the per-replicate BAM files
- name: gffread
  role: Extracts transcript sequences from the assembled merged.gtf for Salmon indexing
  repo: http://cole-trapnell-lab.github.io/cufflinks/assets/downloads/cufflinks-2.2.1.Linux_x86_64.tar.gz
- name: Salmon
  role: Indexes (k=31) and quantifies novel plus annotated transcripts per replicate
    to yield quant.sf for IPA usage
  repo: github.com/COMBINE-lab/salmon
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-46875-8
    title: 'InPACT: a computational method for accurate characterization of intronic
      polyadenylation from RNA sequencing data'
  source_content_hash: sha256:be946968387f495cbab5bf478267e237bac8730e4ff5902d2126b54578f25b29
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - HISAT2
  - InPACT
  - SAMtools
  - Salmon
  - gffread
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# replicate-reproducibility-correlation-analysis

## Summary

Assesses whether InPACT-identified novel IPA isoforms show reproducible expression between biological RNA-seq replicates by independently detecting IPA sites, assembling isoforms, quantifying transcripts with Salmon, and computing the Pearson correlation of novel isoform expression between replicates.

## When to use

Use when two or more biological RNA-seq replicates are available and you need to verify that novel IPA sites and assembled IPA isoforms detected per-replicate are consistent (overlapping sites, correlated expression).

## When NOT to use

- Only a single RNA-seq sample is available — replicate overlap and correlation cannot be computed
- The goal is differential IPA usage between conditions (e.g. control vs LPS-stimulated) rather than within-condition replicate concordance
- Data are 3'-end sequencing (3P-seq, polyA-seq) benchmarks rather than conventional RNA-seq from biological replicates

## Inputs

- Two (or more) biological RNA-seq replicate read sets (FASTQ/SRA, e.g. HEK293 replicates)
- GRCh38 reference genome FASTA and HISAT2 RefSeq-based genome index
- RefSeq annotation GTF (hg38.109.20211119.ncbiRefSeq.gtf)
- Precomputed candidate polyA site file (InPACT_polyAsites.hg38.saf)

## Outputs

- Per-replicate InPACT predict.result.txt (IPA site predictions with skipped/composite type)
- Per-replicate merged.gtf with assembled novel IPA isoforms
- Per-replicate Salmon quant.sf transcript abundances and ipa_usage.txt IPA usage estimates
- Novel IPA site counts per replicate, inter-replicate site overlap, and Pearson correlation coefficient of novel isoform expression between replicates

## How to apply

Run the full InPACT workflow independently on each replicate: align reads to GRCh38 with HISAT2, remove secondary alignments with SAMtools and sort the BAMs, then run InPACT against the RefSeq GTF (hg38.109.20211119.ncbiRefSeq.gtf) with the precomputed InPACT_polyAsites.hg38.saf candidate file, -P 5 threads, and default read thresholds (>=5 spliced reads for skipped, >=10 boundary-crossing reads for composite terminal exons). Assemble novel IPA isoforms per replicate with InPACT_transcript, extract transcript sequences with gffread, build a Salmon index with -k 31, quantify each replicate with salmon quant --validateMappings, and compute per-replicate IPA usage with InPACT_quantify. Finally, count novel IPA sites per replicate with skipped/composite breakdown, compute the overlap of novel sites between replicates, and calculate the Pearson correlation of novel IPA isoform expression across replicates; correct application is confirmed when replicate concordance matches the published reference (471 and 393 novel IPA sites, ~218 overlapping sites, Pearson r = 0.94, P < 0.05).

## Related tools

- **InPACT** (Per-replicate IPA site prediction (InPACT), novel IPA isoform assembly (InPACT_transcript), and IPA usage quantification (InPACT_quantify)) — github.com/YY-TMU/InPACT
- **HISAT2** (Aligns each replicate's reads to the GRCh38 RefSeq-based genome index before IPA detection)
- **SAMtools** (Filters secondary alignments and coordinate-sorts the per-replicate BAM files)
- **gffread** (Extracts transcript sequences from the assembled merged.gtf for Salmon indexing) — http://cole-trapnell-lab.github.io/cufflinks/assets/downloads/cufflinks-2.2.1.Linux_x86_64.tar.gz
- **Salmon** (Indexes (k=31) and quantifies novel plus annotated transcripts per replicate to yield quant.sf for IPA usage) — github.com/COMBINE-lab/salmon

## Examples

```
InPACT -i rep1.sorted.bam -a hg38.109.20211119.ncbiRefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5 && InPACT_transcript --predict_terminal predict.result.txt --annotated_gtf RefSeq.gtf --fa_path genome.fa --save_gtf merged.gtf
```

## Evaluation signals

- Per-replicate novel IPA site counts should reproduce the published values of 471 and 393 sites
- Inter-replicate overlap of novel IPA sites should be approximately 218 sites
- Pearson correlation of novel IPA isoform expression between the two replicates should reach r = 0.94 with P < 0.05
- Each replicate's BAM must be coordinate-sorted with secondary alignments removed before InPACT runs
- predict.result.txt per replicate should contain IPA type assignments (skipped vs composite) and PAU usage estimates compatible with InPACT_quantify input

## Limitations

- Default InPACT read thresholds (>=5 spliced reads for skipped, >=10 boundary-crossing reads for composite) govern sensitivity; shallow sequencing may miss novel IPA sites in one replicate and deflate overlap
- InPACT run time is substantial (~1 hour for the test file on five cores), so per-replicate reruns are computationally non-trivial
- No version number is given for InPACT in the article, so exact reproducibility across tool versions is not guaranteed

## Evidence

- [other] the expression levels of novel assembled IPA isoforms are robustly correlated between the two HEK293 RNA-seq replicates (Pearson's correlation r = 0.94, P < 0.05): [evidence span withheld — anchor status: not_found]
- [methods] Reads flagged as secondary alignment were filtered out using SAMtools: "Reads flagged as secondary alignment were filtered out using SAMtools"
- [readme] Minimum number of spliced reads required to consider the potential skipped terminal exon. [default=5]: "Minimum number of spliced reads required to consider the potential skipped terminal exon. [default=5]"
- [methods] InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5: "InPACT -i sample.bam -a RefSeq.gtf -s InPACT_polyAsites.hg38.saf -P 5"
- [methods] salmon quant -i transcripts_index -l <LIBTYPE> -r reads.fq --validateMappings -o transcripts_quant: "salmon quant -i transcripts_index -l <LIBTYPE> -r reads.fq --validateMappings -o transcripts_quant"
