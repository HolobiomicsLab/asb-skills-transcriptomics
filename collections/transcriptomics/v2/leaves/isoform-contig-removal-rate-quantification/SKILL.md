---
name: isoform-contig-removal-rate-quantification
description: Use when you have cloudrnaSPAdes assembled contigs (FASTA) plus a barcode-to-gene
  assignment table and need to measure what fraction of ambiguous barcode assignments
  a filtering script removes.
when_to_use_negative:
- Input contigs have no barcode-to-gene assignment table, so unique vs ambiguous classification
  cannot be performed
- Goal is read-level filtering (e.g. k-mer coverage read filtering with spades-read-filter)
  rather than post-assembly contig removal by barcode multiplicity
- Evaluation is against simulated data without a fixed reference table (e.g. Table
  3 rows) to compare removal rates against
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0102
tools:
- name: cloudrnaSPAdes
  role: Produces the barcoded RNA-seq isoform assembly contigs whose ambiguous barcode
    assignments are filtered and quantified
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: SPAdes
  role: Underlying assembly toolkit whose cloudrnaSPAdes feature release generates
    the contigs FASTA input
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:8307473e3a92121407d4ede8cbba987e0d14799093816f69532a2199d47b20b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - SPAdes
  - cloudrnaSPAdes
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# isoform-contig-removal-rate-quantification

## Summary

Quantifies how many ambiguously barcode-assigned isoform contigs from a cloudrnaSPAdes real-data assembly are removed by the paper's barcode-based filtering step (contigs whose barcode is assigned to a single gene multiple times), reproducing the ~15% difference between the '1' and '1-fixed' rows of Table 3.

## When to use

Use when you have cloudrnaSPAdes assembled contigs (FASTA) plus a barcode-to-gene assignment table and need to measure what fraction of ambiguous barcode assignments a filtering script removes.

## When NOT to use

- Input contigs have no barcode-to-gene assignment table, so unique vs ambiguous classification cannot be performed
- Goal is read-level filtering (e.g. k-mer coverage read filtering with spades-read-filter) rather than post-assembly contig removal by barcode multiplicity
- Evaluation is against simulated data without a fixed reference table (e.g. Table 3 rows) to compare removal rates against

## Inputs

- Assembled real-data contigs FASTA from cloudrnaSPAdes (e.g. SPIso-seq assembly)
- Barcode-to-gene assignment table for the same data set

## Outputs

- Filtered contigs FASTA containing retained contigs after barcode-based removal of ambiguous assignments
- CSV of removed/retained counts and removal rates per assignment category (barcode_filtering_rates.csv)

## How to apply

Load the assembled real-data contigs FASTA produced by cloudrnaSPAdes together with the barcode-to-gene assignment table for the same data set. For each contig, classify its barcode assignment as unique (barcode maps to a single gene once) or ambiguous (barcode assigned to a single gene multiple times). Run the paper's barcode-filtering step, which removes all contigs whose barcode is assigned to a single gene multiple times, and write the retained contigs to a filtered FASTA. Count removed and retained contigs in each category and compute the removal rate (percentage) separately for ambiguous and unique assignments; unique assignments should remain relatively unchanged while roughly 15% of ambiguous assignments should be removed.

## Related tools

- **cloudrnaSPAdes** (Produces the barcoded RNA-seq isoform assembly contigs whose ambiguous barcode assignments are filtered and quantified) — https://github.com/1dayac/cloudrnaSPAdes-release
- **SPAdes** (Underlying assembly toolkit whose cloudrnaSPAdes feature release generates the contigs FASTA input) — https://github.com/1dayac/cloudrnaSPAdes-release

## Examples

```
python barcode_filter.py --contigs cloudrna_assembly/contigs.fasta --barcode_assignments barcode_to_gene.tsv --out filtered_contigs.fasta --rates barcode_filtering_rates.csv
```

## Evaluation signals

- Ambiguous-assignment removal rate reproduces the ~15% filtering difference between the '1' and '1-fixed' rows of Table 3
- Unique-assignment counts remain relatively unchanged after filtering
- barcode_filtering_rates.csv contains separate removal rates for ambiguous and unique assignment categories
- Filtered FASTA contains only contigs whose barcode is NOT assigned to a single gene multiple times

## Limitations

- The ~15% removal benchmark is specific to the real SPIso-seq data set and Table 3 fixed rows; other data sets may show different rates
- Filtering removes all contigs sharing an ambiguous barcode, so correctly assigned contigs with multiply-assigned barcodes are also discarded
- The skill quantifies removal of ambiguous assignments but does not itself improve assembly accuracy of retained contigs

## Evidence

- [results] Applying the barcode-based filtering between the '1' and '1-fixed' rows of Table 3 removes approximately 15% of the ambiguous assignments, while unique assignments remain relatively unchanged.: [evidence span withheld — anchor status: not_found]
- [abstract] cloudrnaSPAdes assembles isoforms from barcoded RNA-seq linked-read data: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [intro] SPAdes is a general assembly toolkit with multiple pipelines: "SPAdes &ndash; St. Petersburg genome assembler &ndash; is an assembly toolkit containing various assembly pipelines"
