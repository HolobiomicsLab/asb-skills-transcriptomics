---
name: cross-aligner-density-plot-visualization
description: Use when you need to test whether a UMI-read mapping-shift pattern is
  specific to one alignment algorithm by re-aligning the same quality-filtered, UMI-clipped
  FASTQ reads with STAR, GSNAP and HISAT2 and overlaying the resulting shift-distance
  density curves.
when_to_use_negative:
- Input reads have not been quality-filtered and UMI-clipped (fastq_qual_filter /
  fastq_umi_clipper run first), since umi_counterizer.pl assumes the UMI is in the
  FASTQ/SAM header
- Paired-end sequencing data — the ncgr/UMI-analysis pipeline's use case 'only involves
  single ended sequencing at this stage'
- You want to quantify gene-level expression rather than characterize the shift artefact;
  use the gene_umi_counter.pl / read-count branch of the pipeline instead
edam_operation: http://edamontology.org/operation_3431
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: STAR
  role: First aligner run on the UMI-clipped FASTQ reads against the P. patens reference,
    specifying no introns, to produce one sorted SAM input
  repo: https://github.com/alexdobin/STAR
- name: GSNAP
  role: Second aligner used to independently align the same reads for cross-aligner
    comparison
- name: HISAT2
  role: Third aligner used to independently align the same reads for cross-aligner
    comparison
- name: umi_counterizer.pl
  role: Perl script run on each sorted SAM file to identify and pairwise-collapse
    UMI duplicates at the same genomic position, logging all_sam_umis.txt and collapsed_sam_umis.txt
  repo: https://github.com/ncgr/UMI-analysis
- name: umi_analyzer_uthash
  role: C program using uthash preprocessor macros to list all genomic positions where
    each UMI maps (umi_positions.txt)
  repo: https://github.com/ncgr/UMI-analysis
- name: umi_clusterizer.pl
  role: Perl script run on umi_positions.txt to count unique UMIs per genomic position
    (position_umi_counts.bed)
  repo: https://github.com/ncgr/UMI-analysis
- name: Text::Levenshtein::XS
  role: Perl package required by umi_counterizer.pl for computing edit distances between
    UMIs during duplicate collapsing
- name: Better String Library
  role: bstring shared object required in LD_LIBRARY_PATH by the C tools (fastq_qual_filter,
    fastq_umi_clipper) used to prepare the reads before alignment
  repo: https://github.com/msteinert/bstring
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:ecc45af765a89918e9e14437eeb0684ee8ef6fe1e71431b1d8c6aed27d605701
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
  edam_operation: http://edamontology.org/operation_3431
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# cross-aligner-density-plot-visualization

## Summary

Reproduce the UMI-read mapping-shift distributions under three independent aligners (STAR, GSNAP, HISAT2) and plot them as overlapping density curves, demonstrating (as in Fig. 5 of Sena et al. 2018) that the mapping-shift artefact is aligner-independent rather than an artefact of one algorithm.

## When to use

Use when you need to test whether a UMI-read mapping-shift pattern is specific to one alignment algorithm by re-aligning the same quality-filtered, UMI-clipped FASTQ reads with STAR, GSNAP and HISAT2 and overlaying the resulting shift-distance density curves.

## When NOT to use

- Input reads have not been quality-filtered and UMI-clipped (fastq_qual_filter / fastq_umi_clipper run first), since umi_counterizer.pl assumes the UMI is in the FASTQ/SAM header
- Paired-end sequencing data — the ncgr/UMI-analysis pipeline's use case 'only involves single ended sequencing at this stage'
- You want to quantify gene-level expression rather than characterize the shift artefact; use the gene_umi_counter.pl / read-count branch of the pipeline instead

## Inputs

- Quality-filtered, UMI-clipped single-end FASTQ reads (e.g. run_170420 P. patens reads)
- P. patens genomic reference sequence (plus aligner index files for STAR, GSNAP, HISAT2)
- Sorted SAM file from each aligner run

## Outputs

- Sorted SAM file per aligner (STAR, GSNAP, HISAT2)
- all_sam_umis.txt and collapsed_sam_umis.txt logs per alignment
- umi_positions.txt per alignment (each UMI and its genomic mapping positions)
- position_umi_counts.bed (unique UMIs per genomic position per alignment)
- Three overlapping density curves of UMI-read mapping-shift distances (Fig. 5 equivalent)

## How to apply

Take the quality-filtered, UMI-clipped run_170420 Physcomitrella patens FASTQ reads and align them to the P. patens genomic reference three times — with STAR specifying no introns, with GSNAP, and with HISAT2 — each producing a sorted SAM file. Run the Perl script umi_counterizer.pl on each sorted SAM file with the article's max_edit_distance setting to collapse UMI duplicates at the same position pairwise into the most abundant UMI, emitting all_sam_umis.txt and collapsed_sam_umis.txt per alignment. Compile and run the C program umi_analyzer_uthash (uthash preprocessor macros; edit exe.c to set UMI length and maximum position-string length) on each UMI-counts file to emit umi_positions.txt, then run umi_clusterizer.pl to produce position_umi_counts.bed of unique UMIs per genomic position. For each alignment compute mapping-shift distances between reads sharing a UMI, treating reads separated by three or fewer base pairs as derived from the same starting mRNA (threshold configurable since the true distribution is unknown), and plot the three shift-distance distributions as overlapping density curves.

## Related tools

- **STAR** (First aligner run on the UMI-clipped FASTQ reads against the P. patens reference, specifying no introns, to produce one sorted SAM input) — https://github.com/alexdobin/STAR
- **GSNAP** (Second aligner used to independently align the same reads for cross-aligner comparison)
- **HISAT2** (Third aligner used to independently align the same reads for cross-aligner comparison)
- **umi_counterizer.pl** (Perl script run on each sorted SAM file to identify and pairwise-collapse UMI duplicates at the same genomic position, logging all_sam_umis.txt and collapsed_sam_umis.txt) — https://github.com/ncgr/UMI-analysis
- **umi_analyzer_uthash** (C program using uthash preprocessor macros to list all genomic positions where each UMI maps (umi_positions.txt)) — https://github.com/ncgr/UMI-analysis
- **umi_clusterizer.pl** (Perl script run on umi_positions.txt to count unique UMIs per genomic position (position_umi_counts.bed)) — https://github.com/ncgr/UMI-analysis
- **Text::Levenshtein::XS** (Perl package required by umi_counterizer.pl for computing edit distances between UMIs during duplicate collapsing)
- **Better String Library** (bstring shared object required in LD_LIBRARY_PATH by the C tools (fastq_qual_filter, fastq_umi_clipper) used to prepare the reads before alignment) — https://github.com/msteinert/bstring

## Examples

```
perl umi_counterizer.pl aligned_STAR.sorted.sam 1 > umi_counts.txt
```

## Evaluation signals

- The three density curves of UMI-read mapping-shift distances from STAR, GSNAP and HISAT2 overlap, each showing a pronounced mode with reads mapping to either side — reproducing Fig. 5's aligner-independent pattern
- Each aligner run yields the expected intermediate files: all_sam_umis.txt, collapsed_sam_umis.txt, umi_positions.txt and position_umi_counts.bed from the umi_counterizer.pl → umi_analyzer_uthash → umi_clusterizer.pl chain
- The position_umi_counts.bed file reports, per position, the number of unique UMIs in the 'position, number of UMI, UMIs' structure expected from umi_counterizer.pl output
- Shift distances computed with the ≤3 bp same-mRNA threshold are consistent in scale across all three aligners; changing the configurable separation threshold shifts but does not reorder the distributions
- umi_analyzer_uthash output lines pair each UMI with one or more chr:position strings (e.g. 'TGTAAATTGG Chr01:76001 Chr01:76002'), with multi-position UMIs present for shift computation

## Limitations

- The ≤3 bp separation threshold is arbitrary and configurable because the true distribution of mapping shift sizes is unknown; reads derived from the same template could have larger shifts
- No version numbers are given for STAR, GSNAP or HISAT2, and the C/Perl tools have 'not much arg checking... so unexpected behavior may be encountered' (e.g. negative values, substrings longer than read length, silent overwriting of output files)
- umi_analyzer_uthash has no input flexibility and little error checking; exe.c must be edited to specify UMI length, maximum position-string length and maximum expected line length before compiling
- The pipeline assumes single-ended, demultiplexed FASTQ input; conclusions are drawn for the P. patens run_170420 library and may not transfer to other library preparations

## Evidence

- [methods] STAR aligner with no introns: "We use [STAR](https://github.com/alexdobin/STAR) specifying no introns. To get to the next step you need a sorted SAM file."
- [discussion] ≤3 bp same-mRNA threshold: "we considered reads to be derived from the same starting mRNA if they share a UMI and the mapping locations are separated by three or fewer base pairs"
- [discussion] threshold configurable: "reads derived from the same template could have larger mapping shifts, the true distribution being unknown. Accordingly, we make this separation a configurable parameter"
- [readme] umi_counterizer.pl usage: "Run the Perl script umi\_counterizer.pl on your sorted SAM file. This requires the packages Text::Levenshtein::XS"
- [readme] collapse duplicates: "Identify UMI duplicates at the same position. 
   * Collapse duplicates pairwise into most abundant UMI"
- [readme] uthash requirement: "One of these is [uthash](http://troydhanson.github.io/uthash/userguide.html), which is implemented entirely as preprocessor macros. So put uthash.h somewhere sensible"
