---
name: barcode-gene-assignment-classification
description: Use when cloudrnaSPAdes contigs from barcoded (SPIso-seq-style) data
  must have their barcode-to-gene assignments classified as unique vs ambiguous, or
  when quantifying the fraction of ambiguous contigs removable by barcode-multiplicity
  filtering (e.g. Table 3 '1' vs '1-fixed' rows).
when_to_use_negative:
- Input is raw reads rather than assembled contigs — this skill operates post-assembly,
  not at the read level (contrast with spades-read-filter, which filters reads by
  median k-mer coverage).
- Barcodes map to multiple distinct genes rather than to a single gene multiple times
  — the paper's filter targets only the 'assigned to a single gene multiple times'
  ambiguity case.
- Data are non-barcoded standard RNA-seq contigs with no barcode-to-gene assignment
  table available.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0196
tools:
- name: cloudrnaSPAdes
  role: assembles the full-length isoform contigs whose barcode-to-gene assignments
    are classified and filtered by this skill
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: SPAdes
  role: underlying assembly toolkit whose feature release cloudrnaSPAdes produces
    the contigs input to this skill
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:95daf6becb41638aacbf8bb01cfbe2280b43a2e373827197fcb3386eb2a677ae
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/1dayac/cloudrnaSPAdes-release
  tools:
  - SPAdes
  - cloudrnaSPAdes
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# barcode-gene-assignment-classification

## Summary

Classifies barcode-to-gene assignments for contigs assembled from barcoded RNA-seq linked-read data as unique (barcode maps to a single gene once) or ambiguous (barcode assigned to a single gene multiple times), then quantifies how many ambiguous contigs a barcode-multiplicity filter removes.

## When to use

Use when cloudrnaSPAdes contigs from barcoded (SPIso-seq-style) data must have their barcode-to-gene assignments classified as unique vs ambiguous, or when quantifying the fraction of ambiguous contigs removable by barcode-multiplicity filtering (e.g. Table 3 '1' vs '1-fixed' rows).

## When NOT to use

- Input is raw reads rather than assembled contigs — this skill operates post-assembly, not at the read level (contrast with spades-read-filter, which filters reads by median k-mer coverage).
- Barcodes map to multiple distinct genes rather than to a single gene multiple times — the paper's filter targets only the 'assigned to a single gene multiple times' ambiguity case.
- Data are non-barcoded standard RNA-seq contigs with no barcode-to-gene assignment table available.

## Inputs

- assembled contigs FASTA from cloudrnaSPAdes on the real SPIso-seq data set
- barcode-to-gene assignment table (barcode, gene) for the same data set

## Outputs

- filtered FASTA containing only contigs with unique barcode-to-gene assignments
- barcode_filtering_rates.csv with removed/retained counts and removal-rate percentages per assignment category (ambiguous vs unique)

## How to apply

Load the assembled contigs FASTA produced by cloudrnaSPAdes for the real SPIso-seq data set together with the barcode-to-gene assignment table for the same data set. For each contig, look up its barcode and classify the assignment as unique (barcode maps to a single gene exactly once) or ambiguous (barcode assigned to a single gene multiple times). Apply the paper's barcode-filtering step, which removes all contigs whose barcode is assigned to a single gene multiple times, and write retained contigs to a filtered FASTA. Count removed vs retained contigs per category and compute removal rates (percentages) separately for ambiguous and unique assignments, writing the counts to barcode_filtering_rates.csv. The rationale is that multiplicity in barcode-to-gene assignment signals chimeric or mis-assembled isoforms, so the filter should preferentially remove ambiguous assignments (~15% in Table 3) while leaving unique assignments relatively unchanged.

## Related tools

- **cloudrnaSPAdes** (assembles the full-length isoform contigs whose barcode-to-gene assignments are classified and filtered by this skill) — https://github.com/1dayac/cloudrnaSPAdes-release
- **SPAdes** (underlying assembly toolkit whose feature release cloudrnaSPAdes produces the contigs input to this skill) — https://github.com/1dayac/cloudrnaSPAdes-release

## Evaluation signals

- Ambiguous-assignment removal rate reproduces the ~15% difference between the '1' and '1-fixed' rows of Table 3.
- Counts of unique assignments are relatively unchanged before vs after filtering (removal rate near zero).
- The filtered FASTA contains only contigs whose barcode maps to a single gene exactly once; no ambiguous barcodes remain.
- Removed + retained contig counts in barcode_filtering_rates.csv sum to the total contig count in the input FASTA per category.
- Removal rates are reported separately for ambiguous and unique categories, not as a single pooled percentage.

## Limitations

- The barcode-multiplicity filter removes only approximately 15% of ambiguous assignments, so a large majority of ambiguous contigs are retained.
- The filter criterion is based entirely on the barcode-to-gene assignment table; errors in that table propagate directly into the classification.
- Unique assignments are assumed to be unaffected by filtering, so no correction is applied to the unique category.

## Evidence

- [abstract] we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [results] Applying the barcode-based filtering between the '1' and '1-fixed' rows of Table 3 removes approximately 15% of the ambiguous assignments: [evidence span withheld — anchor status: not_found]
- [methods] spades-read-filter is a tool for filtering reads with median kmer coverage less than threshold.: "spades-read-filter is a tool for filtering reads with median kmer coverage less than threshold."
