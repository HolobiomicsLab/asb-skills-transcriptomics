---
name: barcoded-linked-read-contig-filtering
description: Use when you have a cloudrnaSPAdes contigs FASTA from barcoded (SPIso-seq)
  RNA-seq data plus a barcode-to-gene assignment table, and you need to remove contigs
  with multiply-assigned barcodes and quantify the removal rate per assignment class.
when_to_use_negative:
- Input is read-level filtering of FASTQ by k-mer coverage — that is a different task
  handled by spades-read-filter ('filtering reads with median kmer coverage less than
  threshold'), not barcode-based contig filtering
- Contigs lack barcode-to-gene assignments (e.g. assembly of non-barcoded RNA-seq
  data), so ambiguous vs unique classification is impossible
- The question is whether a barcode maps to multiple different genes rather than to
  a single gene multiple times — this skill's ambiguity criterion only covers the
  latter
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: cloudrnaSPAdes
  role: Assembler that produced the input contigs FASTA from barcoded RNA-seq linked-read
    data; its output is the object being filtered
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
- name: SPAdes
  role: Parent assembly toolkit of which cloudrnaSPAdes is a feature release; supplies
    the assembly pipeline whose contigs are post-processed
  repo: https://github.com/1dayac/cloudrnaSPAdes-release
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad781
    title: '<scp>cloudrna</scp> SP <scp>Ades</scp> : isoform assembly using bulk barcoded
      RNA sequencing data'
  source_content_hash: sha256:21a5baa189b4e6b86495c16e66d46fddbd24ef5743bd9ce2ee03dd078fbb9230
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# barcoded-linked-read-contig-filtering

## Summary

Post-assembly filtering step for cloudrnaSPAdes results that removes contigs whose barcode is assigned to a single gene multiple times (ambiguous assignments), quantifying how many such contigs are removed versus uniquely assigned contigs retained. This reproduces the paper's 'fixed' Table 3 correction, where the barcode filter removes ~15% of ambiguous assignments on real SPIso-seq data.

## When to use

Use when you have a cloudrnaSPAdes contigs FASTA from barcoded (SPIso-seq) RNA-seq data plus a barcode-to-gene assignment table, and you need to remove contigs with multiply-assigned barcodes and quantify the removal rate per assignment class.

## When NOT to use

- Input is read-level filtering of FASTQ by k-mer coverage — that is a different task handled by spades-read-filter ('filtering reads with median kmer coverage less than threshold'), not barcode-based contig filtering
- Contigs lack barcode-to-gene assignments (e.g. assembly of non-barcoded RNA-seq data), so ambiguous vs unique classification is impossible
- The question is whether a barcode maps to multiple different genes rather than to a single gene multiple times — this skill's ambiguity criterion only covers the latter

## Inputs

- Assembled real-data contigs FASTA produced by cloudrnaSPAdes (from barcoded SPIso-seq data)
- Barcode-to-gene assignment table for the same data set (barcode ID to gene ID mapping)

## Outputs

- Filtered FASTA containing retained contigs (those whose barcode is not assigned to a single gene multiple times)
- barcode_filtering_rates.csv with removed/retained counts and removal-rate percentages for ambiguous and unique assignments

## How to apply

Load the assembled real-data contigs FASTA produced by cloudrnaSPAdes together with the barcode-to-gene assignment table for the same SPIso-seq data set. For each contig, classify its barcode assignment as unique (barcode maps to a single gene once) or ambiguous (barcode assigned to a single gene multiple times) — the ambiguity criterion is multiplicity of assignment to one gene, not mapping to multiple genes. Run the paper's barcode-filtering step, deleting all contigs whose barcode is assigned to a single gene multiple times, and write retained contigs to a filtered FASTA. Count removed and retained contigs in each category and compute the removal percentage separately for ambiguous and unique assignments; the rationale is that multiply-assigned barcodes mark ambiguous isoform-origin contigs, so pruning them 'fixes' the assignment statistics (the difference between the '1' and '1-fixed' rows of Table 3).

## Related tools

- **cloudrnaSPAdes** (Assembler that produced the input contigs FASTA from barcoded RNA-seq linked-read data; its output is the object being filtered) — https://github.com/1dayac/cloudrnaSPAdes-release
- **SPAdes** (Parent assembly toolkit of which cloudrnaSPAdes is a feature release; supplies the assembly pipeline whose contigs are post-processed) — https://github.com/1dayac/cloudrnaSPAdes-release

## Examples

```
python barcode_filter.py --contigs contigs.fasta --assignments barcode_to_gene.tsv --filtered-out contigs_fixed.fasta --rates barcode_filtering_rates.csv
```

## Evaluation signals

- Ambiguous-assignment removal rate reproduces the ~15% filtering difference between the '1' and '1-fixed' rows of Table 3
- Unique assignments remain relatively unchanged (removal rate near zero) after filtering
- Every contig in the filtered FASTA has a barcode that is assigned to its gene at most once in the assignment table
- Per-category removed + retained counts sum to the total contigs in each class of the input FASTA
- barcode_filtering_rates.csv contains separate removal-rate percentages for ambiguous and unique assignments, not only a global rate

## Limitations

- Filtering removes only ~15% of ambiguous assignments, so the majority of ambiguous assignments are not resolved by this barcode-based rule
- Unique assignments are largely unaffected, so the filter corrects ambiguity statistics but does not improve unique-assignment counts
- Validation is grounded in the real SPIso-seq assembly data set (Table 3 fixed rows); the paper's discussion section content and changelog are absent from available sources, so broader applicability is unverified

## Evidence

- [abstract] cloudrnaSPAdes assembles isoforms from barcoded linked reads: "we present cloudrnaSPAdes, a tool for assembling full-length isoforms from barcoded RNA-seq linked-read data in a reference-free fashion"
- [results] Barcode filtering removes ~15% of ambiguous assignments: [evidence span withheld — anchor status: not_found]
- [methods] Ambiguity criterion is a barcode assigned to a single gene multiple times: [evidence span withheld — anchor status: not_found]
