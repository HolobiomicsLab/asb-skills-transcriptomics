---
name: sirna-class-co-production-resolution
description: Use when small RNA-seq reads from multiple biologically distinct classes
  (e.g., C. elegans 22G- and 26G-RNAs) derive from the same genes and must be resolved
  into separate per-feature count tables rather than merged.
when_to_use_negative:
- Input is already a feature-level counts table — hierarchical read classification
  has no read-level information left to apply
- Only one small RNA class is present and classes cannot share loci — simple counting
  suffices without selection-rule hierarchy
- Reference annotations lack required ID/gene_id/Parent column-9 attributes, so features
  cannot be resolved by tiny-count
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: tiny-count
  role: Standalone counting tool performing hierarchical classification and quantification
    of small RNA reads via user-defined selection rules (5' nucleotide, length, alignment
    position, mismatches)
  repo: github.com/MontgomeryLab/tinyRNA
- name: tinyRNA
  role: End-to-end CWL-coordinated workflow wrapping tiny-count with preprocessing,
    differential expression, and plotting for small RNA-seq analysis
  repo: github.com/MontgomeryLab/tinyRNA
- name: Python
  role: Implementation language for tiny-count and other tinyRNA tools, usable for
    downstream comparison of class-specific counts
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:716483346c6af6737c3be26727a7a36e06e788948c4998c96a728a7968bf1ffb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - tiny-count
  - tinyRNA
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# sirna-class-co-production-resolution

## Summary

Configure tiny-count selection rules to hierarchically classify small RNA-seq reads into distinct classes (e.g., 22G- vs 26G-RNAs) produced from the same locus, filtering by 5' nucleotide, length, and antisense alignment with single-nucleotide precision.

## When to use

Use when small RNA-seq reads from multiple biologically distinct classes (e.g., C. elegans 22G- and 26G-RNAs) derive from the same genes and must be resolved into separate per-feature count tables rather than merged.

## When NOT to use

- Input is already a feature-level counts table — hierarchical read classification has no read-level information left to apply
- Only one small RNA class is present and classes cannot share loci — simple counting suffices without selection-rule hierarchy
- Reference annotations lack required ID/gene_id/Parent column-9 attributes, so features cannot be resolved by tiny-count

## Inputs

- Demultiplexed small RNA-seq FASTQ(.gz) files (e.g., C. elegans germline wild type and mut-16 mutant samples from Reed et al. 2020)
- Reference genome FASTA (chromosome identifiers matching the annotation file, case-sensitive)
- Gene feature annotations in GFF3/GFF2/GTF with ID or gene_id or Parent tags in column 9
- Features Sheet defining hierarchical selection rules (5' nucleotide, length, antisense, mismatch filters)
- Samples Sheet and Run Config / Paths File for pipeline execution

## Outputs

- Per-gene counts matrix (e.g., counts_matrix.tsv) with separate columns/categories per small RNA class (22G-RNA, 26G-RNA, unclassified siRNA)
- Pipeline statistics and counts tables from the tinyRNA workflow

## How to apply

Define selection rules in the tiny-count Features Sheet that encode each class's defining attributes: for nematode 22G-RNAs filter reads by length 22, 5'G, and antisense alignment relative to gene features; for 26G-RNAs filter by length 26, 5'G, and antisense alignment. Arrange the rules hierarchically so reads matching an earlier class are assigned before later classes, resolving co-produced classes from the same gene. Run tiny-count with reads aligned to the genome (FASTA) against gene annotations (GFF3/GTF with ID/Parent tags) to produce a per-gene counts matrix (e.g., counts_matrix.tsv) that separates 22G-RNA, 26G-RNA, and unclassified siRNA categories. Then compare counts between conditions (e.g., mut-16 mutant vs wild type) to identify class-specific depletion. tiny-count can be run standalone via Bioconda or within the tinyRNA end-to-end workflow.

## Related tools

- **tiny-count** (Standalone counting tool performing hierarchical classification and quantification of small RNA reads via user-defined selection rules (5' nucleotide, length, alignment position, mismatches)) — github.com/MontgomeryLab/tinyRNA
- **tinyRNA** (End-to-end CWL-coordinated workflow wrapping tiny-count with preprocessing, differential expression, and plotting for small RNA-seq analysis) — github.com/MontgomeryLab/tinyRNA
- **Python** (Implementation language for tiny-count and other tinyRNA tools, usable for downstream comparison of class-specific counts)

## Examples

```
conda install -c bioconda -c conda-forge tiny-count
```

## Evaluation signals

- Counts table cleanly resolves 22G- and 26G-RNA classes produced from the same genes via the 5'G/length/antisense rules
- 22G-RNA category contains predominantly 22-nt, 5'G antisense reads and 26G-RNA category predominantly 26-nt, 5'G antisense reads (single-nucleotide precision on length)
- Unclassified siRNAs show depletion in mut-16 mutants consistent with the reported MUT-16-dependent siRNA phenotype
- Per-gene counts matrix separates 22G-, 26G-, and unclassified-siRNA categories as distinct columns without double-counting reads across classes
- tiny-count completes against the C. elegans genome FASTA and GFF3 annotations with chromosome identifiers matching between genome and annotation files

## Limitations

- Stable tinyRNA release installation may encounter issues on Apple Silicon-based systems
- GFF3/GTF attributes with Parent tags carrying multiple values are not yet supported
- FASTQ files must be demultiplexed; non-demultiplexed input is not accepted

## Evidence

- [abstract] tiny-count can resolve distinct classes of small RNAs, for example, piRNAs and siRNAs, produced from the same locus.: "tiny-count can resolve distinct classes of small RNAs, for example, piRNAs and siRNAs, produced from the same locus."
- [abstract] Selection rules can filter by 5' nucleotide, length, alignment position, and mismatches.: "Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences."
- [readme] README: counting preferences are set via the Features Sheet.: [evidence span withheld — anchor status: external_doc]
- [readme] README: annotation column-9 attribute requirements.: [evidence span withheld — anchor status: external_doc]
- [readme] README: standalone tiny-count installation via Bioconda.: [evidence span withheld — anchor status: external_doc]
