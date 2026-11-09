---
name: small-rna-seq-read-counting
description: Use when demultiplexed small RNA-seq FASTQ or SAM/aligned reads plus
  GFF3/GTF reference features must be counted into distinct small RNA classes (miRNAs,
  isomiRs, piRNAs, siRNAs such as 22G-/26G-RNAs, tRNA/rRNA fragments) produced from
  the same loci.
when_to_use_negative:
- Input is already a feature count matrix — the downstream tinyRNA differential expression
  step should be used instead of recounting
- Reads are not demultiplexed; FASTQ files must be demultiplexed before tinyRNA/tiny-count
- Annotation files lack `ID`/`gene_id`/`Parent` attributes or use multi-valued `Parent`
  tags, which are not yet supported
edam_operation: http://edamontology.org/operation_0234
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3168
tools:
- name: tiny-count
  role: Core counting tool applying hierarchical selection rules (5′ nucleotide, length,
    alignment position, mismatches) to classify and quantify small RNA reads with
    single-nucleotide precision
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: tinyRNA
  role: End-to-end CWL-coordinated workflow wrapping tiny-count with preprocessing,
    differential expression, and plotting
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: HTSeq-count
  role: Comparison counting tool in the runtime benchmark on the same collapsed/uncollapsed
    SAM files
- name: featureCounts
  role: Comparison counting tool in the runtime benchmark; fastest on collapsed SAM
    (~2.4 s vs tiny-count ~45.1 s) but counts one feature assignment rather than multiple
    small RNA classes in parallel
- name: R
  role: Implements the tinyRNA differential expression statistics step applied to
    the tiny-count counts matrix
  repo: https://github.com/MontgomeryLab/tinyRNA
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:65cd5b2431b7b3c91118bb14fb06c1703787bca20bbb8641896c21283022bb5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MontgomeryLab/tinyRNA
  tools:
  - HTSeq-count
  - Python
  - R
  - featureCounts
  - tiny-count
  - tinyRNA
  edam_operation: http://edamontology.org/operation_0234
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# small-rna-seq-read-counting

## Summary

Hierarchical classification and quantification of small RNA-seq reads with single-nucleotide precision using tiny-count, either standalone or within the tinyRNA CWL-coordinated workflow, driven by user-defined selection rules.

## When to use

Use when demultiplexed small RNA-seq FASTQ or SAM/aligned reads plus GFF3/GTF reference features must be counted into distinct small RNA classes (miRNAs, isomiRs, piRNAs, siRNAs such as 22G-/26G-RNAs, tRNA/rRNA fragments) produced from the same loci.

## When NOT to use

- Input is already a feature count matrix — the downstream tinyRNA differential expression step should be used instead of recounting
- Reads are not demultiplexed; FASTQ files must be demultiplexed before tinyRNA/tiny-count
- Annotation files lack `ID`/`gene_id`/`Parent` attributes or use multi-valued `Parent` tags, which are not yet supported

## Inputs

- Demultiplexed small RNA-seq FASTQ files (plain or .gz)
- SAM alignment files (collapsed or uncollapsed) of reads aligned to a genome, or small RNA/transcript sequences for direct alignment
- Reference genome FASTA with chromosome identifiers matching the annotation (case sensitive)
- Reference feature annotations in GFF3, GFF2, or GTF with `ID`/`gene_id`/`Parent` attributes
- tinyRNA configuration files: Run Config, Paths File, Features Sheet (selection rules), Samples Sheet

## Outputs

- Per-gene/per-feature counts matrix (e.g. counts_matrix.tsv) separating classified small RNA classes (22G-RNA, 26G-RNA, unclassified siRNA, miRNA vs isomiR)
- Counts and pipeline statistics files when run within the tinyRNA workflow
- Differential expression tables (e.g. isomir_differential_expression.csv) when paired with the tinyRNA R-based DE step
- Runtime benchmark tables (e.g. runtime_benchmark.csv) comparing tiny-count, HTSeq-count, and featureCounts on collapsed vs uncollapsed SAM inputs

## How to apply

Assemble four configuration objects: a Run Config, a Paths File, a Features Sheet defining selection rules, and a Samples Sheet (templates via `tiny get-templates`). Author hierarchical selection rules that filter reads by 5′ nucleotide, length, position of alignments relative to reference features, and number of mismatches — e.g. nematode 22G-RNAs as length 22 with 5′G antisense alignments and 26G-RNAs as length 26 with 5′G antisense, ordered hierarchically so classes from the same gene are resolved. Run tiny-count against reads aligned to a genome (or directly to small RNA/transcript sequences) with single-nucleotide precision, aggregating per-sample counts into a counts matrix separating class categories. The reference annotation must be GFF3/GFF2/GTF with each feature carrying an `ID`, `gene_id`, or `Parent` tag, and the FASTA chromosome identifiers must match the annotation and are case sensitive. For end-to-end analysis, run the tinyRNA workflow (`tiny run`) which adds preprocessing, collapse, differential expression (R-based), and plots; tiny-count alone can be run per-input, e.g. to benchmark runtime against HTSeq-count and featureCounts on collapsed vs uncollapsed SAM files.

## Related tools

- **tiny-count** (Core counting tool applying hierarchical selection rules (5′ nucleotide, length, alignment position, mismatches) to classify and quantify small RNA reads with single-nucleotide precision) — https://github.com/MontgomeryLab/tinyRNA
- **tinyRNA** (End-to-end CWL-coordinated workflow wrapping tiny-count with preprocessing, differential expression, and plotting) — https://github.com/MontgomeryLab/tinyRNA
- **HTSeq-count** (Comparison counting tool in the runtime benchmark on the same collapsed/uncollapsed SAM files)
- **featureCounts** (Comparison counting tool in the runtime benchmark; fastest on collapsed SAM (~2.4 s vs tiny-count ~45.1 s) but counts one feature assignment rather than multiple small RNA classes in parallel)
- **R** (Implements the tinyRNA differential expression statistics step applied to the tiny-count counts matrix) — https://github.com/MontgomeryLab/tinyRNA

## Examples

```
conda install -c bioconda -c conda-forge tiny-count
```

## Evaluation signals

- Counts matrix cleanly resolves distinct classes produced from the same locus (e.g. 22G- vs 26G-RNAs separated by 5′G/length/antisense rules; miRNAs vs isomiRs at single-nucleotide precision)
- Biological validation: unclassified siRNAs show depletion in C. elegans mut-16 mutants relative to wild type, consistent with the MUT-16-dependent siRNA phenotype
- Runtime benchmark reproduces the reported ordering on collapsed SAM inputs (tiny-count ~45.1 s vs featureCounts ~2.4 s), acknowledging hardware dependence
- Input GFF3/GTF validates: every feature has an `ID` or `gene_id` or `Parent` tag, and FASTA chromosome identifiers match annotation identifiers case-sensitively
- Pipeline statistics and counts files generated by the tinyRNA workflow are present and consistent across all libraries in the Samples Sheet

## Limitations

- On collapsed SAM files tiny-count is slower than featureCounts (45.1 s vs 2.4 s); its advantage is parallel quantification of multiple small RNA classes with hierarchical rules, so absolute runtimes are hardware-dependent
- Stable tinyRNA installation may encounter issues on Apple Silicon–based systems; the development release addresses this
- Multi-valued `Parent` tags in annotations are not yet supported, and updating an existing tinyRNA installation removes the existing conda environment including extra installed packages

## Evidence

- [abstract] tiny-count is a highly flexible counting tool that allows for hierarchical classification and quantification of small RNA reads from high-throughput sequencing data.: "tiny-count is a highly flexible counting tool that allows for hierarchical classification and quantification of small RNA reads from high-throughput sequencing data."
- [abstract] Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences.: "Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences."
- [readme] Each feature must have an `ID` or `gene_id`  or `Parent` tag (referred to as `ID` henceforth).: [evidence span withheld — anchor status: external_doc]
- [readme] Chromosome identifiers (e.g. Chr1): Should match your reference annotation file chromosome identifiers; Are case sensitive: [evidence span withheld — anchor status: external_doc]
