---
name: cis-nat-type-frequency-quantification
description: Use when genome annotations (e.g. Ensembl GTF releases 105/104 or Ensembl
  plants/fungi release 52) must be converted and screened with CCIVR to tabulate cis-NAT
  type counts and percentages per organism.
when_to_use_negative:
- The goal is expression profiling or differential expression of cis-NAT pairs (e.g.
  TGFβ-stimulated Huh-7 analysis or imprinted KCNQ1/KCNQ1OT1 detection) rather than
  structural type frequency quantification across species — that requires RNA-seq
  columns and the CCIVR expression workflow instead
- Only raw RNA-seq reads are available and the genome annotation has not been converted
  into CCIVR's CSV format (id, Chr, Strand, Start, End); CCIVR depends on accurate
  deposited gene annotations and does not perform mapping (third-party programs such
  as STAR are used for mapping)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CCIVR
  role: 'Primary tool: reads the CSV gene annotation input, performs Plus-to-minus
    and Minus-to-minus (Plus-to-minus/Minus-to-plus) extraction for EB, FO, HH, and
    TT cis-NAT types, and writes Table.csv and Summary.csv per organism'
  repo: https://github.com/CCIVR/ccivr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:b83af1ad2d97867dd42525209c14384eebb51c58fcffa12d4c353f388b14d82d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCIVR/ccivr
  tools:
  - CCIVR
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# cis-nat-type-frequency-quantification

## Summary

Quantify, per species, the absolute number and percentage of genes containing each cis-NAT structural type (EB, FO, HH, TT, and any) using the CCIVR tool on Ensembl-derived gene annotations, to compare cis-NAT prevalence across organisms.

## When to use

Use when genome annotations (e.g. Ensembl GTF releases 105/104 or Ensembl plants/fungi release 52) must be converted and screened with CCIVR to tabulate cis-NAT type counts and percentages per organism.

## When NOT to use

- The goal is expression profiling or differential expression of cis-NAT pairs (e.g. TGFβ-stimulated Huh-7 analysis or imprinted KCNQ1/KCNQ1OT1 detection) rather than structural type frequency quantification across species — that requires RNA-seq columns and the CCIVR expression workflow instead
- Only raw RNA-seq reads are available and the genome annotation has not been converted into CCIVR's CSV format (id, Chr, Strand, Start, End); CCIVR depends on accurate deposited gene annotations and does not perform mapping (third-party programs such as STAR are used for mapping)

## Inputs

- Ensembl GTF gene annotations (Ensembl release 105/104, Ensembl plants release 52, Ensembl fungi release 52) for 11 model organisms
- CCIVR CSV input file with five required columns: id, Chr, Strand, Start, End (order-independent; optional extra columns such as GeneSymbol, gene_biotype, TPM, FPKM, fold_change, padj)

## Outputs

- Table.csv: list of all extracted cis-NAT pairs with structural characteristics (EB, FO, HH, TT) per species
- Summary.csv: absolute number and percentage of each cis-NAT type relative to total genes, plus total genes and genes with cisNats, saved in a ccivr_output directory
- Per-species compiled table of counts and percentages for genes with any, EB, FO, HH, and TT cis-NATs

## How to apply

Convert each organism's Ensembl GTF annotation into a CCIVR CSV input containing the five required columns (id, Chr, Strand, Start, End), which CCIVR recognizes by header characters regardless of column order. Install CCIVR via 'pip install git+https://github.com/CCIVR/ccivr.git' and verify with 'ccivr --help'. Run 'ccivr [path of input CSV] --output [path of saving directory]' for each of the 11 model organisms; CCIVR sequentially performs Plus-to-minus and Minus-to-plus extraction for the EB (embedded), FO (fully-overlapped), HH (head-to-head), and TT (tail-to-tail) cis-NAT types. Parse the generated Table.csv files to tally all extracted cis-NAT pairs by structural type per species, and parse Summary.csv for the absolute number and percentage of each cis-NAT type relative to total genes. Rationale: percentages of cis-NAT-containing genes were shown to increase from lower to higher organism complexity (Fig. 2 of Ohhata et al., Sci Rep 12:15525, 2022), so per-type per-species tabulation is the analysis goal.

## Related tools

- **CCIVR** (Primary tool: reads the CSV gene annotation input, performs Plus-to-minus and Minus-to-minus (Plus-to-minus/Minus-to-plus) extraction for EB, FO, HH, and TT cis-NAT types, and writes Table.csv and Summary.csv per organism) — https://github.com/CCIVR/ccivr

## Examples

```
pip install git+https://github.com/CCIVR/ccivr.git; ccivr [path of input CSV] --output [path of saving directory]
```

## Evaluation signals

- The human test file (Test_file.csv, Huh-7 RNA-seq) should reproduce the README/paper reference values: total genes 65065; genes with cisNats 27351 [42.04%]; EB 8112 [12.47%]; FO 12194 [18.74%]; HH 5992 [9.21%]; TT 6097 [9.37%]
- Each run must complete all eight extraction phases in order (EB, FO, HH, TT, each with Plus-to-minus then Minus-to-plus extracting) and emit Table.csv and Summary.csv in a ccivr_output directory
- Per-species counts and percentages in Summary.csv must agree with the values plotted in Fig. 2 of Ohhata et al., Sci Rep. 12:15525, 2022 (Supplementary Dataset File 1)
- Percentage of cis-NAT-containing genes should show the reported trend of increase from lower to higher organism complexity across the 11 model organisms

## Limitations

- For predicting the composition of various cis-NATs, CCIVR depends on the accuracy of gene annotation including their structure and strand direction deposited in the databases
- CCIVR scripts do not cover the full analysis pipeline; expression profile analysis relies on processed RNA-seq data from third-party programs (STAR for mapping, RSEM for expression profiling, DESeq2 for statistical analysis)
- Runtime can be substantial: completing the whole CCIVR process with the human test file took 6 minutes 34 seconds on an iMac 2017 with a 3.4 GHz Intel Core i5, so large genomes may take dozens of minutes

## Evidence

- [results] Percentage of cis-NAT-containing genes tended to increase from lower to higher organism complexity (Fig. 2): [evidence span withheld — anchor status: not_found]
- [readme] CCIVR extracts EB, FO, HH, TT via Plus-to-minus and Minus-to-plus extraction: "EB
Plus-to-minus extracting
Minus-to-plus extracting
FO
Plus-to-minus extracting
Minus-to-plus extracting
HH
Plus-to-minus extracting
Minus-to-plus extracting
TT
Plus-to-minus"
- [readme] Test file reference values to validate output: "total genes : 65065
genes with cisNats : 27351 [42.04%]
     EB : 8112 [12.47%]
     FO : 12194 [18.74%]
     HH : 5992 [9.21%]
     TT : 6097 [9.37%]"
- [readme] Summary.csv contains absolute numbers and percentages of each cis-NAT type: "**Summary.csv** contains the absolute number of each type of extracted cis-NATs with its percentages of the total genes."
- [discussion] Stated limitation: dependence on annotation accuracy: "For predicting the composition of various cis-NATs by CCIVR, it depends on the accuracy of gene annotation including their structure and strand direction deposited in the databases."
