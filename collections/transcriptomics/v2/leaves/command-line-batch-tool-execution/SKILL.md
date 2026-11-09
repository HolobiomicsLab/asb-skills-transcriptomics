---
name: command-line-batch-tool-execution
description: Use when the same CLI annotation/extraction tool must be run independently
  over multiple genome annotation files (e.g. GTF-converted CSVs for 11 model organisms)
  and per-run outputs parsed and aggregated into a comparative table.
when_to_use_negative:
- The task requires de novo transcript discovery from raw RNA-seq rather than annotation-based
  cis-NAT identification — CCIVR relies on deposited gene annotation including strand
  direction
- The input lacks the five required gene annotation columns or strand direction, which
  CCIVR depends on for accurate cis-NAT prediction
- Only a single-organism analysis with expression profiling is needed — a single direct
  CCIVR run suffices; the batch loop is for comparative multi-species compilation
edam_operation: http://edamontology.org/operation_2451
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: CCIVR
  role: Python command-line tool installed via pip and invoked once per species annotation
    file to extract EB, FO, HH, and TT cis-NAT pairs and produce Table.csv and Summary.csv
  repo: github.com/CCIVR/ccivr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:47e44bd4eeaf62c78dfddcd8f4701e7b9aa4cd27966bf8cb84f251da243e7bec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CCIVR
  edam_operation: http://edamontology.org/operation_2451
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# command-line-batch-tool-execution

## Summary

Batch-execute a Python command-line tool (CCIVR) uniformly across many species-level gene annotation datasets, collecting each run's structured outputs (Table.csv, Summary.csv) for cross-species compilation of cis-NAT counts and percentages.

## When to use

Use when the same CLI annotation/extraction tool must be run independently over multiple genome annotation files (e.g. GTF-converted CSVs for 11 model organisms) and per-run outputs parsed and aggregated into a comparative table.

## When NOT to use

- The task requires de novo transcript discovery from raw RNA-seq rather than annotation-based cis-NAT identification — CCIVR relies on deposited gene annotation including strand direction
- The input lacks the five required gene annotation columns or strand direction, which CCIVR depends on for accurate cis-NAT prediction
- Only a single-organism analysis with expression profiling is needed — a single direct CCIVR run suffices; the batch loop is for comparative multi-species compilation

## Inputs

- CSV-formatted gene annotation files, one per species, with five required columns (id, Chr, Strand, Start, End), derived from Ensembl GTF annotations
- Optionally extended CSVs with extra annotation columns (GeneSymbol, gene_biotype) or expression columns (TPM, FPKM, fold_change, padj)

## Outputs

- Per-species Table.csv listing all extracted cis-NAT pairs with structural characteristics (EB, FO, HH, TT)
- Per-species Summary.csv with absolute counts and percentages of each cis-NAT type relative to total genes
- Compiled cross-species table of genes with any, EB, FO, HH, and TT cis-NATs (counts and percentages)

## How to apply

First normalize every per-species input into the tool's CSV schema — five required columns (id, Chr, Strand, Start, End) in any column order — converting Ensembl GTF annotations (Ensembl release 105/104; Ensembl plants release 52; Ensembl fungi release 52). Install the tool once via pip from its GitHub repo and verify with 'ccivr --help'. Then loop over all input files, invoking 'ccivr [path of input CSV] --output [path of saving directory]' for each; each run sequentially performs Plus-to-minus and Minus-to-plus extraction for the EB, FO, HH, and TT cis-NAT types. After each run, parse the generated Table.csv (all extracted cis-NAT pairs by structural type) and Summary.csv (absolute number and percentage of each cis-NAT type relative to total genes), and compile per-species values for genes with any, EB, FO, HH, and TT cis-NATs. Budget runtime per file — the human test file takes 6 min 34 s on an iMac 2017 (3.4 GHz Intel Core i5), so dozens of minutes per organism on standard hardware.

## Related tools

- **CCIVR** (Python command-line tool installed via pip and invoked once per species annotation file to extract EB, FO, HH, and TT cis-NAT pairs and produce Table.csv and Summary.csv) — github.com/CCIVR/ccivr

## Examples

```
ccivr /data/annotations/human.csv --output /results/human/ ; ccivr /data/annotations/mouse.csv --output /results/mouse/
```

## Evaluation signals

- Each run's stdout status output follows the expected sequence: EB, FO, HH, TT — each with 'Plus-to-minus extracting' then 'Minus-to-plus extracting' — followed by the '< Result >' block
- Per-species Summary.csv values reproduce Fig. 2 of Ohhata et al., Sci Rep. 12:15525, 2022; for the human test file, expect total genes 65065, genes with cisNats 27351 [42.04%], EB 8112 [12.47%], FO 12194 [18.74%], HH 5992 [9.21%], TT 6097 [9.37%]
- Both Table.csv and Summary.csv exist for every species in a 'ccivr_output' directory under the specified --output path
- Percentages in Summary.csv equal counts divided by total genes, and total genes match the input file's gene count
- Cross-species pattern check: the percentage of cis-NAT-containing genes tends to increase from lower to higher organism complexity, as reported in the paper

## Limitations

- Prediction accuracy depends on the accuracy of gene annotation (structure and strand direction) deposited in the source databases (Ensembl/NCBI)
- CCIVR scripts do not cover the full analysis pipeline: expression profiling relies on processed RNA-seq data from third-party programs (STAR for mapping, RSEM for expression profiling, DESeq2 for statistical analysis)
- Runtime is non-trivial per file — a few to dozens of minutes on a standard computer (6 min 34 s for the human test file on an iMac 2017, 3.4 GHz Intel Core i5)

## Evidence

- [abstract] Using CCIVR with genome databases, we demonstrated total cis-NAT pairs from 11 model organisms.: "Using CCIVR with genome databases, we demonstrated total cis-NAT pairs from 11 model organisms."
- [readme] install command: "pip install git+https://github.com/CCIVR/ccivr.git"
- [readme] run command with output dir: "ccivr [path of input CSV] --output [path of saving directory]"
- [readme] output files description: "Two CSV formatted output files **"Table.csv"** and **"Summary.csv"** will be generated and saved in a new directory named **"ccivr_output"**."
- [readme] input schema requirement: "The input file must contain five different gene annotations listed in their independent columns"
- [readme] runtime expectation: "it takes 6 minutes 34 seconds using an iMac 2017 equipped with a 3.4 GHz Intel Core i5"
- [discussion] annotation accuracy limitation: "it depends on the accuracy of gene annotation including their structure and strand direction deposited in the databases"
- [discussion] third-party dependency limitation: "it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis"
