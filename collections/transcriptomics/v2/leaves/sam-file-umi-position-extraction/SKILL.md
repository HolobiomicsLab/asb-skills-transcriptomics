---
name: sam-file-umi-position-extraction
description: Use when UMI-containing FASTQ reads have been aligned to a genomic reference
  (e.g. STAR with no introns) yielding a sorted SAM file, and you need each UMI's
  complete list of genomic mapping positions to measure mapping-shift distances between
  reads sharing a UMI.
when_to_use_negative:
- Your SAM file is unsorted — the pipeline explicitly requires 'a sorted SAM file'
  and umi_analyzer_uthash makes rigid assumptions about space-delimited input line
  structure with little error checking
- Your sequencing data contain no UMIs — the article notes 'In the absence of UMI
  this artefact is undetectable', so per-UMI position extraction is meaningless
- Your libraries are paired-end — the README states 'Our use case only involves single
  ended sequencing at this stage'
tools:
- name: umi_counterizer.pl
  role: Perl script run on the sorted SAM file to identify UMI duplicates at the same
    genomic position, collapse duplicates pairwise into the most abundant UMI, and
    count UMIs per position
  repo: https://github.com/ncgr/UMI-analysis
- name: umi_analyzer_uthash
  role: C program using uthash preprocessor macros to hash UMI-to-position records
    from the counterizer output and emit umi_positions.txt listing every genomic position
    where each UMI maps
  repo: https://github.com/ncgr/UMI-analysis
- name: Text::Levenshtein::XS
  role: Perl package providing fast edit-distance computation used by umi_counterizer.pl
    to compare UMIs against max_edit_distance
- name: uthash
  role: Header-only C hash implemented entirely as preprocessor macros, enabling fast
    UMI-to-positions storage without linking a library
  repo: http://troydhanson.github.io/uthash/userguide.html
- name: Better String Library
  role: C string library whose shared object must be in LD_LIBRARY_PATH for the pipeline's
    compiled C components
  repo: https://github.com/msteinert/bstring
- name: STAR
  role: Aligner used (specifying no introns) to produce the sorted SAM file of UMI-clipped
    reads that this skill consumes
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:c3b2eaf1d7785975a97a78bcfd9bd98e71abc483f883a7ca819f8704a783683d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncgr/UMI-analysis
  tools:
  - Better String Library
  - C
  - GSNAP
  - HISAT2
  - STAR
  - Text::Levenshtein::XS
  - Try::Tiny
  - umi_analyzer_uthash
  - umi_counterizer.pl
  - uthash
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# sam-file-umi-position-extraction

## Summary

Extract the full set of genomic mapping positions for every UMI from a sorted SAM alignment, the prerequisite step for quantifying the UMI-read mapping shift artefact. It converts position-UMI counts into per-UMI position lists so reads sharing a UMI can be compared coordinate-by-coordinate.

## When to use

Use when UMI-containing FASTQ reads have been aligned to a genomic reference (e.g. STAR with no introns) yielding a sorted SAM file, and you need each UMI's complete list of genomic mapping positions to measure mapping-shift distances between reads sharing a UMI.

## When NOT to use

- Your SAM file is unsorted — the pipeline explicitly requires 'a sorted SAM file' and umi_analyzer_uthash makes rigid assumptions about space-delimited input line structure with little error checking
- Your sequencing data contain no UMIs — the article notes 'In the absence of UMI this artefact is undetectable', so per-UMI position extraction is meaningless
- Your libraries are paired-end — the README states 'Our use case only involves single ended sequencing at this stage'

## Inputs

- Sorted SAM file from aligning quality-filtered, UMI-clipped FASTQ reads to the P. patens genomic reference (e.g. STAR, GSNAP, or HISAT2 output)
- max_edit_distance parameter for umi_counterizer.pl
- umi_counterizer.pl output (position, number of UMI, space-delimited UMIs) as input to umi_analyzer_uthash
- uthash.h header and compiled fastq_umi_clipper/umi_analyzer_uthash C projects (bstring shared object in LD_LIBRARY_PATH)

## Outputs

- all_sam_umis.txt (every UMI observed in the SAM file, sorted)
- collapsed_sam_umis.txt (non-redundant list of UMI found in the SAM file, sorted)
- umi_positions.txt (each UMI and its genomic mapping positions, e.g. 'TGTAAATTGG Chr01:76001 Chr01:76002')
- position_umi_counts.bed (unique UMI count per genomic position)

## How to apply

Run the Perl script umi_counterizer.pl on the sorted SAM file with the article's max_edit_distance setting: it identifies UMI duplicates at the same genomic position, collapses duplicates pairwise into the most abundant UMI (UMIs separated by more than max_edit_distance at the same position are kept distinct), and logs all_sam_umis.txt and collapsed_sam_umis.txt. Compile and run the C program umi_analyzer_uthash on the UMI-counts output, which uses uthash preprocessor macros for hash-based lookup because scripted alternatives are far too slow on large SAM-derived files; before compiling you must edit exe.c to specify the UMI length, the maximum position-string length in bytes, and the maximum expected line length. Finally run umi_clusterizer.pl on the resulting umi_positions.txt to produce position_umi_counts.bed counting unique UMIs per genomic position. This position inventory is then used to compute mapping-shift distances, treating reads sharing a UMI whose positions are separated by three or fewer base pairs as derived from the same starting mRNA.

## Related tools

- **umi_counterizer.pl** (Perl script run on the sorted SAM file to identify UMI duplicates at the same genomic position, collapse duplicates pairwise into the most abundant UMI, and count UMIs per position) — https://github.com/ncgr/UMI-analysis
- **umi_analyzer_uthash** (C program using uthash preprocessor macros to hash UMI-to-position records from the counterizer output and emit umi_positions.txt listing every genomic position where each UMI maps) — https://github.com/ncgr/UMI-analysis
- **Text::Levenshtein::XS** (Perl package providing fast edit-distance computation used by umi_counterizer.pl to compare UMIs against max_edit_distance)
- **uthash** (Header-only C hash implemented entirely as preprocessor macros, enabling fast UMI-to-positions storage without linking a library) — http://troydhanson.github.io/uthash/userguide.html
- **Better String Library** (C string library whose shared object must be in LD_LIBRARY_PATH for the pipeline's compiled C components) — https://github.com/msteinert/bstring
- **STAR** (Aligner used (specifying no introns) to produce the sorted SAM file of UMI-clipped reads that this skill consumes) — https://github.com/alexdobin/STAR

## Examples

```
umi_counterizer.pl sorted_samfile max_edit_distance
```

## Evaluation signals

- umi_counterizer.pl output lines follow the format position, number of UMI, space-delimited UMIs (e.g. 'Chr01:219196 6 TGATGGAGGG TCAACCGGGT ...')
- umi_positions.txt contains one line per UMI with all its genomic positions, e.g. 'TGTAAATTGG Chr01:76001 Chr01:76002'; a UMI observed at multiple coordinates appears on a single line with multiple positions
- all_sam_umis.txt and collapsed_sam_umis.txt are produced and sorted; collapsed_sam_umis.txt is a non-redundant subset of all_sam_umis.txt
- UMIs at the same mapping position separated by more than max_edit_distance remain distinct (not collapsed) in the output
- Downstream, repeating the extraction over STAR, GSNAP and HISAT2 alignments should yield overlapping mapping-shift density distributions, reproducing Fig. 5's aligner-independent pattern

## Limitations

- umi_analyzer_uthash has 'no flexibility here, and not much error checking' and makes hard assumptions about input structure — you must edit exe.c with the correct UMI length, maximum position-string length in bytes, and maximum line length before compiling
- The 3 bp separation threshold for treating reads as derived from the same starting mRNA is an arbitrary configurable parameter since 'reads derived from the same template could have larger mapping shifts, the true distribution being unknown'
- The pipeline's Perl/C programs perform minimal argument checking; unexpected behavior may be encountered and output files are overwritten without warning

## Evidence

- [methods] Run the Perl script umi_counterizer.pl on your sorted SAM file.: [evidence span withheld — anchor status: not_found]
- [methods] Identify UMI duplicates at the same position, collapse into most abundant UMI: "Identify UMI duplicates at the same position. 
   * Collapse duplicates pairwise into most abundant UMI"
- [readme] UMIs beyond max_edit_distance at same position are kept distinct: [evidence span withheld — anchor status: external_doc]
- [readme] umi_analyzer_uthash requires editing exe.c constants before compiling: "You need to edit exe.c before compiling. Specify the length of the UMI and the maximum length of the position string in bytes."
- [discussion] Reads sharing a UMI separated by ≤3 bp are from the same starting mRNA: "we considered reads to be derived from the same starting mRNA if they share a UMI and the mapping locations are separated by three or fewer base pairs"
