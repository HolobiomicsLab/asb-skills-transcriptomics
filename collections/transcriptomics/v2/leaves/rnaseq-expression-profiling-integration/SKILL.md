---
name: rnaseq-expression-profiling-integration
description: Use when you have processed RNA-seq expression values for genes (e.g.
  from STAR/RSEM/DESeq2) and want them carried through a CCIVR cis-NAT identification
  run into Table.csv and Summary.csv outputs.
when_to_use_negative:
- You only have raw sequencing reads — CCIVR requires already processed RNA-seq data
  (mapping, expression profiling, and statistics done with tools such as STAR, RSEM,
  and DESeq2)
- Gene annotations lack accurate structure and strand direction in the source database,
  since CCIVR's cis-NAT prediction depends on annotation accuracy
- You need de novo transcriptome discovery rather than annotation-based identification
  of cis-NATs (e.g. the Perl-based NATpipe use case)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: CCIVR
  role: Python package that reads the annotation+expression CSV and attaches RNA-seq
    expression information to the extracted cis-NATs during the identification run
  repo: https://github.com/CCIVR/ccivr
- name: STAR
  role: Third-party program for mapping used upstream to produce the processed RNA-seq
    data CCIVR consumes
- name: RSEM
  role: Third-party program for expression profiling that generates the TPM/FPKM columns
    integrated into the CCIVR input
- name: DESeq2
  role: Statistical analysis tool that produces fold_change and padj (q-value) columns
    for the CCIVR input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:05d28cb9946413241c69c1ec3146722eedc066472d6e462b681be862486ea2c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCIVR/ccivr
  tools:
  - CCIVR
  - DESeq2
  - RSEM
  - STAR
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# rnaseq-expression-profiling-integration

## Summary

Integrate RNA-seq expression profiling data (TPM, FPKM, fold_change, padj) as extra columns of the CSV gene-annotation input consumed by CCIVR, so identified cis-NAT pairs carry expression information alongside structural characteristics (EB, FO, HH, TT).

## When to use

Use when you have processed RNA-seq expression values for genes (e.g. from STAR/RSEM/DESeq2) and want them carried through a CCIVR cis-NAT identification run into Table.csv and Summary.csv outputs.

## When NOT to use

- You only have raw sequencing reads — CCIVR requires already processed RNA-seq data (mapping, expression profiling, and statistics done with tools such as STAR, RSEM, and DESeq2)
- Gene annotations lack accurate structure and strand direction in the source database, since CCIVR's cis-NAT prediction depends on annotation accuracy
- You need de novo transcriptome discovery rather than annotation-based identification of cis-NATs (e.g. the Perl-based NATpipe use case)

## Inputs

- CSV-formatted gene dataset with five required annotation columns (id, Chr, Strand, Start, End), e.g. from the Ensembl database
- Processed RNA-seq expression values per gene: TPM, FPKM, fold_change, padj (q-value), e.g. from RSEM/DESeq2 output
- Optional extra gene annotation columns (GeneSymbol, gene_source, gene_biotype, GO_analysis)

## Outputs

- Table.csv in a new 'ccivr_output' directory: list of all extracted cis-NATs with structural characteristics (EB, FO, HH, TT) and attached expression profiles
- Summary.csv: absolute numbers and percentages of each cis-NAT type

## How to apply

First process raw RNA-seq data with third-party programs (STAR for mapping, RSEM for expression profiling, DESeq2 for statistical analysis), since CCIVR itself requires processed RNA-seq data. Build a CSV input whose five required columns are id, Chr, Strand, Start, and End (obtainable from Ensembl), then append expression columns named exactly 'TPM', 'FPKM', 'fold_change', and 'padj'; CCIVR recognizes columns by their characters, not their order, so column order is flexible. Run 'ccivr [path of input CSV]' so that expression profiling is attached during the CCIVR process and propagated to the extracted cis-NAT list. Validate against the test-file reference: total genes 65065, genes with cisNats 27351 [42.04%], EB 8112 [12.47%], FO 12194 [18.74%], HH 5992 [9.21%], TT 6097 [9.37%].

## Related tools

- **CCIVR** (Python package that reads the annotation+expression CSV and attaches RNA-seq expression information to the extracted cis-NATs during the identification run) — https://github.com/CCIVR/ccivr
- **STAR** (Third-party program for mapping used upstream to produce the processed RNA-seq data CCIVR consumes)
- **RSEM** (Third-party program for expression profiling that generates the TPM/FPKM columns integrated into the CCIVR input)
- **DESeq2** (Statistical analysis tool that produces fold_change and padj (q-value) columns for the CCIVR input)

## Examples

```
pip install git+https://github.com/CCIVR/ccivr.git; ccivr Test_file.csv --output ./ccivr_output
```

## Evaluation signals

- Console < Result > block and Summary.csv report total genes : 65065 and genes with cisNats : 27351 [42.04%] on the distributed Test_file.csv
- Per-type percentages match reference values: EB 8112 [12.47%], FO 12194 [18.74%], HH 5992 [9.21%], TT 6097 [9.37%]
- Two CSV outputs, Table.csv and Summary.csv, are written into a new directory named 'ccivr_output'
- Table.csv rows carry the appended expression columns (TPM, FPKM, fold_change, padj) for each extracted cis-NAT pair
- Input validation: CCIVR recognizes the five required columns id, Chr, Strand, Start, End regardless of their order in the CSV header

## Limitations

- CCIVR scripts do not cover the full analysis pipeline — expression profile analysis relies on processed RNA-seq data from third-party programs (STAR, RSEM, DESeq2)
- Prediction of cis-NAT composition depends on the accuracy of gene annotation, including gene structure and strand direction, deposited in databases such as Ensembl and NCBI
- Completing the whole CCIVR process with the test file takes from a few to dozens of minutes on a standard computer (6 min 34 s on an iMac 2017, 3.4 GHz Intel Core i5)

## Evidence

- [methods] Attaching information for expression profiling obtained from RNA-seq analysis is also available during the CCIVR process by adding extra columns such as **"TPM"**, **"FPKM"**, **"fold_change"**: "Attaching information for expression profiling obtained from RNA-seq analysis is also available during the CCIVR process by adding extra columns such as **"TPM"**, **"FPKM"**, **"fold_change"**,"
- [discussion] it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis: "for performing the expression profile analysis of cis-NATs by CCIVR, it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and"
- [readme] The test file contains human gene datasets accompanied by expression profiling of human hepatoma Huh-7 cells with TGFß stimulation: "(The test file contains human gene datasets accompanied by expression profiling of human hepatoma Huh-7 cells with TGFß
stimulation, which is consistent with the input file used for the CCIVR"
- [discussion] CCIVR enables functional studies of cis-NATs in both transcriptional and translational regulation: "Therefore, CCIVR enables functional studies of cis-NATs in both transcriptional and translational regulation."
