---
name: umi-duplicate-collapsing
description: 'Use when counting molecules from UMI-tagged RNA-Seq alignments: a sorted
  SAM file exists and duplicate UMI at identical genomic positions must be merged
  into single observations before per-gene counting.'
when_to_use_negative:
- The FASTQ/UMI clipping step has not yet been performed (fastq_umi_clipper must first
  move the UMI into the read header) or reads are not yet aligned to a sorted SAM
  file
- The goal is read-level rather than molecule-level counting, or the data contain
  no UMIs so the artefact cannot even be detected
- True biological duplicates at identical positions should be retained as separate
  molecules and no edit-distance-based merging is desired
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Perl
  role: Language of umi_counterizer.pl, the script that identifies UMI duplicates
    at the same genomic position and collapses them pairwise into the most abundant
    UMI
- name: Text::Levenshtein::XS
  role: Provides fast Levenshtein edit-distance computation used to decide whether
    two UMIs at the same position are within max_edit_distance and should be merged
- name: Try::Tiny
  role: Perl dependency of umi_counterizer.pl for error handling during pairwise UMI
    comparison
- name: uthash
  role: C preprocessor-macro hash used by the downstream umi_analyzer_uthash step
    that maps each collapsed UMI to its genomic positions (umi_positions.txt)
  repo: http://troydhanson.github.io/uthash/userguide.html
- name: STAR
  role: Produces the sorted SAM file (aligned with no introns) that is the input to
    umi_counterizer.pl
  repo: https://github.com/alexdobin/STAR
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-31064-7
    title: Unique Molecular Identifiers reveal a novel sequencing artefact with implications
      for RNA-Seq based gene expression analysis
  source_content_hash: sha256:5eb83d317201da5abe982cb2bdfb3ad446a651cca3498823d3f0879affa92896
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - C
  - Perl
  - STAR
  - Text::Levenshtein::XS
  - Try::Tiny
  - pcregrep
  - uthash
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# umi-duplicate-collapsing

## Summary

Collapse duplicate UMI at the same genomic position in a sorted SAM file so that reads sharing a UMI (within a configurable Levenshtein edit-distance threshold) count as a single starting mRNA molecule, using the umi_counterizer.pl Perl script from the ncgr/UMI-analysis pipeline.

## When to use

Use when counting molecules from UMI-tagged RNA-Seq alignments: a sorted SAM file exists and duplicate UMI at identical genomic positions must be merged into single observations before per-gene counting.

## When NOT to use

- The FASTQ/UMI clipping step has not yet been performed (fastq_umi_clipper must first move the UMI into the read header) or reads are not yet aligned to a sorted SAM file
- The goal is read-level rather than molecule-level counting, or the data contain no UMIs so the artefact cannot even be detected
- True biological duplicates at identical positions should be retained as separate molecules and no edit-distance-based merging is desired

## Inputs

- Sorted SAM file from STAR alignment of UMI-clipped FASTQ reads
- max_edit_distance parameter (integer Levenshtein distance threshold for merging UMIs)

## Outputs

- STDOUT stream of genomic position, number of UMI, and space-delimited UMIs (input to umi_analyzer_uthash)
- all_sam_umis.txt: sorted log of every UMI observed in the SAM file
- collapsed_sam_umis.txt: sorted non-redundant list of UMIs after collapsing

## How to apply

Run umi_counterizer.pl on the sorted SAM file with a max_edit_distance argument: at the same mapping position, UMIs closer than this Levenshtein distance (computed via Text::Levenshtein::XS, with Try::Tiny for error handling) are collapsed pairwise into the most abundant UMI, while more distant UMIs are kept as distinct molecules. The script opens the sorted SAM, identifies UMI duplicates at the same position, collapses them, counts the UMIs at each position, and prints position, UMI count, and the space-delimited UMIs to STDOUT, while logging every observed UMI to all_sam_umis.txt and the non-redundant set to collapsed_sam_umis.txt. The collapsed output feeds downstream steps (umi_analyzer_uthash to find UMI genomic positions, then bedtools intersect against a GFF and gene_umi_counter.pl for per-gene UMI counts). Collapsed counting serves as the control condition when testing whether uncollapsed counting inflates gene expression estimates in DESeq2 comparisons.

## Related tools

- **Perl** (Language of umi_counterizer.pl, the script that identifies UMI duplicates at the same genomic position and collapses them pairwise into the most abundant UMI)
- **Text::Levenshtein::XS** (Provides fast Levenshtein edit-distance computation used to decide whether two UMIs at the same position are within max_edit_distance and should be merged)
- **Try::Tiny** (Perl dependency of umi_counterizer.pl for error handling during pairwise UMI comparison)
- **uthash** (C preprocessor-macro hash used by the downstream umi_analyzer_uthash step that maps each collapsed UMI to its genomic positions (umi_positions.txt)) — http://troydhanson.github.io/uthash/userguide.html
- **STAR** (Produces the sorted SAM file (aligned with no introns) that is the input to umi_counterizer.pl) — https://github.com/alexdobin/STAR

## Examples

```
perl umi_counterizer.pl sorted.sam 1 > position_umi_counts.txt
```

## Evaluation signals

- Position-wise output format is correct: each line is 'Chr:position count UMIs...' with space-delimited UMIs (e.g. 'Chr01:219196 6 TGATGGAGGG TCAACCGGGG ...')
- collapsed_sam_umis.txt is sorted and non-redundant, while all_sam_umis.txt records every UMI observed in the SAM file
- UMIs at the same position differing by more than max_edit_distance are NOT collapsed; those within the threshold merge into the single most abundant UMI
- Per-gene counts derived from collapsed output serve as the control in DESeq2 comparisons; on the paper's data, collapsing reproduces the reported differences (60 La Manno genes at adjusted P<0.05; 17% of Jaitin and 0.6% of Nikaido genes at P<0.05 versus uncollapsed counting)
- Downstream umi_positions.txt can be generated without parsing errors, confirming the STDOUT structure matches umi_analyzer_uthash's space-delimited expectations

## Limitations

- Not finished software: little argument checking, so unexpected behavior (e.g. substrings longer than read length, negative values) may occur and output files are overwritten without warning
- The 3 bp mapping-shift collapse threshold is an arbitrary configurable parameter; the true distribution of mapping shifts is unknown and reads from the same template could have larger shifts
- The scripting-based pipeline is deliberately speed-sensitive: umi_analyzer_uthash assumes a rigid input structure with no error checking, and the UMI length and maximum line length must be hard-coded in exe.c before compiling
- Only single-ended sequencing was exercised by the authors' use case

## Evidence

- [readme] README describes the collapse rule: [evidence span withheld — anchor status: external_doc]
- [readme] README describes pairwise collapsing behavior: "Identify UMI duplicates at the same position. 
   * Collapse duplicates pairwise into most abundant UMI"
- [discussion] Article discussion on grouping clusters as single observations: "our analysis pipeline, which groups clusters of closely-spaced reads together as single observations"
- [methods] Article methods entry point for the collapsing step: [evidence span withheld — anchor status: not_found]
