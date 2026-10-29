---
name: umi-read-cluster-collapsing
description: Use when processing quality-filtered, UMI-clipped single-end RNA-Seq
  data (e.g., sorted SAM alignments of P. patens SCRB-Seq reads) where reads sharing
  a UMI must be collapsed to per-molecule counts despite mapping to slightly different
  genomic coordinates.
when_to_use_negative:
- Reads carry no UMI — the artefact is undetectable without UMI and collapsing is
  not meaningful
- Input is already a collapsed per-gene count matrix (e.g., UMI counts per gene after
  gene_umi_counter.pl) rather than alignments
- Paired-end data or non-single-end library layouts the pipeline does not support
  (the README states 'Our use case only involves single ended sequencing at this stage')
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: fastq_qual_filter
  role: C program (requires Better String Library in LD_LIBRARY_PATH) that quality-filters
    FASTQ reads on any substring, allowing a stricter threshold on the UMI than the
    rest of the read
  repo: https://github.com/ncgr/UMI-analysis
- name: fastq_umi_clipper
  role: C program that clips the UMI from the 5' end of each read, places it in the
    FASTQ header replacing the index, and optionally erases trailing Gs from the SMARTer
    template-switching system
  repo: https://github.com/ncgr/UMI-analysis
- name: STAR
  role: Aligner used to map UMI-clipped reads to the P. patens genomic reference specifying
    no introns, producing the sorted SAM required by umi_counterizer.pl
  repo: https://github.com/alexdobin/STAR
- name: umi_counterizer.pl
  role: Perl script (requires Text::Levenshtein::XS and Try::Tiny) that identifies
    UMI duplicates at the same genomic position and collapses duplicates pairwise
    into the most abundant UMI
  repo: https://github.com/ncgr/UMI-analysis
- name: umi_analyzer_uthash
  role: C program using uthash preprocessor macros to find all genomic positions where
    each UMI maps, emitting umi_positions.txt
  repo: https://github.com/ncgr/UMI-analysis
- name: bedtools
  role: Intersects position_umi_counts.bed with a GFF file to find gene annotations
    matching UMI positions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:4b8d8f25afdf0aa435241ae4440c6ff1138b21d805cc952f19457e273270e01a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncgr/UMI-analysis
  tools:
  - C
  - GSNAP
  - HISAT2
  - STAR
  - Text::Levenshtein::XS
  - Try::Tiny
  - bedtools
  - fastq_qual_filter
  - fastq_umi_clipper
  - umi_analyzer_uthash
  - umi_counterizer.pl
  - uthash
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# umi-read-cluster-collapsing

## Summary

Collapse UMI-tagged reads into single molecule observations by clustering UMIs at the same genomic position with an edit-distance threshold, then grouping reads sharing a UMI whose mapping positions are separated by ≤3 bp. This corrects for the UMI mapping-shift artefact that otherwise over-estimates transcript abundance in UMI-based RNA-Seq counting.

## When to use

Use when processing quality-filtered, UMI-clipped single-end RNA-Seq data (e.g., sorted SAM alignments of P. patens SCRB-Seq reads) where reads sharing a UMI must be collapsed to per-molecule counts despite mapping to slightly different genomic coordinates.

## When NOT to use

- Reads carry no UMI — the artefact is undetectable without UMI and collapsing is not meaningful
- Input is already a collapsed per-gene count matrix (e.g., UMI counts per gene after gene_umi_counter.pl) rather than alignments
- Paired-end data or non-single-end library layouts the pipeline does not support (the README states 'Our use case only involves single ended sequencing at this stage')

## Inputs

- Quality-filtered FASTQ file (e.g., from fastq_qual_filter with min_ok threshold on the UMI substring)
- UMI-clipped FASTQ with UMI moved into the FASTQ header (fastq_umi_clipper output)
- Sorted SAM file from an aligner (STAR with no introns, GSNAP, or HISAT2)
- Genomic reference sequence and optional GFF annotation for gene-level aggregation
- umi_counterizer.pl output (position, number of UMI, space-delimited UMIs) for umi_analyzer_uthash

## Outputs

- all_sam_umis.txt (every UMI observed in the SAM file) and collapsed_sam_umis.txt (non-redundant UMI list)
- umi_positions.txt listing each UMI and its genomic mapping positions
- position_umi_counts.bed counting unique UMIs per genomic position
- Mapping-shift distance distributions between reads sharing a UMI (density plots per aligner)

## How to apply

After quality filtering (fastq_qual_filter, e.g., min_ok 30) and UMI clipping (fastq_umi_clipper, which moves the UMI into the FASTQ header), align reads to the genomic reference — the article uses STAR specifying no introns — and produce a sorted SAM file. Run the Perl script umi_counterizer.pl on the sorted SAM with a max_edit_distance argument so that UMIs within the edit-distance threshold at the same mapping position are collapsed pairwise into the most abundant UMI, writing all_sam_umis.txt and collapsed_sam_umis.txt. Compile umi_analyzer_uthash (which requires uthash preprocessor macros, with UMI length and maximum position-string length hard-coded in exe.c) to emit umi_positions.txt listing every UMI and its genomic mapping positions, then run umi_clusterizer.pl to produce position_umi_counts.bed counting unique UMIs per position. Finally compute mapping-shift distances between reads sharing a UMI, treating reads separated by three or fewer base pairs as derived from the same starting mRNA; this 3 bp separation is a configurable parameter because the true shift distribution is unknown.

## Related tools

- **fastq_qual_filter** (C program (requires Better String Library in LD_LIBRARY_PATH) that quality-filters FASTQ reads on any substring, allowing a stricter threshold on the UMI than the rest of the read) — https://github.com/ncgr/UMI-analysis
- **fastq_umi_clipper** (C program that clips the UMI from the 5' end of each read, places it in the FASTQ header replacing the index, and optionally erases trailing Gs from the SMARTer template-switching system) — https://github.com/ncgr/UMI-analysis
- **STAR** (Aligner used to map UMI-clipped reads to the P. patens genomic reference specifying no introns, producing the sorted SAM required by umi_counterizer.pl) — https://github.com/alexdobin/STAR
- **umi_counterizer.pl** (Perl script (requires Text::Levenshtein::XS and Try::Tiny) that identifies UMI duplicates at the same genomic position and collapses duplicates pairwise into the most abundant UMI) — https://github.com/ncgr/UMI-analysis
- **umi_analyzer_uthash** (C program using uthash preprocessor macros to find all genomic positions where each UMI maps, emitting umi_positions.txt) — https://github.com/ncgr/UMI-analysis
- **bedtools** (Intersects position_umi_counts.bed with a GFF file to find gene annotations matching UMI positions)

## Examples

```
umi_counterizer.pl sorted.sam 2 > position_umi_counts.txt
```

## Evaluation signals

- Run the pipeline with three aligners (STAR no introns, GSNAP, HISAT2): the UMI-read mapping-shift distance distributions should overlap, with a pronounced mode and reads mapping to either side, reproducing the aligner-independence result of Fig. 5
- umi_counterizer.pl must produce all_sam_umis.txt (every UMI observed) and collapsed_sam_umis.txt (non-redundant sorted UMI list); at a given position, UMIs separated by more than max_edit_distance must not be collapsed
- umi_positions.txt entries should list a UMI followed by one or more 'Chr:position' tokens (e.g., 'TGTAAATTGG Chr01:76001 Chr01:76002'), with the UMI length matching the exe.c setting
- Collapsed (control) versus uncollapsed (case) replicate comparison should show reduced apparent transcript abundance in collapsed data; uncollapsed counting over-estimates certain transcript abundances
- Positional format checks: umi_counterizer.pl output lines must be 'position<TAB>count<TAB>UMI1 UMI2 ...' (space-delimited UMIs), which umi_analyzer_uthash's strtok_r parsing requires

## Limitations

- The 3 bp separation threshold is an arbitrary, configurable parameter; reads derived from the same template could have larger mapping shifts and the true distribution is unknown
- The contribution of PCR stutter from simple tandem repeats cannot be concluded because mapping shifts also occur with reads of normal or high entropy
- Not finished software: users must read/edit C and Perl, install libraries (Better String Library, uthash, Text::Levenshtein::XS, Try::Tiny), edit makefiles, and set paths; the programs do little argument checking and overwrite outputs without warning
- umi_analyzer_uthash has no flexibility and little error checking — UMI length, maximum position-string length, and maximum line length must be hard-coded in exe.c before compiling
- The pipeline supports only single-end sequencing at this stage

## Evidence

- [readme] UMIs greater than max_edit_distance from each other will be considered different UMI and not collapsed into one: [evidence span withheld — anchor status: external_doc]
- [discussion] reads to be derived from the same starting mRNA if they share a UMI and the mapping locations are separated by three or fewer base pairs: "we considered reads to be derived from the same starting mRNA if they share a UMI and the mapping locations are separated by three or fewer base pairs"
- [methods] Collapse duplicates pairwise into most abundant UMI: "Identify UMI duplicates at the same position. 
   * Collapse duplicates pairwise into most abundant UMI"
- [discussion] reads derived from the same template could have larger mapping shifts, the true distribution being unknown. Accordingly, we make this separation a configurable parameter: "reads derived from the same template could have larger mapping shifts, the true distribution being unknown. Accordingly, we make this separation a configurable parameter"
- [readme] It replaces the index at the end of the FASTQ header with the UMI: "You tell the program how long the UMI is at the start of each read. It replaces the index at the end of the FASTQ header with the UMI."
