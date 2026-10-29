---
name: mapping-shift-distance-quantification
description: Use when quality-filtered, UMI-clipped FASTQ reads have been aligned
  with multiple aligners and you need to test whether reads sharing a UMI map to closely
  spaced, distinct genomic coordinates (the mapping shift artefact).
when_to_use_negative:
- Data contain no UMIs — the artefact is undetectable without UMI, so shift-distance
  quantification is meaningless
- Goal is standard gene-level expression counting from a single aligner rather than
  characterising within-UMI positional spread
- Reads are paired-end or the pipeline assumes single-ended sequencing not supported
  by the ncgr/UMI-analysis scripts
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3511
tools:
- name: STAR
  role: Align UMI-clipped reads to the P. patens reference specifying no introns;
    one of three aligners tested for shift-pattern invariance
  repo: https://github.com/alexdobin/STAR
- name: GSNAP
  role: Alternative aligner run on the same FASTQ to confirm the shift distribution
    is not aligner-specific
- name: HISAT2
  role: Third aligner whose sorted SAM should yield the same UMI-read shift pattern
- name: umi_counterizer.pl
  role: Perl script identifying UMI duplicates at the same position and collapsing
    them pairwise into the most abundant UMI from sorted SAM files
- name: umi_analyzer_uthash
  role: C program using uthash preprocessor macros to emit umi_positions.txt listing
    each UMI and its genomic mapping positions
- name: umi_clusterizer.pl
  role: Perl script producing position_umi_counts.bed counting unique UMIs per genomic
    position
- name: Text::Levenshtein::XS
  role: Perl package used by umi_counterizer.pl for edit-distance comparisons between
    UMIs
- name: uthash
  role: Header-only C hash (preprocessor macros) enabling fast UMI-to-positions tracking
    in umi_analyzer_uthash
  repo: http://troydhanson.github.io/uthash/userguide.html
- name: Better String Library
  role: bstring shared object required in LD_LIBRARY_PATH by the C FASTQ filtering/clipping
    steps feeding this workflow
  repo: https://github.com/msteinert/bstring
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:872045a5329025817226b57c267aa69d2dd6dc2a9a96ac413bbf096491734636
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexdobin/STAR
  tools:
  - Better String Library
  - C
  - GSNAP
  - HISAT2
  - STAR
  - Text::Levenshtein::XS
  - Try::Tiny
  - umi_analyzer_uthash
  - umi_clusterizer.pl
  - umi_counterizer.pl
  - uthash
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3511
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.55
n_claims: 20
---

# mapping-shift-distance-quantification

## Summary

Quantify the UMI-read mapping shift artefact by computing distances between genomic positions of reads sharing the same UMI, and verify via STAR, GSNAP and HISAT2 alignments that the shift distribution is aligner-independent.

## When to use

Use when quality-filtered, UMI-clipped FASTQ reads have been aligned with multiple aligners and you need to test whether reads sharing a UMI map to closely spaced, distinct genomic coordinates (the mapping shift artefact).

## When NOT to use

- Data contain no UMIs — the artefact is undetectable without UMI, so shift-distance quantification is meaningless
- Goal is standard gene-level expression counting from a single aligner rather than characterising within-UMI positional spread
- Reads are paired-end or the pipeline assumes single-ended sequencing not supported by the ncgr/UMI-analysis scripts

## Inputs

- Quality-filtered, UMI-clipped run_170420 FASTQ reads
- Physcomitrella patens genomic reference sequence
- Sorted SAM files from STAR, GSNAP and HISAT2 alignments
- all_sam_umis.txt and collapsed_sam_umis.txt logs from umi_counterizer.pl
- umi_positions.txt per alignment from umi_analyzer_uthash

## Outputs

- position_umi_counts.bed counting unique UMIs per genomic position, per aligner
- Mapping-shift distance distributions between reads sharing a UMI
- Three overlapping density curves of UMI-read shift distributions (STAR, GSNAP, HISAT2), reproducing Fig. 5

## How to apply

Take the quality-filtered, UMI-clipped run_170420 FASTQ reads and align them to the P. patens genomic reference three times using STAR (specifying no introns), GSNAP, and HISAT2, keeping a sorted SAM file per aligner. Run umi_counterizer.pl on each sorted SAM with the article's max_edit_distance setting to collapse UMI duplicates pairwise into the most abundant UMI at the same position, emitting all_sam_umis.txt and collapsed_sam_umis.txt. Compile and run umi_analyzer_uthash (uthash preprocessor macros) on each UMI-counts file to produce umi_positions.txt listing each UMI and all genomic positions it maps to, then run umi_clusterizer.pl to obtain position_umi_counts.bed of unique UMIs per position. For each alignment, compute mapping-shift distances between reads sharing a UMI, classifying reads separated by three or fewer base pairs as derived from the same starting mRNA (threshold configurable). Plot the three shift-distance density curves and check they overlap with a pronounced mode and reads mapping to either side, reproducing Fig. 5.

## Related tools

- **STAR** (Align UMI-clipped reads to the P. patens reference specifying no introns; one of three aligners tested for shift-pattern invariance) — https://github.com/alexdobin/STAR
- **GSNAP** (Alternative aligner run on the same FASTQ to confirm the shift distribution is not aligner-specific)
- **HISAT2** (Third aligner whose sorted SAM should yield the same UMI-read shift pattern)
- **umi_counterizer.pl** (Perl script identifying UMI duplicates at the same position and collapsing them pairwise into the most abundant UMI from sorted SAM files)
- **umi_analyzer_uthash** (C program using uthash preprocessor macros to emit umi_positions.txt listing each UMI and its genomic mapping positions)
- **umi_clusterizer.pl** (Perl script producing position_umi_counts.bed counting unique UMIs per genomic position)
- **Text::Levenshtein::XS** (Perl package used by umi_counterizer.pl for edit-distance comparisons between UMIs)
- **uthash** (Header-only C hash (preprocessor macros) enabling fast UMI-to-positions tracking in umi_analyzer_uthash) — http://troydhanson.github.io/uthash/userguide.html
- **Better String Library** (bstring shared object required in LD_LIBRARY_PATH by the C FASTQ filtering/clipping steps feeding this workflow) — https://github.com/msteinert/bstring

## Examples

```
umi_counterizer.pl sorted_star.sam 1 > umi_counts_star.txt && umi_analyzer_uthash umi_counts_star.txt > umi_positions.txt && umi_clusterizer.pl umi_positions.txt > position_umi_counts.bed
```

## Evaluation signals

- The three density plots of shift distances from STAR, GSNAP and HISAT2 overlap, showing the same basic pattern with a pronounced mode and reads mapping to either side
- Reads sharing a UMI with mapping locations separated by ≤3 bp are classified as derived from the same starting mRNA (configurable threshold applied consistently across aligners)
- umi_counterizer.pl produces sorted all_sam_umis.txt (every UMI observed) and collapsed_sam_umis.txt (non-redundant UMIs) for each of the three alignments
- umi_positions.txt lists each UMI followed by space-delimited positions (e.g. 'TGTAAATTGG Chr01:76001 Chr01:76002'), with UMI length and maximum position-string length specified in exe.c before compiling
- position_umi_counts.bed rows follow the 'position, number of UMI, space-delimited UMIs' format per genomic position

## Limitations

- The 3 bp collapse threshold is an arbitrary configurable parameter; reads derived from the same template could have larger mapping shifts and the true shift-size distribution is unknown
- Not much argument checking in the C and Perl programs — inputs longer than read length or negative values cause unexpected behavior, and output files are overwritten without warning
- umi_analyzer_uthash makes strict assumptions about input structure (space-delimited lines, fixed UMI length, max line length hardcoded in exe.c) with little error checking
- Pipeline use case is single-ended sequencing only at this stage

## Evidence

- [results] All three aligners yield the same basic pattern: [evidence span withheld — anchor status: not_found]
- [discussion] 3 bp same-mRNA criterion: "we considered reads to be derived from the same starting mRNA if they share a UMI and the mapping locations are separated by three or fewer base pairs"
- [methods] Align independently with STAR no introns: "Do this however you want. We use [STAR](https://github.com/alexdobin/STAR) specifying no introns. To get to the next step you need a sorted SAM file."
- [readme] umi_counterizer.pl behavior: "Identify UMI duplicates at the same position. 
   * Collapse duplicates pairwise into most abundant UMI"
- [readme] umi_positions.txt output: "Output is each UMI and where it maps in the genome. This file is usually called umi_positions.txt."
- [discussion] Threshold is configurable: "reads derived from the same template could have larger mapping shifts, the true distribution being unknown. Accordingly, we make this separation a configurable parameter"
