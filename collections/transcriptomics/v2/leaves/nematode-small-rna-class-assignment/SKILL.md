---
name: nematode-small-rna-class-assignment
description: Use when quantifying C. elegans small RNA classes (22G-/26G-RNAs) from
  genome-aligned small RNA-seq FASTQ data, especially when distinct classes derive
  from the same genes and must be separated by 5′ nucleotide, length, and strandedness
  rules.
when_to_use_negative:
- Input is already a per-feature counts table — hierarchical re-counting from aligned
  reads is unnecessary
- The small RNA classes of interest cannot be distinguished by 5′ nucleotide, length,
  alignment position, or mismatch count (tiny-count's supported filter axes)
- Reads are not demultiplexed or the genome FASTA chromosome identifiers do not match
  the annotation file identifiers
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: tiny-count
  role: Core counting tool that applies user-defined hierarchical selection rules
    (5′ nucleotide, length, antisense alignment) to classify and quantify small RNA
    reads from the same loci
  repo: github.com/MontgomeryLab/tinyRNA
- name: tinyRNA
  role: End-to-end workflow wrapper (coordinated with CWL) that preprocesses FASTQ,
    runs tiny-count, and exports the counts matrix
  repo: github.com/MontgomeryLab/tinyRNA
- name: Python
  role: Implementation language of tiny-count/tinyRNA; used for downstream comparison
    of mutant vs wild type per-gene counts
  repo: github.com/MontgomeryLab/tinyRNA
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:328d436b56485764d40878239052f54dea8401dc99248aceaa980e41497f0fe9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - tiny-count
  - tinyRNA
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# nematode-small-rna-class-assignment

## Summary

Configure tiny-count selection rules to hierarchically classify C. elegans 22G- and 26G-RNAs by length, 5′G, and antisense strandedness, resolving distinct siRNA classes produced from the same gene loci in wild type vs mut-16 small RNA-seq data.

## When to use

Use when quantifying C. elegans small RNA classes (22G-/26G-RNAs) from genome-aligned small RNA-seq FASTQ data, especially when distinct classes derive from the same genes and must be separated by 5′ nucleotide, length, and strandedness rules.

## When NOT to use

- Input is already a per-feature counts table — hierarchical re-counting from aligned reads is unnecessary
- The small RNA classes of interest cannot be distinguished by 5′ nucleotide, length, alignment position, or mismatch count (tiny-count's supported filter axes)
- Reads are not demultiplexed or the genome FASTA chromosome identifiers do not match the annotation file identifiers

## Inputs

- Demultiplexed small RNA-seq FASTQ(.gz) files (wild type and mut-16 mutant C. elegans germline samples)
- C. elegans reference genome in FASTA format (chromosome identifiers matching annotation, case sensitive)
- C. elegans gene annotations in GFF3/GFF2/GTF with ID/gene_id/Parent tags in column 9
- tinyRNA configuration files: Run Config, Paths File, Features Sheet (selection rules), Samples Sheet

## Outputs

- Per-gene counts matrix (e.g. counts_matrix.tsv) separating 22G-RNA, 26G-RNA, and unclassified siRNA categories
- Pipeline statistics and hierarchical classification report from tiny-count

## How to apply

Download the C. elegans germline small RNA-seq FASTQ files (wild type and mut-16, Reed et al. 2020) plus the C. elegans genome FASTA and gene GFF3/GTF annotations from the MontgomeryLab reference data site. Define hierarchical selection rules in a Features Sheet: 22G-RNAs filtered by length 22 and 5′G with antisense alignment, 26G-RNAs by length 26 and 5′G with antisense alignment, ordered so classes produced from the same gene are resolved by rule precedence. Run tiny-count (alone or within the tinyRNA workflow, configured via Run Config, Paths File, Features Sheet, and Samples Sheet) to quantify genome-aligned reads against gene features with single-nucleotide precision. Export the per-gene counts matrix separating 22G-RNA, 26G-RNA, and unclassified siRNA categories, then compare mutant vs wild type counts to identify MUT-16-dependent unclassified siRNAs.

## Related tools

- **tiny-count** (Core counting tool that applies user-defined hierarchical selection rules (5′ nucleotide, length, antisense alignment) to classify and quantify small RNA reads from the same loci) — github.com/MontgomeryLab/tinyRNA
- **tinyRNA** (End-to-end workflow wrapper (coordinated with CWL) that preprocesses FASTQ, runs tiny-count, and exports the counts matrix) — github.com/MontgomeryLab/tinyRNA
- **Python** (Implementation language of tiny-count/tinyRNA; used for downstream comparison of mutant vs wild type per-gene counts) — github.com/MontgomeryLab/tinyRNA

## Examples

```
conda install -c bioconda -c conda-forge tiny-count
```

## Evaluation signals

- Counts table cleanly resolves 22G- and 26G-RNA classes from the same genes via the 5′G/length/antisense rules, with no cross-class assignment at single-gene level
- Unclassified siRNAs show depletion in mut-16 mutants consistent with the reported MUT-16-dependent siRNA phenotype
- Counts matrix columns are separated by class category (22G-RNA, 26G-RNA, unclassified) as required
- Reference genome chromosome identifiers match annotation chromosome identifiers and are case-sensitive per README requirements
- tiny-count reports pipeline statistics confirming reads were aligned to the genome and quantified against gene features

## Limitations

- Selection rules can only filter by 5′ nucleotide, length, alignment position relative to features, and mismatch number — classes requiring other criteria cannot be resolved
- Parent tags with multiple values are not yet supported in GFF3/GTF annotations
- Feature annotations must carry an ID, gene_id, or Parent tag in column 9; attribute values containing commas must represent lists

## Evidence

- [abstract] Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences.: "Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences."
- [abstract] tiny-count can resolve distinct classes of small RNAs produced from the same locus: "tiny-count can resolve distinct classes of small RNAs, for example, piRNAs and siRNAs, produced from the same locus."
- [abstract] It can distinguish small RNA variants, such as miRNAs and isomiRs, with single-nucleotide precision.: "It can distinguish small RNA variants, such as miRNAs and isomiRs, with single-nucleotide precision."
- [abstract] tiny-count can quantify reads aligned to a genome or directly to small RNA or transcript sequences: "tiny-count can quantify reads aligned to a genome or directly to small RNA or transcript sequences."
- [readme] Files must be demultiplexed (README input requirements): [evidence span withheld — anchor status: external_doc]
