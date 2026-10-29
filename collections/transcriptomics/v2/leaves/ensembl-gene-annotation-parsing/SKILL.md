---
name: ensembl-gene-annotation-parsing
description: Use when raw Ensembl GTF annotations must be transformed into CCIVR's
  required CSV input with id, Chr, Strand, Start, End columns before running cis-NAT
  extraction.
when_to_use_negative:
- The annotation source is not gene-locational data from Ensembl or a comparable database
  — CCIVR depends on accurate gene annotation including structure and strand direction
  deposited in the databases.
- The goal is transcript-level or isoform-level cis-NAT analysis requiring assembled
  transcriptomes, which tools like NATpipe handle rather than gene-annotation-based
  CCIVR input.
- The input file already exists in the five-column CCIVR CSV format (e.g. the repository's
  data/Test_file.csv), so no parsing is needed.
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: CCIVR
  role: Downstream program whose required five-column CSV input schema (id, Chr, Strand,
    Start, End) defines the parsing output format
  repo: https://github.com/CCIVR/ccivr
- name: Ensembl
  role: Source database from which gene annotations (id, chromosome, strand, start,
    end) are obtained before conversion to CSV
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:0d5302fa64b7a5cc6e2495eeded8a83ac2153295144c29e893b632be08f9caad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCIVR/ccivr
  tools:
  - CCIVR
  - Ensembl
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# ensembl-gene-annotation-parsing

## Summary

Converts Ensembl GTF gene annotations (e.g. Ensembl release 105/104, Ensembl plants release 52, Ensembl fungi release 52) into the five-column CSV format required by CCIVR for cis-NAT identification.

## When to use

Use when raw Ensembl GTF annotations must be transformed into CCIVR's required CSV input with id, Chr, Strand, Start, End columns before running cis-NAT extraction.

## When NOT to use

- The annotation source is not gene-locational data from Ensembl or a comparable database — CCIVR depends on accurate gene annotation including structure and strand direction deposited in the databases.
- The goal is transcript-level or isoform-level cis-NAT analysis requiring assembled transcriptomes, which tools like NATpipe handle rather than gene-annotation-based CCIVR input.
- The input file already exists in the five-column CCIVR CSV format (e.g. the repository's data/Test_file.csv), so no parsing is needed.

## Inputs

- Ensembl GTF gene annotation files (e.g. Ensembl release 105/104, Ensembl plants release 52, Ensembl fungi release 52)
- Optional RNA-seq expression tables (TPM, FPKM, fold_change, padj) keyed by gene id

## Outputs

- CSV-formatted CCIVR input file with five gene annotation columns (id, Chr, Strand, Start, End), optionally with extra annotation and expression columns

## How to apply

Download gene annotations from the Ensembl database (the paper used Ensembl release 105/104, Ensembl plants release 52, and Ensembl fungi release 52) and extract per-gene records into a CSV. The CCIVR input must contain five columns: id (e.g. ENSMUSG00000085715), Chr (e.g. X), Strand (e.g. +), Start (minimum gene location, e.g. 103431517), and End (maximum gene location, e.g. 103484957); column order is flexible because CCIVR recognizes the header characters rather than positions. Optionally attach extra annotation columns (GeneSymbol, gene_source, gene_biotype, GO_analysis) or RNA-seq expression columns (TPM, FPKM, fold_change, padj) so CCIVR can perform expression profiling in the same run. Validate the parsed file against the repository's test file (data/Test_file.csv, human Huh-7 genes with TGFß stimulation) and by running ccivr on it to reproduce the known summary (total genes 65065; genes with cisNats 27351 [42.04%]).

## Related tools

- **CCIVR** (Downstream program whose required five-column CSV input schema (id, Chr, Strand, Start, End) defines the parsing output format) — https://github.com/CCIVR/ccivr
- **Ensembl** (Source database from which gene annotations (id, chromosome, strand, start, end) are obtained before conversion to CSV)

## Examples

```
ccivr [path of input CSV] --output [path of saving directory]
```

## Evaluation signals

- The parsed CSV contains all five required columns with recognizable headers (id, Chr, Strand, Start, End), in any column order, and one row per gene.
- Running ccivr on the parsed human test-file equivalent reproduces the README reference values: total genes 65065; genes with cisNats 27351 [42.04%]; EB 8112 [12.47%]; FO 12194 [18.74%]; HH 5992 [9.21%]; TT 6097 [9.37%].
- Example gene records match Ensembl coordinates — e.g. murine Tsix: id ENSMUSG00000085715, Chr X, Strand +, Start 103431517, End 103484957 (GRCm39).
- ccivr --help runs successfully after installation, confirming the parsed input feeds a working CCIVR pipeline.
- Schema check: extra columns beyond the required five are optional annotation/expression fields (TPM, FPKM, fold_change, padj) and do not break CCIVR's column recognition.

## Limitations

- CCIVR's cis-NAT prediction depends on the accuracy of gene annotation, including gene structure and strand direction, as deposited in the databases.
- Installation via pip requires git on the machine; otherwise the ZIP file must be downloaded and installed from a local path, and pip3 may be needed instead of pip.
- If ccivr --help does not run, the $PATH environment setting must be checked.

## Evidence

- [methods] The input file must contain five different gene annotations listed in their independent columns: "The input file must contain five different gene annotations listed in their independent columns"
- [methods] These gene annotations can be obtained from the Ensembl database.: "These gene annotations can be obtained from the Ensembl database."
- [readme] The order of these five columns is changeable because CCIVR recognizes their characters: "The order of these five columns is changeable because CCIVR recognizes their characters (*i.e.*, **id**, **Chr**, **Strand**, **Start**, **End**) but not their order."
- [readme] Attaching information for expression profiling obtained from RNA-seq analysis is also available: "Attaching information for expression profiling obtained from RNA-seq analysis is also available during the CCIVR process by adding extra columns such as **“TPM”**, **“FPKM”**, **“fold_change”**,"
- [discussion] For predicting the composition of various cis-NATs by CCIVR, it depends on the accuracy of gene annotation: "For predicting the composition of various cis-NATs by CCIVR, it depends on the accuracy of gene annotation including their structure and strand direction deposited in the databases."
