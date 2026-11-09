---
name: selection-rule-based-read-filtering
description: Use when small RNA-seq FASTQ reads must be classified and quantified
  against GFF3/GTF feature annotations while resolving overlapping features or distinguishing
  miRNAs from isomiRs at single-nucleotide precision.
when_to_use_negative:
- Input is already a feature-level counts matrix — the selection-rule filtering applies
  to raw read-to-feature assignment, not to post-count matrices.
- Reads are full-length transcript RNA-seq rather than small RNA-seq; tiny-count is
  designed for small RNA reads including miRNAs, isomiRs, piRNAs, siRNAs, and tRNA/rRNA
  fragments.
- Feature annotations are non-standard (missing ID/gene_id/Parent tags or multi-value
  Parent tags), which the README explicitly does not support.
edam_operation: http://edamontology.org/operation_0233
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: tiny-count
  role: performs hierarchical classification and quantification of small RNA reads
    via user-defined selection rules with single-nucleotide precision
  repo: https://anaconda.org/bioconda/tiny-count
- name: tinyRNA
  role: end-to-end workflow that wraps tiny-count and coordinates counting, collapse,
    and R-based differential expression steps via CWL
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: R
  role: downstream differential expression statistics on the tiny-count feature counts
    matrix
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:5ff056610778b033bfb159ee621a6c2df10d4245bc37dd929ae4efa095a30c83
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - tiny-count
  - tinyRNA
  edam_operation: http://edamontology.org/operation_0233
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# selection-rule-based-read-filtering

## Summary

Apply tiny-count's hierarchical selection rules to classify and quantify small RNA-seq reads (miRNAs vs isomiRs, and tRNA/rRNA fragments) with single-nucleotide precision by filtering on 5′ nucleotide, length, alignment position, and mismatch count.

## When to use

Use when small RNA-seq FASTQ reads must be classified and quantified against GFF3/GTF feature annotations while resolving overlapping features or distinguishing miRNAs from isomiRs at single-nucleotide precision.

## When NOT to use

- Input is already a feature-level counts matrix — the selection-rule filtering applies to raw read-to-feature assignment, not to post-count matrices.
- Reads are full-length transcript RNA-seq rather than small RNA-seq; tiny-count is designed for small RNA reads including miRNAs, isomiRs, piRNAs, siRNAs, and tRNA/rRNA fragments.
- Feature annotations are non-standard (missing ID/gene_id/Parent tags or multi-value Parent tags), which the README explicitly does not support.

## Inputs

- demultiplexed small RNA-seq reads in FASTQ (.gz)
- reference feature annotations in GFF3/GFF2/GTF format with ID/gene_id/Parent attributes
- reference genome in FASTA format (chromosome identifiers matching the annotation file)
- tinyRNA Features Sheet encoding hierarchical selection rules (5′ nucleotide, length, alignment position, mismatch number)

## Outputs

- counts matrix of features (miRNAs, isomiRs, tRNA/rRNA fragments) per sample
- pipeline statistics files from tinyRNA
- differential expression table (e.g. isomiRs significantly upregulated in ZSWIM8-knockout lines)

## How to apply

Run tiny-count against demultiplexed FASTQ reads (optionally genome- or transcriptome-aligned) plus reference feature annotations (GFF3/GFF2/GTF with ID/gene_id/Parent tags in column 9). Define selection rules in the tinyRNA Features Sheet that hierarchically filter reads by 5′ nucleotide, length, position of alignments relative to reference features, and number of mismatches to reference sequences, so ambiguous loci (e.g. piRNAs vs siRNAs from the same locus) and miRNA/isomiR variants are resolved rather than double-counted. tiny-count can be run standalone (installed via Bioconda) or as part of the tinyRNA end-to-end workflow configured through a Run Config, Paths File, Features Sheet, and Samples Sheet. Aggregate per-sample counts into a feature-by-sample counts matrix suitable for downstream R-based differential expression (e.g. testing isomiRs upregulated in ZSWIM8-knockout vs control lines).

## Related tools

- **tiny-count** (performs hierarchical classification and quantification of small RNA reads via user-defined selection rules with single-nucleotide precision) — https://anaconda.org/bioconda/tiny-count
- **tinyRNA** (end-to-end workflow that wraps tiny-count and coordinates counting, collapse, and R-based differential expression steps via CWL) — https://github.com/MontgomeryLab/tinyRNA
- **R** (downstream differential expression statistics on the tiny-count feature counts matrix)

## Examples

```
conda install -c bioconda -c conda-forge tiny-count
```

## Evaluation signals

- Output counts matrix has one column per sample (matching the Samples Sheet) and one row per assigned feature with counts only for features passing the selection rules.
- miRNA and isomiR variants are distinguished at single-nucleotide resolution — e.g. isomiRs differing by a single nucleotide at the 5′ end or length appear as separate features.
- Reads from the same locus are partitioned among distinct classes (e.g. piRNAs vs siRNAs) with no double assignment.
- Pipeline statistics report counts consistent with the mismatch, length, 5′-nucleotide, and position filters defined in the Features Sheet.
- Downstream differential expression table (isomir_differential_expression.csv) contains only isomiR features significant in the contrast of interest (e.g. control vs ZSWIM8 knockout).

## Limitations

- GFF3/GFF2/GTF annotations with Parent tags carrying multiple values are not yet supported.
- Stable tinyRNA release installation may fail on Apple Silicon–based systems (the development release addresses this); the standalone tiny-count conda install is the lighter-weight fallback.
- FASTQ files must be demultiplexed before counting.
- Reference genome chromosome identifiers are case sensitive and must match the annotation file.

## Evidence

- [abstract] Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences.: "Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences."
- [abstract] single-nucleotide precision: "It can distinguish small RNA variants, such as miRNAs and isomiRs, with single-nucleotide precision."
- [abstract] tiny-count can resolve distinct classes of small RNAs, for example, piRNAs and siRNAs, produced from the same locus: "tiny-count can resolve distinct classes of small RNAs, for example, piRNAs and siRNAs, produced from the same locus."
- [readme] Features Sheet: [evidence span withheld — anchor status: external_doc]
- [readme] GFF annotation requirements: [evidence span withheld — anchor status: external_doc]
