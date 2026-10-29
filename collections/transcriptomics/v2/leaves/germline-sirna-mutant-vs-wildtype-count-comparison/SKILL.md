---
name: germline-sirna-mutant-vs-wildtype-count-comparison
description: Use when small RNA-seq FASTQ data from wild type and mutant samples (e.g.
  C. elegans germline, mut-16) must be quantified per-gene by small RNA class defined
  by length, 5′ nucleotide, and antisense strandedness, with classes resolved from
  the same loci.
when_to_use_negative:
- Input is already a counts/feature table rather than raw aligned reads — tiny-count
  quantifies reads from FASTQ/BAM against reference features, not pre-existing feature
  tables
- The small RNA classes of interest cannot be defined by 5′ nucleotide, length, position
  relative to features, or mismatch number — selection-rule filtering will not separate
  them
- _reads are not demultiplexed; the workflow requires demultiplexed FASTQ files per
  sample
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: tiny-count
  role: Core counting tool performing hierarchical classification and quantification
    of small RNA reads with single-nucleotide precision using selection rules (5′
    nucleotide, length, alignment position, mismatches)
  repo: github.com/MontgomeryLab/tinyRNA
- name: tinyRNA
  role: End-to-end workflow (CWL-coordinated) that wraps tiny-count with pre-processing,
    counts/statistics, and differential expression analysis; provides Run Config,
    Paths File, Features Sheet, and Samples Sheet
  repo: github.com/MontgomeryLab/tinyRNA
- name: Python
  role: Implementation language of tiny-count/tinyRNA tools and used for comparing
    counts between mut-16 mutant and wild type samples
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:3725965b742f65c6296edeabdddd53ff3d3d04059f2e588bd444c03085ce835f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - tiny-count
  - tinyRNA
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# germline-sirna-mutant-vs-wildtype-count-comparison

## Summary

A method for hierarchically classifying and quantifying C. elegans small RNA classes (22G-RNAs, 26G-RNAs, unclassified siRNAs) from germline small RNA-seq with tiny-count, using 5′ nucleotide/length/antisense selection rules, and comparing counts between mut-16 mutants and wild type to identify MUT-16-dependent siRNAs.

## When to use

Use when small RNA-seq FASTQ data from wild type and mutant samples (e.g. C. elegans germline, mut-16) must be quantified per-gene by small RNA class defined by length, 5′ nucleotide, and antisense strandedness, with classes resolved from the same loci.

## When NOT to use

- Input is already a counts/feature table rather than raw aligned reads — tiny-count quantifies reads from FASTQ/BAM against reference features, not pre-existing feature tables
- The small RNA classes of interest cannot be defined by 5′ nucleotide, length, position relative to features, or mismatch number — selection-rule filtering will not separate them
- _reads are not demultiplexed; the workflow requires demultiplexed FASTQ files per sample

## Inputs

- Demultiplexed small RNA-seq FASTQ files (.fastq or .fastq.gz) for wild type and mut-16 mutant C. elegans germline samples
- C. elegans reference genome FASTA (chromosome identifiers matching the annotation file, case sensitive)
- C. elegans gene feature annotations in GFF3/GFF2/GTF with ID/gene_id/Parent attributes in column 9
- tinyRNA configuration files: Run Config, Paths File, Features Sheet (hierarchical selection rules), Samples Sheet

## Outputs

- Per-gene counts matrix (counts_matrix.tsv) with columns separating 22G-RNA, 26G-RNA, and unclassified siRNA categories per sample
- Comparison of counts between mut-16 mutant and wild type identifying unclassified siRNAs depleted in mut-16

## How to apply

Download demultiplexed FASTQ files for wild type and mut-16 mutant germline samples (Reed et al. 2020 deposit) plus the C. elegans genome FASTA and GFF3/GTF gene annotations (MontgomeryLab reference data site). Configure tiny-count Features Sheet selection rules hierarchically: 22G-RNAs filter reads by length 22, 5′G, and antisense alignment to gene features; 26G-RNAs filter by length 26, 5′G, and antisense alignment; rules are ordered so 22G- and 26G-RNA classes produced from the same gene are resolved by hierarchical classification with single-nucleotide precision. Run tiny-count (or the tinyRNA pipeline with Run Config, Paths File, Features Sheet, and Samples Sheet) to quantify reads aligned to the genome against the gene features, exporting a per-gene counts matrix (counts_matrix.tsv) separating 22G-RNA, 26G-RNA, and unclassified siRNA categories. Finally, compare per-gene counts between mut-16 mutant and wild type samples (e.g. with the tinyRNA differential expression step or Python) to identify unclassified siRNAs depleted in mut-16, consistent with the MUT-16-dependent siRNA phenotype.

## Related tools

- **tiny-count** (Core counting tool performing hierarchical classification and quantification of small RNA reads with single-nucleotide precision using selection rules (5′ nucleotide, length, alignment position, mismatches)) — github.com/MontgomeryLab/tinyRNA
- **tinyRNA** (End-to-end workflow (CWL-coordinated) that wraps tiny-count with pre-processing, counts/statistics, and differential expression analysis; provides Run Config, Paths File, Features Sheet, and Samples Sheet) — github.com/MontgomeryLab/tinyRNA
- **Python** (Implementation language of tiny-count/tinyRNA tools and used for comparing counts between mut-16 mutant and wild type samples)

## Examples

```
conda install -c bioconda -c conda-forge tiny-count
```

## Evaluation signals

- The counts table must cleanly resolve 22G- and 26G-RNA classes produced from the same genes via the 5′ nucleotide/length/antisense selection rules (hierarchical classification with single-nucleotide precision)
- Unclassified siRNA counts must show depletion in mut-16 mutants relative to wild type, consistent with the reported MUT-16-dependent siRNA phenotype
- 22G-RNA counts correspond to reads of length 22 with 5′G aligned antisense; 26G-RNA counts correspond to reads of length 26 with 5′G aligned antisense
- Genome FASTA chromosome identifiers match the reference annotation chromosome identifiers (case sensitive), and every feature has an ID/gene_id/Parent tag so quantification covers all gene features
- Pipeline statistics generated by the tinyRNA workflow report counting results for all samples in the Samples Sheet without unassigned read categories that would indicate misconfigured selection rules

## Limitations

- Stable-release installation may encounter issues on Apple Silicon–based systems; the development release addresses these installation issues
- Selection rules can only filter reads by 5′ nucleotide, length, alignment position relative to reference features, and mismatch number — classes not separable by these axes cannot be distinguished
- GFF3/GFF2/GTF annotations must follow specific attribute requirements (ID/gene_id/Parent tags; comma values must be lists; multi-value Parent tags not yet supported), and chromosome identifiers are case sensitive
- Updating an existing tinyRNA installation removes the existing conda environment, including any additionally installed packages

## Evidence

- [abstract] Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences.: "Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences."
- [abstract] tiny-count can resolve distinct classes of small RNAs, for example, piRNAs and siRNAs, produced from the same locus.: "tiny-count can resolve distinct classes of small RNAs, for example, piRNAs and siRNAs, produced from the same locus."
- [readme] The pipeline requires that you identify: Your preferences for the pipeline and its steps via the Run Config, The location of your config files and other file inputs via the Paths File, Your selection preferences for feature counting via the Features Sheet, Your samples via the Samples Sheet: [evidence span withheld — anchor status: external_doc]
- [readme] Sequencing data ... FASTQ(.gz) ... Files must be demultiplexed.: [evidence span withheld — anchor status: external_doc]
- [abstract] tiny-count can be run alone or as part of tinyRNA, a workflow that provides a basic all-in-one command line-based solution for small RNA-seq data analysis: "tiny-count can be run alone or as part of tinyRNA, a workflow that provides a basic all-in-one command line-based solution for small RNA-seq data analysis"
