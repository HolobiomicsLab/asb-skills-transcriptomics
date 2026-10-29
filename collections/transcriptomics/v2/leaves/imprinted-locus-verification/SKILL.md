---
name: imprinted-locus-verification
description: Use when RNA-seq data from parthenogenetic (pESC) and androgenetic (aESC)
  embryonic stem cells (or other allele-biased expression settings) must be screened
  to confirm a known imprinted cis-NAT pair is identified and expressed from opposite
  parental alleles.
when_to_use_negative:
- Raw (unprocessed) FASTQ data — CCIVR requires RNA-seq data already processed by
  third-party programs such as STAR for mapping, RSEM for expression profiling, and
  DESeq2 for statistics
- Inputs lacking strand-direction or accurate gene annotation, since CCIVR's prediction
  depends on annotation accuracy of structure and strand direction deposited in databases
- Strand-specific novel transcript discovery without a genome annotation — tools like
  NASTI-seq or NATpipe are designed for de novo or strand-error-aware detection instead
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0621
tools:
- name: CCIVR
  role: Extracts total cis-NAT pairs (EB, FO, HH, TT) from genome-annotated input
    CSV with attached RNA-seq expression columns, generating Table.csv and Summary.csv
    used to verify the imprinted pair
  repo: https://github.com/CCIVR/ccivr
- name: pip
  role: Installs CCIVR from the GitHub repository ('pip install git+https://github.com/CCIVR/ccivr.git')
- name: pandas
  role: Prepares the input CSV ensuring the five required annotation columns and DEG-derived
    expression columns are present
- name: Ensembl
  role: Source of gene annotations (id, Chr, Strand, Start, End) used to build the
    CCIVR input file
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:d4f87afd81d94a1be6bb8b0ab860aaac6aae28c4941d596aab178d380828d657
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCIVR/ccivr
  tools:
  - CCIVR
  - Ensembl
  - pandas
  - pip
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# imprinted-locus-verification

## Summary

Verifies that a known imprinted cis-NAT pair (e.g. KCNQ1OT1/KCNQ1) is recovered from RNA-seq of parthenogenetic and androgenetic embryonic stem cells by running CCIVR, computing sense–antisense expression correlations from DEG columns, and confirming the pair falls in the negatively correlated set.

## When to use

Use when RNA-seq data from parthenogenetic (pESC) and androgenetic (aESC) embryonic stem cells (or other allele-biased expression settings) must be screened to confirm a known imprinted cis-NAT pair is identified and expressed from opposite parental alleles.

## When NOT to use

- Raw (unprocessed) FASTQ data — CCIVR requires RNA-seq data already processed by third-party programs such as STAR for mapping, RSEM for expression profiling, and DESeq2 for statistics
- Inputs lacking strand-direction or accurate gene annotation, since CCIVR's prediction depends on annotation accuracy of structure and strand direction deposited in databases
- Strand-specific novel transcript discovery without a genome annotation — tools like NASTI-seq or NATpipe are designed for de novo or strand-error-aware detection instead

## Inputs

- CSV-formatted gene dataset with five annotation columns (id, Chr, Strand, Start, End) from Ensembl
- DEG-derived expression columns (e.g. TPM, FPKM, fold_change, padj) from pESC/aESC RNA-seq processed with STAR, RSEM, and DESeq2

## Outputs

- Table.csv listing total extracted cis-NATs with structural types (EB, FO, HH, TT) and attached expression columns
- Summary.csv with absolute number and percentage of each cis-NAT type
- Pearson/Spearman correlation counts of positively (expected 164) versus negatively (expected 29) correlated cis-NAT pairs
- Confirmation that KCNQ1/KCNQ1OT1 appears in the negatively correlated EB pair set

## How to apply

Prepare a CSV input in pandas containing the five required gene annotation columns (id, Chr, Strand, Start, End) plus DEG-derived expression columns (e.g. fold_change, padj) from prior DESeq2 analysis; CCIVR recognizes column characters, not column order. Install CCIVR with pip from github.com/CCIVR/ccivr and verify with 'ccivr --help'. Run 'ccivr [input CSV] --output [dir]' to execute the EB, FO, HH, and TT plus-to-minus and minus-to-plus extractions, producing Table.csv and Summary.csv in a ccivr_output directory. Parse Table.csv and compute Pearson/Spearman correlation of sense–antisense expression for each cis-NAT pair, counting positively versus negatively correlated pairs against the reported benchmark (164 positive, 29 negative). Finally filter the negatively correlated, embedded-type (EB) pairs to confirm the imprinted KCNQ1/KCNQ1OT1 pair is present, matching Fig. 3D–E of the paper.

## Related tools

- **CCIVR** (Extracts total cis-NAT pairs (EB, FO, HH, TT) from genome-annotated input CSV with attached RNA-seq expression columns, generating Table.csv and Summary.csv used to verify the imprinted pair) — https://github.com/CCIVR/ccivr
- **pip** (Installs CCIVR from the GitHub repository ('pip install git+https://github.com/CCIVR/ccivr.git'))
- **pandas** (Prepares the input CSV ensuring the five required annotation columns and DEG-derived expression columns are present)
- **Ensembl** (Source of gene annotations (id, Chr, Strand, Start, End) used to build the CCIVR input file)

## Examples

```
ccivr pESC_aESC_genes_with_DEG.csv --output ./ccivr_output
```

## Evaluation signals

- Reproduced count of positively correlated cis-NAT pairs equals the reported 164
- Reproduced count of negatively correlated cis-NAT pairs equals the reported 29
- The KCNQ1/KCNQ1OT1 embedded-type pair is present in the negatively correlated subset of Table.csv
- ccivr run prints the expected processing sequence (EB/FO/HH/TT, each with Plus-to-minus and Minus-to-minus extracting) and writes both Table.csv and Summary.csv in ccivr_output
- Input CSV passes the column check: CCIVR recognizes id, Chr, Strand, Start, End by column characters regardless of order

## Limitations

- Prediction depends on the accuracy of gene annotation including structure and strand direction deposited in databases
- CCIVR scripts do not cover the full pipeline; expression profile analysis relies on processed RNA-seq data from third-party programs (STAR for mapping, RSEM for expression profiling, DESeq2 for statistical analysis)
- Full run on a test file can take a few to dozens of minutes on standard hardware (6 min 34 s on an iMac 2017, 3.4 GHz Core i5)

## Evidence

- [abstract] CCIVR analysis with RNA-seq data from parthenogenetic and androgenetic embryonic stem cells identified well-known imprinted cis-NAT pair, KCNQ1/KCNQ1OT1: "CCIVR analysis with RNA-seq data from parthenogenetic and androgenetic embryonic stem cells identified well-known imprinted cis-NAT pair, KCNQ1/KCNQ1OT1"
- [methods] Install CCIVR via pip: "You can install CCIVR via a package manager `pip` by typing the following command:"
- [methods] Output files: "Two CSV formatted output files **"Table.csv"** and **"Summary.csv"** will be generated and saved in a new directory named **"ccivr_output"**"
- [readme] Column recognition: "The order of these five columns is changeable because CCIVR recognizes their characters (*i.e.*, **id**, **Chr**, **Strand**, **Start**, **End**) but not their order."
- [results] Benchmark counts: [evidence span withheld — anchor status: not_found]
- [discussion] Limitation: third-party dependence: "it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis"
- [discussion] Limitation: annotation accuracy: "For predicting the composition of various cis-NATs by CCIVR, it depends on the accuracy of gene annotation including their structure and strand direction deposited in the databases."
