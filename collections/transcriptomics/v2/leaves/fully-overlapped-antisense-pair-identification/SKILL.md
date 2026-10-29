---
name: fully-overlapped-antisense-pair-identification
description: Use when you have a CSV gene annotation with id, Chr, Strand, Start,
  End columns and need to classify opposite-strand overlaps as fully-overlapped (FO)
  cis-NAT pairs, e.g. reproducing the Xist/Tsix example.
when_to_use_negative:
- The annotation lacks reliable strand direction or the five required columns — CCIVR
  explicitly depends on the accuracy of deposited annotation including strand direction
- The goal is de novo discovery of NATs from raw RNA-seq reads rather than annotation-based
  structural classification — annotation-free pipelines like NATpipe or NASTI-seq
  are the appropriate family
- You need embedded (EB), head-to-head (HH), or tail-to-tail (TT) classification only;
  the FO containment criterion is a distinct overlap geometry
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0622
tools:
- name: CCIVR
  role: Runs the EB/FO/HH/TT extraction; its 'Minus-to-plus extracting' FO pass pairs
    minus-strand genes with completely containing plus-strand genes on the same chromosome
  repo: github.com/CCIVR/ccivr
- name: pandas
  role: Used to load the Ensembl GRCm39 annotation CSV and partition/filter genes
    by strand and chromosome before applying the FO criterion
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:083f9c599af6c8141fe82aad5a7553d7909dd86f8ab6c11374b3d401ee52dc26
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CCIVR
  - pandas
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# fully-overlapped-antisense-pair-identification

## Summary

Extracts fully-overlapped (FO) cis-NAT pairs — opposite-strand gene pairs on the same chromosome where one partner's transcribed region completely contains the other's — using the CCIVR 'from-minus-to-plus' FO extraction logic, demonstrated on Ensembl GRCm39 mouse annotation recovering the Xist/Tsix pair.

## When to use

Use when you have a CSV gene annotation with id, Chr, Strand, Start, End columns and need to classify opposite-strand overlaps as fully-overlapped (FO) cis-NAT pairs, e.g. reproducing the Xist/Tsix example.

## When NOT to use

- The annotation lacks reliable strand direction or the five required columns — CCIVR explicitly depends on the accuracy of deposited annotation including strand direction
- The goal is de novo discovery of NATs from raw RNA-seq reads rather than annotation-based structural classification — annotation-free pipelines like NATpipe or NASTI-seq are the appropriate family
- You need embedded (EB), head-to-head (HH), or tail-to-tail (TT) classification only; the FO containment criterion is a distinct overlap geometry

## Inputs

- CSV-formatted gene annotation file with required columns id, Chr, Strand, Start, End (e.g. Ensembl GRCm39 mouse gene table, 55,146 genes)
- Optional extra annotation columns (GeneSymbol, gene_source, gene_biotype, GO_analysis) and expression columns (TPM, FPKM, fold_change, padj)

## Outputs

- Table.csv — list of extracted cis-NAT pairs with structural type label 'FO' plus gene ids, chromosome, strands, and coordinates
- Summary.csv — absolute number and percentage of each cis-NAT type (EB, FO, HH, TT) relative to total genes

## How to apply

Load the gene annotation CSV (e.g. Ensembl GRCm39 mouse, 55,146 genes) requiring the five CCIVR columns id, Chr, Strand, Start, End, and partition genes by strand per chromosome. In the from-minus-to-plus FO pass, first select minus-strand genes on the target chromosome (1,352 on the X chromosome in the Xist example), then compare each minus-strand gene's Start–End interval against plus-strand genes on the same chromosome and apply the CCIVR FO criterion: one partner's transcribed region completely contains the other's on the opposite strand. Record each matched pair with gene ids, chromosome, strands and coordinates, labeling the type 'FO' as CCIVR does in Table.csv (the run prints 'FO / Plus-to-minus extracting / Minus-to-plus extracting' status lines). The counterpart plus-to-minus pass should yield the reciprocal pairings; counts and percentages of FO pairs land in Summary.csv (e.g. FO 12,194 [18.74%] in the README test-file example).

## Related tools

- **CCIVR** (Runs the EB/FO/HH/TT extraction; its 'Minus-to-plus extracting' FO pass pairs minus-strand genes with completely containing plus-strand genes on the same chromosome) — github.com/CCIVR/ccivr
- **pandas** (Used to load the Ensembl GRCm39 annotation CSV and partition/filter genes by strand and chromosome before applying the FO criterion)

## Examples

```
ccivr mouse_GRCm39_genes.csv --output ccivr_output
```

## Evaluation signals

- Output Table.csv exists in CSV format and contains the Xist/Tsix pair as an opposite-strand fully-overlapped pair on the same chromosome
- Every row labeled 'FO' satisfies the containment invariant: one partner's Start–End interval completely contains the other's, with opposite Strand values on the same Chr
- Intermediate count of minus-strand genes on the target chromosome matches the reference example (1,352 on the X chromosome for Xist)
- Summary.csv FO counts/percentages are plausible against the README test-file reference (FO 12,194 [18.74%] of total genes)
- Both directional passes (plus-to-minus and minus-to-plus) complete, matching the CCIVR status output 'FO / Plus-to-minus extracting / Minus-to-plus extracting'

## Limitations

- CCIVR prediction of cis-NAT composition depends on the accuracy of gene annotation, including structure and strand direction, deposited in databases (e.g. Ensembl)
- CCIVR scripts do not cover the full analysis pipeline; expression profiling relies on third-party programs (STAR for mapping, RSEM for expression, DESeq2 for statistics)
- Identification is restricted to cis (same-chromosome) sense/antisense pairs; trans interactions are out of scope

## Evidence

- [readme] a table file containing a list of total extracted cis-NATs accompanied by their structural characteristics, including embedded (EB), fully-overlapped (FO): "a table file containing a list of total extracted cis-NATs accompanied by their structural characteristics, including embedded (EB), fully-overlapped (FO)"
- [methods] The input file must contain five different gene annotations listed in their independent columns: "The input file must contain five different gene annotations listed in their independent columns"
- [methods] FO
Plus-to-minus extracting
Minus-to-plus extracting: "FO
Plus-to-minus extracting
Minus-to-plus extracting"
- [results] In the from-minus-to-plus FO extraction, CCIVR first selects minus-strand genes on the same chromosome as the target (1,352 genes for Xist's X chromosome): [evidence span withheld — anchor status: not_found]
