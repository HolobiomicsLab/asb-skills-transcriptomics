---
name: mirna-isomir-hierarchical-read-counting
description: Use when you have demultiplexed small RNA-seq FASTQ reads plus miRNA
  reference annotations (GFF3/GTF) and need to distinguish miRNAs from isomiRs at
  single-nucleotide resolution and test their abundance between conditions (e.g. control
  vs knockout).
when_to_use_negative:
- Input is already a feature count matrix — tiny-count is for raw FASTQ-level classification/quantification,
  not re-analysis of count tables
- You need isoform quantification for long-read or mRNA transcriptomics rather than
  small RNA variants (miRNA/isomiR, piRNA, siRNA, tRNA/rRNA fragments)
- Your reference annotations lack the required ID/gene_id/Parent column-9 attributes,
  since tiny-count's feature selection depends on these tags
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0099
tools:
- name: tiny-count
  role: Hierarchical classification and quantification of small RNA reads into miRNA/isomiR
    features using 5′ nucleotide, length, alignment-position, and mismatch selection
    rules
  repo: github.com/MontgomeryLab/tinyRNA
- name: tinyRNA
  role: End-to-end CWL-coordinated workflow wrapping tiny-count plus R-based differential
    expression analysis for small RNA-seq
  repo: github.com/MontgomeryLab/tinyRNA
- name: R
  role: Runs the tinyRNA differential expression statistics step testing feature abundance
    between conditions
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:195554b0e1de9f23a44bab60f82d6caf86b9495b8981911b2f9fca6009b5f547
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - tiny-count
  - tinyRNA
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# mirna-isomir-hierarchical-read-counting

## Summary

Classify and quantify small RNA-seq reads as miRNAs/isomiRs (or piRNAs, siRNAs, tRNA/rRNA fragments) with single-nucleotide precision using tiny-count's hierarchical selection rules, then feed the per-sample counts matrix into tinyRNA's R-based differential expression step to detect condition-specific isomiRs.

## When to use

Use when you have demultiplexed small RNA-seq FASTQ reads plus miRNA reference annotations (GFF3/GTF) and need to distinguish miRNAs from isomiRs at single-nucleotide resolution and test their abundance between conditions (e.g. control vs knockout).

## When NOT to use

- Input is already a feature count matrix — tiny-count is for raw FASTQ-level classification/quantification, not re-analysis of count tables
- You need isoform quantification for long-read or mRNA transcriptomics rather than small RNA variants (miRNA/isomiR, piRNA, siRNA, tRNA/rRNA fragments)
- Your reference annotations lack the required ID/gene_id/Parent column-9 attributes, since tiny-count's feature selection depends on these tags

## Inputs

- Demultiplexed small RNA-seq FASTQ files (.fastq or .fastq.gz)
- Reference annotations in GFF3/GFF2/GTF format with ID/gene_id/Parent attributes in column 9
- Reference genome FASTA with chromosome identifiers matching the annotation file
- tinyRNA configuration files: Run Config, Paths File, Features Sheet (selection rules), Samples Sheet

## Outputs

- Counts matrix of miRNA and isomiR features per sample (collapsed FASTA and counts/pipeline statistics files)
- Differential expression table (e.g. isomir_differential_expression.csv) with significantly changed isomiRs
- Pipeline statistics and plots from the tinyRNA workflow

## How to apply

Install tiny-count via Bioconda or the full tinyRNA suite from the MontgomeryLab/tinyRNA repository, then prepare the four configuration inputs: Run Config, Paths File, Features Sheet (hierarchical selection preferences), and Samples Sheet (obtainable via `tiny get-templates`). Run tiny-count with selection rules that filter reads by 5′ nucleotide, length, position of alignments relative to reference features, and number of mismatches to reference sequences, so reads are classified and quantified as miRNAs or isomiRs with single-nucleotide precision; tiny-count can align reads to a genome or directly to small RNA/transcript sequences. Aggregate per-sample counts into a counts matrix across all libraries, then run tinyRNA's differential expression analysis (R-based statistics) to compare conditions. Finally filter the differential expression table for features significantly changed in the condition of interest (e.g. isomiRs upregulated in ZSWIM8-knockout lines) and write results such as isomir_differential_expression.csv.

## Related tools

- **tiny-count** (Hierarchical classification and quantification of small RNA reads into miRNA/isomiR features using 5′ nucleotide, length, alignment-position, and mismatch selection rules) — github.com/MontgomeryLab/tinyRNA
- **tinyRNA** (End-to-end CWL-coordinated workflow wrapping tiny-count plus R-based differential expression analysis for small RNA-seq) — github.com/MontgomeryLab/tinyRNA
- **R** (Runs the tinyRNA differential expression statistics step testing feature abundance between conditions)

## Examples

```
conda install -c bioconda -c conda-forge tiny-count
```

## Evaluation signals

- Counts matrix contains both canonical miRNA and isomiR feature rows resolved at single-nucleotide precision (distinct 5′ nucleotide / length / mismatch variants not collapsed)
- Reads mapping to the same locus are resolved into distinct classes (e.g. piRNAs vs siRNAs) rather than merged
- Pipeline statistics files are generated and per-sample counts are present for every library in the Samples Sheet
- Differential expression output (e.g. isomir_differential_expression.csv) contains only significantly changed isomiR features for the tested contrast
- Reproducing the Shi et al. 2020 human control vs ZSWIM8-knockout dataset should recover the known upregulated isomiRs reported in the source task

## Limitations

- Stable tinyRNA releases may have installation issues on Apple Silicon–based systems; the development release addresses this
- Reference annotations with Parent tags having multiple values are not yet supported
- Updating an existing installation removes the existing conda environment, including any additional packages installed within it
- No discussion-section changelog was available in the provided article text, so additional caveats may be undocumented

## Evidence

- [abstract] Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences.: "Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences."
- [abstract] It can distinguish small RNA variants, such as miRNAs and isomiRs, with single-nucleotide precision.: "It can distinguish small RNA variants, such as miRNAs and isomiRs, with single-nucleotide precision."
- [abstract] tiny-count can be run alone or as part of tinyRNA, a workflow that provides a basic all-in-one command line-based solution for small RNA-seq data analysis: "tiny-count can be run alone or as part of tinyRNA, a workflow that provides a basic all-in-one command line-based solution for small RNA-seq data analysis"
- [readme] Files must be demultiplexed. Reference genome FASTA chromosome identifiers should match the reference annotation file chromosome identifiers and are case sensitive.: [evidence span withheld — anchor status: external_doc]
- [readme] The pipeline requires that you identify: Your preferences for the pipeline and its steps via the Run Config, the location of your config files via the Paths File, selection preferences via the Features Sheet, and your samples via the Samples Sheet.: "Your selection preferences for feature counting via the **Features Sheet**
- Your samples via the **Samples Sheet**"
