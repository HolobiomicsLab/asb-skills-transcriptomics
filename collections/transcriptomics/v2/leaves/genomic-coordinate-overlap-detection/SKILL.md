---
name: genomic-coordinate-overlap-detection
description: Use when you have CSV gene annotations (id, Chr, Strand, Start, End)
  for a genome and need to enumerate all antisense gene pairs whose loci overlap,
  stratified by overlap structure.
when_to_use_negative:
- The task requires strand-specific transcript-level overlap detection from RNA-seq
  reads rather than genome-annotation-based gene-pair extraction (CCIVR depends on
  gene annotation accuracy, not read-level evidence)
- You need expression-profile statistics computed end-to-end — CCIVR's scripts do
  not cover the full pipeline and expect processed RNA-seq data from third-party programs
  (e.g., STAR for mapping, RSEM for expression, DESeq2 for statistics)
- The gene annotation lacks reliable strand direction, since overlap classification
  (HH vs TT, EB, FO) depends on correct strand assignment
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CCIVR
  role: Primary tool that performs Plus-to-minus and Minus-to-plus overlap extraction
    for EB, FO, HH, and TT cis-NAT types from CSV gene annotations
  repo: https://github.com/CCIVR/ccivr
- name: BEDTools
  role: Alternative C++ tool that allows identification of overlapped cis-NAT pairs
    based on genome annotation
- name: Ensembl
  role: Source database of the gene annotations converted into CCIVR's CSV input format
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:b4dbf4a6993f23a5814e7d9e12ec66c19334f189ee1c7800d05544538edbef31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCIVR/ccivr
  tools:
  - BEDTools
  - CCIVR
  - Ensembl
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# genomic-coordinate-overlap-detection

## Summary

Detects cis-natural antisense transcript (cis-NAT) pairs by testing genomic coordinate overlaps between genes on opposite strands, classifying each pair as embedded (EB), fully-overlapped (FO), head-to-head (HH), or tail-to-tail (TT). As implemented in CCIVR, it consumes CSV gene annotations and produces per-pair tables plus per-type counts and percentages.

## When to use

Use when you have CSV gene annotations (id, Chr, Strand, Start, End) for a genome and need to enumerate all antisense gene pairs whose loci overlap, stratified by overlap structure.

## When NOT to use

- The task requires strand-specific transcript-level overlap detection from RNA-seq reads rather than genome-annotation-based gene-pair extraction (CCIVR depends on gene annotation accuracy, not read-level evidence)
- You need expression-profile statistics computed end-to-end — CCIVR's scripts do not cover the full pipeline and expect processed RNA-seq data from third-party programs (e.g., STAR for mapping, RSEM for expression, DESeq2 for statistics)
- The gene annotation lacks reliable strand direction, since overlap classification (HH vs TT, EB, FO) depends on correct strand assignment

## Inputs

- CSV-formatted gene annotation file with five required columns (id, Chr, Strand, Start, End), columns in any order
- Optional extra annotation columns (GeneSymbol, gene_source, gene_biotype, GO_analysis)
- Optional RNA-seq expression columns (TPM, FPKM, fold_change, padj)
- Ensembl GTF gene annotations (Ensembl release 105/104; Ensembl plants release 52; Ensembl fungi release 52)

## Outputs

- Table.csv: list of all extracted cis-NAT pairs with structural characteristics (EB, FO, HH, TT)
- Summary.csv: absolute count and percentage of each cis-NAT type relative to total genes
- Stdout report: total genes, genes with cisNats count and percentage, per-type counts and percentages

## How to apply

Prepare a CSV input with the five required columns — id, Chr, Strand, Start, End — where Start is the minimum and End the maximum gene location; column order is flexible because CCIVR recognizes the column characters. Convert Ensembl GTF annotations (e.g., Ensembl release 105/104 or Ensembl plants/fungi release 52) into this format. Install the tool with 'pip install git+https://github.com/CCIVR/ccivr.git' and verify with 'ccivr --help'. Run 'ccivr [input CSV] --output [dir]'; the program performs Plus-to-minus and Minus-to-plus extraction for each of the EB, FO, HH, and TT structural types in sequence. Parse the generated Table.csv (all extracted cis-NAT pairs with structural characteristics) and Summary.csv (absolute number and percentage of each cis-NAT type relative to total genes). Optionally attach extra columns such as TPM, FPKM, fold_change, or padj to carry RNA-seq expression profiling through the same run.

## Related tools

- **CCIVR** (Primary tool that performs Plus-to-minus and Minus-to-plus overlap extraction for EB, FO, HH, and TT cis-NAT types from CSV gene annotations) — https://github.com/CCIVR/ccivr
- **BEDTools** (Alternative C++ tool that allows identification of overlapped cis-NAT pairs based on genome annotation)
- **Ensembl** (Source database of the gene annotations converted into CCIVR's CSV input format)

## Examples

```
pip install git+https://github.com/CCIVR/ccivr.git; ccivr [path of input CSV] --output [path of saving directory]
```

## Evaluation signals

- Human test file (Huh-7 with TGFβ stimulation) should reproduce: total genes 65065; genes with cisNats 27351 [42.04%]; EB 8112 [12.47%]; FO 12194 [18.74%]; HH 5992 [9.21%]; TT 6097 [9.37%]
- Both output files (Table.csv and Summary.csv) are written into a new directory named 'ccivr_output'
- Summary.csv percentages of each cis-NAT type are computed relative to total genes in the input CSV, not relative to cis-NAT-containing genes
- Across the 11 model organisms, per-species counts and percentages should agree with the values plotted in Fig. 2 of Ohhata et al., Sci Rep. 12:15525, 2022
- Run completes after the four sequential extraction blocks (EB, FO, HH, TT, each with Plus-to-minus and Minus-to-plus passes); expect a few to dozens of minutes runtime (6 min 34 s for the test file on an iMac 2017 with a 3.4 GHz Intel Core i5)

## Limitations

- Predicting the composition of various cis-NATs by CCIVR depends on the accuracy of gene annotation, including their structure and strand direction, as deposited in the databases
- CCIVR scripts do not cover the full analysis pipeline; expression profile analysis relies on processed RNA-seq data from third-party programs such as STAR, RSEM, and DESeq2
- Installation via 'pip install git+https://github.com/CCIVR/ccivr.git' requires git; otherwise download and unzip the repository and install from the local path, or use pip3 if pip fails

## Evidence

- [methods] The input file must contain five different gene annotations listed in their independent columns: "The input file must contain five different gene annotations listed in their independent columns"
- [methods] ccivr [path of input CSV] --output [path of saving directory]: "Two CSV formatted output files **"Table.csv"** and **"Summary.csv"** will be generated and saved in a new directory named **"ccivr_output"**"
- [abstract] comprehensively identifies all kinds of cis-NATs based on genome annotation with expression data obtained from RNA-seq: "comprehensively identifies all kinds of cis-NATs based on genome annotation with expression data obtained from RNA-seq"
- [results] the percentage of cis-NAT-containing genes tended to increase from lower to higher organism complexity: "the percentage of cis-NAT-containing genes tended to increase from lower to higher organism complexity"
- [discussion] For predicting the composition of various cis-NATs by CCIVR, it depends on the accuracy of gene annotation including their structure and strand direction deposited in the databases: "For predicting the composition of various cis-NATs by CCIVR, it depends on the accuracy of gene annotation including their structure and strand direction deposited in the databases."
- [readme] it takes 6 minutes 34 seconds using an iMac 2017 equipped with a 3.4 GHz Intel Core i5: "it takes 6 minutes 34 seconds using an iMac 2017 equipped with a 3.4 GHz Intel Core i5"
