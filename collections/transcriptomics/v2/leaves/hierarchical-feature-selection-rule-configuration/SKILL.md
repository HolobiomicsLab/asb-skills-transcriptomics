---
name: hierarchical-feature-selection-rule-configuration
description: Use when small RNA-seq reads mapping to the same loci must be split into
  distinct classes (e.g., 22G- vs 26G-RNAs by length 22/26, 5′G, antisense alignment)
  or resolved at single-nucleotide precision, rather than bulk-counted to features.
when_to_use_negative:
- Input is already a feature-level count table — no reads need to be aligned or classified
  by selection rules
- The analysis only needs bulk quantification to transcripts/genomic features with
  no class distinctions, variants (miRNAs/isomiRs), or same-locus conflicts to resolve
- The reference annotation lacks the required ID/gene_id or Parent tags, or features
  have Parent tags with multiple values (unsupported by tiny-count)
edam_operation: http://edamontology.org/operation_0234
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: tiny-count
  role: Core counting tool that applies user-defined hierarchical selection rules
    (5′ nucleotide, length, alignment position, mismatch number) to classify and quantify
    small RNA reads at single-nucleotide precision
  repo: github.com/MontgomeryLab/tinyRNA
- name: tinyRNA
  role: End-to-end workflow coordinating tiny-count with config files (Run Config,
    Paths File, Features Sheet, Samples Sheet), producing counts tables and statistics
  repo: github.com/MontgomeryLab/tinyRNA
- name: Python
  role: Implementation language of tiny-count/tinyRNA tools; used for downstream comparison
    of class-specific count matrices (e.g., mut-16 vs wild type)
  repo: github.com/MontgomeryLab/tinyRNA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:16a152fdd7b444efef97bc201a6e6eff60b81747185f40f207203212c8d37a44
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - tiny-count
  - tinyRNA
  edam_operation: http://edamontology.org/operation_0234
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# hierarchical-feature-selection-rule-configuration

## Summary

Configure tiny-count selection rules to hierarchically classify and quantify small RNA-seq reads by 5′ nucleotide, length, alignment position relative to features, and mismatch count — e.g., separating C. elegans 22G- and 26G-RNAs produced from the same genes with single-nucleotide precision.

## When to use

Use when small RNA-seq reads mapping to the same loci must be split into distinct classes (e.g., 22G- vs 26G-RNAs by length 22/26, 5′G, antisense alignment) or resolved at single-nucleotide precision, rather than bulk-counted to features.

## When NOT to use

- Input is already a feature-level count table — no reads need to be aligned or classified by selection rules
- The analysis only needs bulk quantification to transcripts/genomic features with no class distinctions, variants (miRNAs/isomiRs), or same-locus conflicts to resolve
- The reference annotation lacks the required ID/gene_id or Parent tags, or features have Parent tags with multiple values (unsupported by tiny-count)

## Inputs

- Demultiplexed small RNA-seq FASTQ(.gz) files (e.g., C. elegans germline wild type and mut-16 mutant samples)
- Reference genome FASTA with chromosome identifiers matching the annotation file (case sensitive)
- Reference annotations in GFF3/GFF2/GTF with column-9 ID/gene_id or Parent tags
- Features Sheet defining hierarchical selection rules (5′ nucleotide, length, alignment position, mismatch count)
- Samples Sheet and Paths File / Run Config for the tinyRNA pipeline

## Outputs

- Per-gene counts matrix (counts_matrix.tsv) with columns/categories per selection-rule class (e.g., 22G-RNA, 26G-RNA, unclassified siRNA)
- Pipeline statistics and documentation of counts produced by the tinyRNA workflow

## How to apply

Collect demultiplexed FASTQ(.gz) reads plus a reference genome FASTA and GFF3/GFF2/GTF annotations whose chromosome identifiers match the genome file exactly and whose column-9 attributes carry ID/gene_id/Parent tags. Write selection rules in the tinyRNA Features Sheet that encode each small RNA class as a filter combination — for C. elegans, 22G-RNAs as length 22 with 5′G and antisense alignment, 26G-RNAs as length 26 with 5′G and antisense alignment — and order the rules hierarchically so classes produced from the same gene are resolved in a defined priority order. Run tiny-count (alone, or via the tinyRNA workflow) to quantify genome-aligned reads against gene features, applying the hierarchical rules at single-nucleotide precision. Export the per-gene counts matrix separating class categories (e.g., 22G-RNA, 26G-RNA, unclassified siRNA) as counts_matrix.tsv, then compare counts across conditions. For the mut-16 case, the expected behavior is that unclassified siRNAs depleted in mut-16 mutants are identified while 22G/26G classes from the same genes remain cleanly separated.

## Related tools

- **tiny-count** (Core counting tool that applies user-defined hierarchical selection rules (5′ nucleotide, length, alignment position, mismatch number) to classify and quantify small RNA reads at single-nucleotide precision) — github.com/MontgomeryLab/tinyRNA
- **tinyRNA** (End-to-end workflow coordinating tiny-count with config files (Run Config, Paths File, Features Sheet, Samples Sheet), producing counts tables and statistics) — github.com/MontgomeryLab/tinyRNA
- **Python** (Implementation language of tiny-count/tinyRNA tools; used for downstream comparison of class-specific count matrices (e.g., mut-16 vs wild type)) — github.com/MontgomeryLab/tinyRNA

## Examples

```
conda activate tinyrna; tiny run   # full workflow using Run Config, Paths File, Features Sheet (selection rules), and Samples Sheet
```

## Evaluation signals

- The counts table cleanly resolves distinct small RNA classes produced from the same genes (e.g., 22G- vs 26G-RNAs) via the 5′ nucleotide/length/antisense selection rules, with no reads double-assigned across rules
- Class definitions are honored exactly: 22G-RNAs are length 22 with 5′G and antisense alignment; 26G-RNAs are length 26 with 5′G and antisense alignment
- Biological validation: unclassified siRNAs show depletion in mut-16 mutants relative to wild type, consistent with the MUT-16-dependent siRNA phenotype
- Chromosome identifiers in the genome FASTA and annotation file match exactly (case sensitive), so every feature is resolvable during counting
- GFF3/GTF attributes parse correctly under tiny-count requirements: every feature has ID or gene_id or Parent, and comma-containing values represent lists

## Limitations

- GFF/GTF Parent tags with multiple values are not yet supported, which limits how some discontinuous or grouped features can be classified
- Stable tinyRNA releases may have installation issues on Apple Silicon–based systems; the development release addresses this
- tiny-count requires demultiplexed FASTQ input — multiplexed reads must be split before counting

## Evidence

- [abstract] Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences.: "Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences."
- [abstract] tiny-count can resolve distinct classes produced from the same locus.: "tiny-count can resolve distinct classes of small RNAs, for example, piRNAs and siRNAs, produced from the same locus."
- [readme] Your selection preferences for feature counting via the Features Sheet.: [evidence span withheld — anchor status: external_doc]
- [readme] The goal of this specific repository is to provide an entire workflow for processing small RNA sequencing data with user-defined hierarchical feature selection rules.: "The goal of this specific repository is to provide an entire workflow for processing small RNA sequencing data with user-defined hierarchical feature selection rules."
