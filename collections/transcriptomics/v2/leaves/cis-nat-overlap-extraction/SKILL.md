---
name: cis-nat-overlap-extraction
description: Use when a gene annotation (CSV with id, Chr, Strand, Start, End) must
  be mined for opposite-strand cis-NAT pairs, especially fully-overlapped (FO) pairs
  where one gene's transcribed interval completely contains its partner's.
when_to_use_negative:
- Input is already a CCIVR output file (Table.csv or Summary.csv) rather than a five-column
  gene annotation input
- Raw RNA-seq FASTQ must be mapped and quantified first — CCIVR consumes processed
  RNA-seq data from third-party programs such as STAR, RSEM, and DESeq2
- Detection must operate on de novo assembled transcriptomes (NATpipe-style) or strand-specific
  error-rate-aware detection (NASTI-seq-style) rather than genome annotation coordinates
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: CCIVR
  role: Python package that performs the EB/FO/HH/TT overlap extraction, including
    Plus-to-minus and Minus-to-plus extracting passes, from a five-column CSV annotation
  repo: https://github.com/CCIVR/ccivr.git
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:a31c28a86ae22337521c03d6992a879b839ea76d8f41fa64b930892a5fe2d375
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCIVR/ccivr.git
  tools:
  - CCIVR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cis-nat-overlap-extraction

## Summary

Extraction of cis-natural antisense transcript (cis-NAT) pairs from a genome annotation by overlap classification (EB, FO, HH, TT), reproducing CCIVR's fully-overlapped (FO) minus-to-plus/plus-to-minus logic, e.g. to recover the mouse Xist/Tsix pair from Ensembl GRCm39.

## When to use

Use when a gene annotation (CSV with id, Chr, Strand, Start, End) must be mined for opposite-strand cis-NAT pairs, especially fully-overlapped (FO) pairs where one gene's transcribed interval completely contains its partner's.

## When NOT to use

- Input is already a CCIVR output file (Table.csv or Summary.csv) rather than a five-column gene annotation input
- Raw RNA-seq FASTQ must be mapped and quantified first — CCIVR consumes processed RNA-seq data from third-party programs such as STAR, RSEM, and DESeq2
- Detection must operate on de novo assembled transcriptomes (NATpipe-style) or strand-specific error-rate-aware detection (NASTI-seq-style) rather than genome annotation coordinates

## Inputs

- CSV-formatted gene annotation with columns id, Chr, Strand, Start, End (e.g., Ensembl GRCm39 mouse gene dataset)
- Optional extra annotation columns (e.g., GeneSymbol, gene_source, gene_biotype, GO_analysis)
- Optional RNA-seq expression columns (TPM, FPKM, fold_change, padj)

## Outputs

- CSV table of extracted cis-NAT pairs labeled by structural type (EB, FO, HH, TT) with gene ids, chromosome, strands, and Start/End coordinates (CCIVR Table.csv format)
- CSV summary of absolute number and percentage of each cis-NAT type (CCIVR Summary.csv format)

## How to apply

Load a CSV-formatted gene annotation (e.g., Ensembl GRCm39 mouse, 55,146 genes) requiring the five CCIVR columns: id, Chr, Strand, Start, End (column order is irrelevant since CCIVR recognizes the header names). Partition genes by strand and chromosome; for the minus-to-plus FO extraction, first select minus-strand genes on the target chromosome (e.g., 1,352 genes on the X chromosome for the Xist example), then compare each minus-strand gene's Start-End interval against plus-strand genes on the same chromosome, keeping pairs where one partner's transcribed region completely contains the other's on the opposite strand. Record each matched pair as 'FO' with gene ids, chromosome, strands, and coordinates, verifying known pairs such as Xist/Tsix are recovered; run the same logic in the plus-to-minus direction to complete the FO pass (CCIVR also performs EB, HH, and TT passes). Optionally write the FO cis-NAT list to a CSV table file, mirroring CCIVR's Table.csv output, and summarize counts/percentages per type as in Summary.csv.

## Related tools

- **CCIVR** (Python package that performs the EB/FO/HH/TT overlap extraction, including Plus-to-minus and Minus-to-plus extracting passes, from a five-column CSV annotation) — https://github.com/CCIVR/ccivr.git

## Examples

```
ccivr [path of input CSV] --output [path of saving directory]
```

## Evaluation signals

- Output CSV exists in CCIVR Table.csv format and contains the Xist/Tsix pair as an opposite-strand fully-overlapped pair on the same chromosome
- Pre-filter minus-strand gene count on the target chromosome matches the reference (1,352 genes on the mouse X chromosome in the GRCm39 Fig. 1C example)
- Every reported FO pair satisfies complete containment: one partner's Start-End interval fully encloses the other's on the opposite strand
- Running ccivr displays the expected processing status lines (EB/FO/HH/TT, each with 'Plus-to-minus extracting' and 'Minus-to-plus extracting') before writing results
- Summary.csv per-type counts and percentages are consistent with the rows labeled EB, FO, HH, TT in Table.csv

## Limitations

- Prediction accuracy depends on the deposited gene annotation, including gene structure and strand direction, in the source database (e.g., Ensembl)
- CCIVR scripts do not cover the full pipeline: expression profiling relies on third-party programs (STAR for mapping, RSEM for expression, DESeq2 for statistics)
- Runtime can be substantial: the 65,065-gene test file takes 6 minutes 34 seconds on an iMac 2017 with a 3.4 GHz Intel Core i5

## Evidence

- [readme] a table file containing a list of total extracted cis-NATs accompanied by their structural characteristics, including embedded (EB), fully-overlapped (FO): "a table file containing a list of total extracted cis-NATs accompanied by their structural characteristics, including embedded (EB), fully-overlapped (FO)"
- [methods] The input file must contain five different gene annotations listed in their independent columns, as follows:: "The input file must contain five different gene annotations listed in their independent columns, as follows:"
- [readme] Two CSV formatted output files **"Table.csv"** and **"Summary.csv"** will be generated and saved in a new directory named **"ccivr_output"**.: "Two CSV formatted output files **"Table.csv"** and **"Summary.csv"** will be generated and saved in a new directory named **"ccivr_output"**."
- [readme] Plus-to-minus extracting
Minus-to-plus extracting: "Plus-to-minus extracting
Minus-to-plus extracting"
- [readme] The example above is for the murine *Tsix* gene, from Ensembl GRCm39): "The example above is for the murine *Tsix* gene, from Ensembl GRCm39)"
- [discussion] For predicting the composition of various cis-NATs by CCIVR, it depends on the accuracy of gene annotation including their structure and strand direction: "For predicting the composition of various cis-NATs by CCIVR, it depends on the accuracy of gene annotation including their structure and strand direction"
