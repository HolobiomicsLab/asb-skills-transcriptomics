---
name: inversely-correlated-pair-heatmap-visualization
description: Reference for when CCIVR Table.csv contains fold_change and padj columns
  for both strands and you need to classify cis-NAT pairs into expression combinations
  and visualize the down-up subset as a heatmap.
when_to_use_negative:
- Input is raw RNA-seq reads rather than processed expression-profiled gene annotation
  data — CCIVR expects a CSV with gene locational information, not FASTQ (upstream
  STAR/RSEM/DESeq2 processing is required first).
- Gene annotation lacks strand direction or Start/End coordinates — CCIVR's prediction
  depends on the accuracy of gene annotation including structure and strand direction
  deposited in databases.
- Goal is identification of cis-NATs themselves rather than visualization of expression
  combinations of already-extracted pairs.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0622
tools:
- name: CCIVR
  role: Extracts total cis-NAT pairs with structural characteristics (EB, FO, HH,
    TT) and expression profiling from processed RNA-seq gene annotation CSV; its Table.csv
    is parsed for expression-combination classification and heatmap construction
  repo: github.com/CCIVR/ccivr
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:37240072eaf1a32bbaad53a42ceb179f652c10eaaf9b82336c92afdd852c9213
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CCIVR
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# inversely-correlated-pair-heatmap-visualization

## Summary

Skill for tabulating sense–antisense cis-NAT pairs by expression combination (down-down, down-up, up-down, up-up) from CCIVR's Table.csv and visualizing the inversely correlated (down-up) pairs as a heatmap reproducing Figure 4D of the TGFβ-stimulated Huh-7 analysis.

## When to use

Reference for when CCIVR Table.csv contains fold_change and padj columns for both strands and you need to classify cis-NAT pairs into expression combinations and visualize the down-up subset as a heatmap.

## When NOT to use

- Input is raw RNA-seq reads rather than processed expression-profiled gene annotation data — CCIVR expects a CSV with gene locational information, not FASTQ (upstream STAR/RSEM/DESeq2 processing is required first).
- Gene annotation lacks strand direction or Start/End coordinates — CCIVR's prediction depends on the accuracy of gene annotation including structure and strand direction deposited in databases.
- Goal is identification of cis-NATs themselves rather than visualization of expression combinations of already-extracted pairs.

## Inputs

- CCIVR input CSV with gene annotations (id, Chr, Strand, Start, End) plus expression profiling columns TPM, FPKM, fold_change, padj
- ccivr_output/Table.csv listing extracted cis-NAT pairs with structural characteristics (EB, FO, HH, TT)
- ccivr_output/Summary.csv with absolute number and percentage of each cis-NAT type

## Outputs

- fig4b_cisnat_expression_combination_table.csv with pair counts per expression combination (down-down, down-up, up-down, up-up) for EB, FO, HH, TT types
- PNG heatmap of down-up (inversely correlated) pairs reproducing Figure 4D

## How to apply

Run CCIVR on the input CSV (id, Chr, Strand, Start, End plus TPM, FPKM, fold_change, padj columns) and parse ccivr_output/Table.csv. For each extracted sense-antisense pair, classify the expression direction of each strand (down or up) using the fold_change and padj columns, then tabulate pair counts per combination (down-down, down-up, up-down, up-up) separately for the EB, FO, HH, and TT structural types, writing fig4b_cisnat_expression_combination_table.csv. Finally, build a heatmap of the down-up (inversely correlated) pairs from that table as a PNG figure reproducing Figure 4D. Validate against Summary.csv: the test file should classify 65,065 genes with 27,351 (42.04%) containing cis-NATs (EB 8112 [12.47%], FO 12194 [18.74%], HH 5992 [9.21%], TT 6097 [9.37%]), and combination-table totals should match the cis-NAT counts in Table.csv.

## Related tools

- **CCIVR** (Extracts total cis-NAT pairs with structural characteristics (EB, FO, HH, TT) and expression profiling from processed RNA-seq gene annotation CSV; its Table.csv is parsed for expression-combination classification and heatmap construction) — github.com/CCIVR/ccivr

## Examples

```
ccivr data/Test_file.csv --output results/
```

## Evaluation signals

- ccivr_output/Summary.csv reproduces the test-file composition: total genes 65065, genes with cisNats 27351 [42.04%], EB 8112 [12.47%], FO 12194 [18.74%], HH 5992 [9.21%], TT 6097 [9.37%]
- Combination-table totals match the cis-NAT counts in Table.csv
- Down-up pairs include fully-overlapped pairs BCAR3-AS1/BCAR3 and GPX2/CHURC1, which showed significant alterations upon SETD2 knockdown
- Heatmap PNG reproduces Figure 4D of the paper
- Processing completes on the test file in a few to dozens of minutes (~6 min 34 s on an iMac 2017 3.4 GHz Intel Core i5)

## Limitations

- Prediction depends on the accuracy of gene annotation including structure and strand direction deposited in databases
- CCIVR scripts do not cover the full pipeline and rely on third-party programs (STAR for mapping, RSEM for expression profiling, DESeq2 for statistical analysis)
- Full processing of the test file takes a few to dozens of minutes on a standard computer

## Evidence

- [abstract] CCIVR identified cis-NAT pairs that demonstrate inversely correlated expression upon TGFβ stimulation including cis-NATs that functionally repress their partner genes by introducing epigenetic: "CCIVR identified cis-NAT pairs that demonstrate inversely correlated expression upon TGFβ stimulation including cis-NATs that functionally repress their partner genes by introducing epigenetic"
- [readme] Two CSV formatted output files Table.csv and Summary.csv will be generated and saved in a new directory named ccivr_output: "Two CSV formatted output files **"Table.csv"** and **"Summary.csv"** will be generated and saved in a new directory named **"ccivr_output"**"
- [readme] Running CCIVR with the test file yields total genes 65065, genes with cisNats 27351 [42.04%], EB 8112 [12.47%], FO 12194 [18.74%], HH 5992 [9.21%], TT 6097 [9.37%]: "total genes : 65065
genes with cisNats : 27351 [42.04%]
     EB : 8112 [12.47%]
     FO : 12194 [18.74%]
     HH : 5992 [9.21%]
     TT : 6097 [9.37%]"
- [readme] Test file contains human gene datasets with expression profiling of human hepatoma Huh-7 cells with TGFβ stimulation, consistent with figure 4 input: "The test file contains human gene datasets accompanied by expression profiling of human hepatoma Huh-7 cells with TGFß stimulation, which is consistent with the input file used for the CCIVR analysis"
- [readme] Attaching information for expression profiling obtained from RNA-seq analysis is also available during the CCIVR process by adding extra columns such as TPM, FPKM, fold_change, padj: "Attaching information for expression profiling obtained from RNA-seq analysis is also available during the CCIVR process by adding extra columns such as **"TPM"**, **"FPKM"**, **"fold_change"**,"
