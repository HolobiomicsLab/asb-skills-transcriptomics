---
name: cis-nat-identification-from-genome-annotation
description: Use when the task is to comprehensively extract cis-NAT pairs from genome
  annotation (id, Chr, Strand, Start, End columns, e.g. from Ensembl) with optional
  RNA-seq expression profiling (TPM, FPKM, fold_change, padj), and to classify them
  into EB/FO/HH/TT structural types.
when_to_use_negative:
- Input is raw sequencing data rather than processed gene-level annotation — CCIVR
  relies on third-party tools (STAR for mapping, RSEM for expression, DESeq2 for statistics)
  upstream; run those first.
- The task requires strand-specific RNA-seq error-rate-aware cis-NAT detection from
  transcriptome assemblies — tools like NASTI-seq (R) or NATpipe (Perl) cover de novo
  assembled transcriptomes instead.
- Downstream gene annotation (structure and strand direction) in the database is unknown
  or unreliable, since CCIVR predictions depend on annotation accuracy.
edam_operation: http://edamontology.org/operation_2434
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0621
tools:
- name: CCIVR
  role: Python package that extracts total cis-NAT pairs (EB, FO, HH, TT) from annotation
    CSV input and writes Table.csv and Summary.csv
  repo: https://github.com/CCIVR/ccivr
- name: pip
  role: package manager used to install CCIVR from the GitHub repository
  repo: https://github.com/CCIVR/ccivr
- name: STAR
  role: third-party mapper used upstream to produce the processed RNA-seq data CCIVR
    consumes
- name: RSEM
  role: third-party tool used upstream for expression profiling whose output columns
    feed CCIVR input
- name: DESeq2
  role: third-party tool used upstream for statistical analysis yielding fold_change
    and padj columns for CCIVR input
- name: Ensembl
  role: source database for the gene annotations (id, Chr, Strand, Start, End) required
    in the input CSV
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:d61ecfe504e1d0d38e9c3bfeaf9fcbcd310283eadc9b95dd005d42eeb09b1ce9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCIVR/ccivr
  tools:
  - CCIVR
  - DESeq2
  - Ensembl
  - RSEM
  - STAR
  - pip
  edam_operation: http://edamontology.org/operation_2434
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cis-nat-identification-from-genome-annotation

## Summary

Identify all cis-natural antisense transcript (cis-NAT) pairs — embedded (EB), fully-overlapped (FO), head-to-head (HH), and tail-to-tail (TT) — from a gene annotation CSV, optionally with RNA-seq expression columns, using the CCIVR Python program.

## When to use

Use when the task is to comprehensively extract cis-NAT pairs from genome annotation (id, Chr, Strand, Start, End columns, e.g. from Ensembl) with optional RNA-seq expression profiling (TPM, FPKM, fold_change, padj), and to classify them into EB/FO/HH/TT structural types.

## When NOT to use

- Input is raw sequencing data rather than processed gene-level annotation — CCIVR relies on third-party tools (STAR for mapping, RSEM for expression, DESeq2 for statistics) upstream; run those first.
- The task requires strand-specific RNA-seq error-rate-aware cis-NAT detection from transcriptome assemblies — tools like NASTI-seq (R) or NATpipe (Perl) cover de novo assembled transcriptomes instead.
- Downstream gene annotation (structure and strand direction) in the database is unknown or unreliable, since CCIVR predictions depend on annotation accuracy.

## Inputs

- CSV-formatted gene annotation file with five required columns: id, Chr, Strand, Start, End (e.g. from Ensembl, GRCm39-style coordinates)
- Optional extra annotation columns: GeneSymbol, gene_source, gene_biotype, GO_analysis
- Optional RNA-seq expression columns: TPM, FPKM, fold_change, padj (q-value), e.g. DESeq2-derived DEG columns

## Outputs

- Table.csv — list of all extracted cis-NATs with structural characteristics (EB, FO, HH, TT) plus attached annotation/expression columns
- Summary.csv — absolute number and percentage of each cis-NAT type (e.g. test file: 27351 genes with cisNats [42.04%]; EB 8112, FO 12194, HH 5992, TT 6097)

## How to apply

Prepare a CSV input containing five required gene annotation columns (id, Chr, Strand, Start, End) in any order — CCIVR recognizes column characters, not column order — optionally adding extra annotation (GeneSymbol, gene_biotype) and expression columns (TPM, FPKM, fold_change, padj). Install via 'pip install git+https://github.com/CCIVR/ccivr.git' (or pip3; unzip+local pip install if git is missing) and verify with 'ccivr --help'. Run 'ccivr [path of input CSV] --output [path of saving directory]' so CCIVR performs the EB, FO, HH, and TT plus-to-minus and minus-to-minus extractions and writes Table.csv and Summary.csv into a ccivr_output directory. Downstream, parse Table.csv and compute Pearson/Spearman correlations of sense-antisense expression to find positively (164) versus negatively (29) correlated pairs, as demonstrated with pESC/aESC RNA-seq data where the imprinted embedded pair KCNQ1/KCNQ1OT1 was recovered in the negatively correlated set.

## Related tools

- **CCIVR** (Python package that extracts total cis-NAT pairs (EB, FO, HH, TT) from annotation CSV input and writes Table.csv and Summary.csv) — https://github.com/CCIVR/ccivr
- **pip** (package manager used to install CCIVR from the GitHub repository) — https://github.com/CCIVR/ccivr
- **STAR** (third-party mapper used upstream to produce the processed RNA-seq data CCIVR consumes)
- **RSEM** (third-party tool used upstream for expression profiling whose output columns feed CCIVR input)
- **DESeq2** (third-party tool used upstream for statistical analysis yielding fold_change and padj columns for CCIVR input)
- **Ensembl** (source database for the gene annotations (id, Chr, Strand, Start, End) required in the input CSV)

## Examples

```
pip install git+https://github.com/CCIVR/ccivr.git; ccivr [path of input CSV] --output [path of saving directory]
```

## Evaluation signals

- Summary.csv reports cis-NAT counts split by EB/FO/HH/TT with percentages of total genes, matching the expected processing-status output format (e.g. test file: 65065 total genes, 27351 genes with cisNats [42.04%]).
- Reproduced benchmark counts: 164 positively correlated and 29 negatively correlated cis-NAT pairs from pESC/aESC RNA-seq data when correlating sense-antisense expression.
- The imprinted embedded-type cis-NAT pair KCNQ1/KCNQ1OT1 (paternal antisense, maternal sense) appears in the negatively correlated set of Table.csv, as in Fig. 3D-E.
- 'ccivr --help' runs successfully after installation, confirming PATH configuration.
- Table.csv and Summary.csv are generated in a new 'ccivr_output' directory (or the --output target).

## Limitations

- Prediction of cis-NAT composition depends on the accuracy of gene annotation, including structure and strand direction, deposited in the databases.
- CCIVR scripts do not cover the full pipeline: it uses processed RNA-seq data produced by third-party programs (STAR, RSEM, DESeq2).
- pip install from git fails without git installed; in that case download the ZIP and 'pip install [full path of unzipped package]', or try pip3.
- Running the full process with the test file takes from a few to dozens of minutes (e.g. 6 minutes 34 seconds on an iMac 2017, 3.4 GHz Intel Core i5).

## Evidence

- [abstract] comprehensively identifies all kinds of cis-NATs based on genome annotation with expression data obtained from RNA-seq: "comprehensively identifies all kinds of cis-NATs based on genome annotation with expression data obtained from RNA-seq"
- [readme] input file requirements: "The input file must contain five different gene annotations listed in their independent columns, as follows: - **id**: gene id of each gene."
- [readme] column order: "The order of these five columns is changeable because CCIVR recognizes their characters (*i.e.*, **id**, **Chr**, **Strand**, **Start**, **End**) but not their order."
- [methods] expression columns: "Attaching information for expression profiling obtained from RNA-seq analysis is also available during the CCIVR process by adding extra columns such as **"TPM"**, **"FPKM"**, **"fold_change"**,"
- [methods] output files: "Two CSV formatted output files **"Table.csv"** and **"Summary.csv"** will be generated and saved in a new directory named **"ccivr_output"**"
- [discussion] third-party dependence: "it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis"
- [discussion] annotation accuracy limitation: "it depends on the accuracy of gene annotation including their structure and strand direction deposited in the databases."
- [abstract] KCNQ1 finding: "CCIVR analysis with RNA-seq data from parthenogenetic and androgenetic embryonic stem cells identified well-known imprinted cis-NAT pair, KCNQ1/KCNQ1OT1"
