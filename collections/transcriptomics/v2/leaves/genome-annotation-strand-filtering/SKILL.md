---
name: genome-annotation-strand-filtering
description: Use when preparing a genome annotation CSV for CCIVR-style cis-NAT extraction
  and you need to restrict candidate antisense partners to same-chromosome, opposite-strand
  genes (e.g., selecting the 1,352 minus-strand X-chromosome genes for the Xist/Tsix
  FO example).
when_to_use_negative:
- The input annotation lacks strand direction or coordinates — CCIVR's predictions
  depend on the accuracy of gene annotation including structure and strand direction
  deposited in databases, so incomplete columns invalidate the filtering
- The goal is de novo transcriptome-based NAT discovery rather than annotation-based
  pair extraction (tools like NATpipe or NASTI-seq are designed for assembled transcriptomes
  or strand-specific read error rates instead)
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: CCIVR
  role: Python package that reads the strand-filtered annotation CSV, extracts EB/FO/HH/TT
    cis-NAT pairs per chromosome and strand, and writes Table.csv and Summary.csv
  repo: https://github.com/CCIVR/ccivr
- name: pandas
  role: Used to load and partition the Ensembl GRCm39 annotation CSV by Strand and
    chromosome before interval comparisons
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:f0c5bb350943afcea015babc9bcb6fef1a0d56a30ec30c0e55bfc2edf86ba7bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCIVR/ccivr
  tools:
  - CCIVR
  - pandas
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# genome-annotation-strand-filtering

## Summary

A preprocessing step in CCIVR's cis-NAT identification that partitions an Ensembl gene annotation CSV (id, Chr, Strand, Start, End) by strand and chromosome, so that opposite-strand interval comparisons (EB, FO, HH, TT) can be performed per chromosome. In the Xist/Tsix example, this reduces 55,146 mouse GRCm39 genes to 1,352 minus-strand genes on the X chromosome before FO extraction.

## When to use

Use when preparing a genome annotation CSV for CCIVR-style cis-NAT extraction and you need to restrict candidate antisense partners to same-chromosome, opposite-strand genes (e.g., selecting the 1,352 minus-strand X-chromosome genes for the Xist/Tsix FO example).

## When NOT to use

- The input annotation lacks strand direction or coordinates — CCIVR's predictions depend on the accuracy of gene annotation including structure and strand direction deposited in databases, so incomplete columns invalidate the filtering
- The goal is de novo transcriptome-based NAT discovery rather than annotation-based pair extraction (tools like NATpipe or NASTI-seq are designed for assembled transcriptomes or strand-specific read error rates instead)

## Inputs

- CSV-formatted Ensembl gene annotation file with five required columns: id, Chr, Strand, Start, End (e.g., mouse GRCm39, 55,146 genes); optional extra columns (GeneSymbol, gene_biotype, TPM, FPKM, fold_change, padj)

## Outputs

- CSV-formatted table (Table.csv) listing extracted cis-NAT pairs with structural type labels (EB, FO, HH, TT), gene ids, chromosome, strands, and coordinates
- Optionally, an intermediate per-strand/per-chromosome gene subset (e.g., 1,352 minus-strand X-chromosome genes)

## How to apply

Load the Ensembl GRCm39 annotation (55,146 mouse genes) with pandas, requiring the five CCIVR-recognized columns: id, Chr, Strand, Start, End (column order is changeable because CCIVR matches column names, not positions). Partition the table by Strand and, for each target gene (e.g., minus-strand Xist), subset to same-strand genes on the target gene's chromosome — in the Fig. 1C example this yields 1,352 minus-strand genes on the X chromosome. For each filtered gene, compare its Start-End interval against same-chromosome opposite-strand genes and apply the appropriate CCIVR overlap criterion (for the from-minus-to-plus FO process: one partner's transcribed region completely contains the other's on the opposite strand). Record matched pairs with gene ids, chromosome, strands, and coordinates, then write the result to a CSV-formatted table; verify the Xist/Tsix pair appears as an opposite-strand fully-overlapped pair on the same chromosome.

## Related tools

- **CCIVR** (Python package that reads the strand-filtered annotation CSV, extracts EB/FO/HH/TT cis-NAT pairs per chromosome and strand, and writes Table.csv and Summary.csv) — https://github.com/CCIVR/ccivr
- **pandas** (Used to load and partition the Ensembl GRCm39 annotation CSV by Strand and chromosome before interval comparisons)

## Examples

```
ccivr /path/to/Ensembl_GRCm39_genes.csv --output /path/to/ccivr_output
```

## Evaluation signals

- The output is a CSV-formatted table and, for a from-minus-to-plus FO run on mouse GRCm39, contains the Xist/Tsix pair as an opposite-strand, same-chromosome fully-overlapped pair
- Per-chromosome strand subsets are consistent with the annotation: e.g., 1,352 minus-strand genes recovered on the X chromosome for the Xist example
- Column schema check: the input contains exactly the five CCIVR-recognized column names (id, Chr, Strand, Start, End), in any order
- Summary.csv reports absolute counts and percentages of EB, FO, HH, and TT cis-NATs that sum consistently with Table.csv pair counts
- Every reported pair's two genes are on the same Chr value with opposite Strand values, and their Start-End intervals satisfy the declared overlap criterion (containment for FO)

## Limitations

- CCIVR depends on the accuracy of gene annotation (structure and strand direction) deposited in databases such as Ensembl and NCBI
- CCIVR scripts do not cover the full analysis pipeline: RNA-seq data must be pre-processed with third-party programs (STAR for mapping, RSEM for expression profiling, DESeq2 for statistics)

## Evidence

- [results] In the from-minus-to-plus FO extraction, CCIVR first selects minus-strand genes on the same chromosome as the target (1,352 genes for Xist's X chromosome): [evidence span withheld — anchor status: not_found]
- [methods] Input file schema: "The input file must contain five different gene annotations listed in their independent columns"
- [methods] FO criterion: [evidence span withheld — anchor status: not_found]
- [readme] Output files: "Two CSV formatted output files **"Table.csv"** and **"Summary.csv"** will be generated and saved in a new directory named **"ccivr_output"**"
- [readme] Column name matching: "The order of these five columns is changeable because CCIVR recognizes their characters"
