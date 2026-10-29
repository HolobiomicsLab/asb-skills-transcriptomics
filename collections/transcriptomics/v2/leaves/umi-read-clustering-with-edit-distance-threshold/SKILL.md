---
name: umi-read-clustering-with-edit-distance-threshold
description: Use when processing UMI-containing single-cell RNA-Seq FASTQ/SAM data
  (e.g. SCRB-Seq, CEL-Seq2, tagmentation libraries) and you need to collapse duplicate
  UMI reads at the same genomic position into molecule counts before gene-level quantification.
when_to_use_negative:
- Input reads lack UMIs — the artefact and the collapse step are undetectable/meaningless
  without UMI
- The dataset is bulk RNA-Seq without molecular identifiers, where standard duplicate-marking
  deduplication (e.g. Picard MarkDuplicates) is the appropriate approach
- You need to quantify spliced transcripts — the paper's STAR run specifies no introns,
  so this pipeline is not set up for splice-aware alignment
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3170
tools:
- name: fastq_qual_filter
  role: C program to quality-filter FASTQ reads on a chosen substring (min_ok threshold,
    0-based offset, length), e.g. stricter threshold on the UMI itself
  repo: https://github.com/ncgr/UMI-analysis
- name: fastq_umi_clipper
  role: C program to move the 5' UMI into the FASTQ header and optionally erase protocol-derived
    trailing Gs (Clontech SMARTer) before alignment
  repo: https://github.com/ncgr/UMI-analysis
- name: STAR
  role: Aligns clipped reads to the genomic reference specifying no introns; output
    must be a sorted SAM file
  repo: https://github.com/alexdobin/STAR
- name: umi_counterizer.pl
  role: Perl script that identifies UMI duplicates at the same mapping position and
    collapses duplicates pairwise into the most abundant UMI, using max_edit_distance
    (3 bp) with Text::Levenshtein::XS
  repo: https://github.com/ncgr/UMI-analysis
- name: umi_analyzer_uthash
  role: C project using uthash macros to find all genomic positions where each UMI
    maps, producing umi_positions.txt
  repo: https://github.com/ncgr/UMI-analysis
- name: umi_clusterizer.pl
  role: Counts unique UMI at each genomic position where a UMI is found
  repo: https://github.com/ncgr/UMI-analysis
- name: bedtools
  role: Optionally intersects position_umi_counts.bed with a GFF file (filtered via
    pcregrep '\tgene\t') for gene-level behaviour confirmation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:820977a0e3e71e90dc1dbec702370ba97e2acd2310c69dbde0da53b3370c2f56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncgr/UMI-analysis
  tools:
  - C
  - STAR
  - Text::Levenshtein::XS
  - Try::Tiny
  - bedtools
  - fastq_qual_filter
  - fastq_umi_clipper
  - pcregrep
  - umi_analyzer_uthash
  - umi_clusterizer.pl
  - umi_counterizer.pl
  - uthash
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# umi-read-clustering-with-edit-distance-threshold

## Summary

Collapse sequencing reads sharing a UMI into single mRNA-molecule observations by grouping UMIs at the same genomic mapping position that are within a configurable Levenshtein edit-distance threshold (3 bp in the paper), revealing the mapping-shift artefact where identical-origin reads map to closely spaced locations.

## When to use

Use when processing UMI-containing single-cell RNA-Seq FASTQ/SAM data (e.g. SCRB-Seq, CEL-Seq2, tagmentation libraries) and you need to collapse duplicate UMI reads at the same genomic position into molecule counts before gene-level quantification.

## When NOT to use

- Input reads lack UMIs — the artefact and the collapse step are undetectable/meaningless without UMI
- The dataset is bulk RNA-Seq without molecular identifiers, where standard duplicate-marking deduplication (e.g. Picard MarkDuplicates) is the appropriate approach
- You need to quantify spliced transcripts — the paper's STAR run specifies no introns, so this pipeline is not set up for splice-aware alignment

## Inputs

- Demultiplexed single-end FASTQ files containing 5' UMI sequences
- Sorted SAM files from STAR alignment (no introns specified)
- GFF gene annotation file for optional gene-level intersection

## Outputs

- all_sam_umis.txt (every UMI observed in the SAM file, sorted)
- collapsed_sam_umis.txt (non-redundant list of UMIs, sorted)
- umi_positions.txt (each UMI and the genomic positions where it maps)
- position_umi_counts.bed (UMI counts at each genomic position)
- umi_bed_intersection_gff.txt (UMI positions intersected with gene annotations)

## How to apply

First quality-filter the FASTQ with fastq_qual_filter (e.g. min_ok 30 over the first 10 bases covering the UMI substring), then clip the 5' UMI with fastq_umi_clipper (specifying clip_length and any protocol-derived trailing Gs, e.g. g_length for Clontech SMARTer libraries). Align the clipped reads with STAR specifying no introns and produce a sorted SAM file. Run umi_counterizer.pl on the sorted SAM with max_edit_distance=3 so that UMIs at the same mapping position within 3 bp edit distance are collapsed pairwise into the most abundant UMI; it logs all_sam_umis.txt and collapsed_sam_umis.txt. Then map each UMI to its genomic positions with the C project src/c/umi_analyzer_uthash (after editing exe.c to set UMI length, position-string length, and max line length) and count unique UMIs per position with umi_clusterizer.pl into position_umi_counts.bed. The rationale: reads with different mapping coordinates may derive from the same starting molecule, so naive position-based deduplication over-counts transcripts; grouping clusters of closely-spaced reads as single observations accounts for this.

## Related tools

- **fastq_qual_filter** (C program to quality-filter FASTQ reads on a chosen substring (min_ok threshold, 0-based offset, length), e.g. stricter threshold on the UMI itself) — https://github.com/ncgr/UMI-analysis
- **fastq_umi_clipper** (C program to move the 5' UMI into the FASTQ header and optionally erase protocol-derived trailing Gs (Clontech SMARTer) before alignment) — https://github.com/ncgr/UMI-analysis
- **STAR** (Aligns clipped reads to the genomic reference specifying no introns; output must be a sorted SAM file) — https://github.com/alexdobin/STAR
- **umi_counterizer.pl** (Perl script that identifies UMI duplicates at the same mapping position and collapses duplicates pairwise into the most abundant UMI, using max_edit_distance (3 bp) with Text::Levenshtein::XS) — https://github.com/ncgr/UMI-analysis
- **umi_analyzer_uthash** (C project using uthash macros to find all genomic positions where each UMI maps, producing umi_positions.txt) — https://github.com/ncgr/UMI-analysis
- **umi_clusterizer.pl** (Counts unique UMI at each genomic position where a UMI is found) — https://github.com/ncgr/UMI-analysis
- **bedtools** (Optionally intersects position_umi_counts.bed with a GFF file (filtered via pcregrep '\tgene\t') for gene-level behaviour confirmation)

## Examples

```
umi_counterizer.pl sorted_samfile 3
```

## Evaluation signals

- collapsed_sam_umis.txt contains a strictly non-redundant, sorted subset of all_sam_umis.txt
- At each position in position_umi_counts.bed, no two retained UMIs exceed the max_edit_distance (3 bp) from each other while collapsed pairs were merged into the most abundant UMI
- Reproduced cluster-size read fractions match published values: ~19% of reads in size-one clusters for run_171108 and ~75% for the La Manno data set
- Mapping-shift density distributions of pooled UMI-read clusters peak at zero shift with a bell-shaped decay (Figs. 2-4 equivalents)
- Gene-level bedtools intersection with pcregrep '\tgene\t' yields counts consistent with position-level UMI counts

## Limitations

- The 3 bp collapse threshold is an arbitrary configurable parameter; the true distribution of mapping shift sizes is unknown and reads derived from the same template could have larger shifts
- Not finished software: usefulness is limited unless the user can read and edit C and Perl; makefiles must be edited, libraries installed (bstring in LD_LIBRARY_PATH, uthash.h), and paths set
- Little argument checking: e.g. substring longer than read length or negative values cause unexpected behavior, and output files are overwritten without warning
- Only single-ended sequencing is covered by the authors' use case
- Contribution of PCR stutter from simple tandem repeats cannot be concluded because shifts also occur with normal-entropy reads
- umi_analyzer_uthash has no input flexibility or much error checking and makes strict assumptions about space-delimited structure of the umi_counterizer.pl output

## Evidence

- [discussion] we considered reads to be derived from the same starting mRNA if they share a UMI and the mapping locations are separated by three or fewer base pairs: "we considered reads to be derived from the same starting mRNA if they share a UMI and the mapping locations are separated by three or fewer base pairs"
- [readme] umi_counterizer.pl collapse rule: [evidence span withheld — anchor status: external_doc]
- [readme] Collapse duplicates pairwise into most abundant UMI: "Identify UMI duplicates at the same position. 
   * Collapse duplicates pairwise into most abundant UMI"
- [abstract] reads sharing a UMI map to closely spaced locations: "reads sharing a UMI, and therefore presumed to be derived from the same mRNA molecule, frequently map to different, but closely spaced locations."
- [discussion] Our pipeline groups closely-spaced read clusters: "We are making available our analysis pipeline, which groups clusters of closely-spaced reads together as single observations"
- [discussion] ART parameter is configurable: "reads derived from the same template could have larger mapping shifts, the true distribution being unknown. Accordingly, we make this separation a configurable parameter"
