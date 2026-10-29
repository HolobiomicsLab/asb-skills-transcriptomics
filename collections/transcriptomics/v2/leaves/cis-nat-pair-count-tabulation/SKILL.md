---
name: cis-nat-pair-count-tabulation
description: Use when a CCIVR run has produced Table.csv with cis-NAT pairs carrying
  fold_change and padj expression columns, and you need expression-combination counts
  (down-down, down-up, up-down, up-up) per EB/FO/HH/TT type or a down-up pair set.
when_to_use_negative:
- The input CSV carries only the five gene annotation columns (id, Chr, Strand, Start,
  End) with no expression profiling columns (fold_change/padj) — CCIVR will extract
  structural cis-NATs but no down/up combination classification is possible.
- You need de novo NAT discovery from unassembled transcriptomes (e.g. NATpipe-style)
  rather than tabulating pairs already extracted by CCIVR from genome annotation.
- The cis-NAT pair list was produced by a different tool (e.g. BEDTools annotation-based
  overlap only) and lacks the CCIVR Table.csv schema with structural types and expression
  columns.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CCIVR
  role: Generates the Table.csv/Summary.csv whose cis-NAT pairs (EB/FO/HH/TT) are
    parsed and counted by expression combination in this skill
  repo: https://github.com/CCIVR/ccivr
- name: DESeq2
  role: Upstream statistical analysis that supplies the fold_change and padj columns
    used to call each strand down or up
- name: RSEM
  role: Upstream expression profiling that supplies the TPM/FPKM values accompanying
    the pairs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:f72bbc3c6d0ca7a028e46612befaea6366f244da01b35fc0948ca7d334025485
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCIVR/ccivr
  tools:
  - CCIVR
  - DESeq2
  - RSEM
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# cis-nat-pair-count-tabulation

## Summary

Post-processing skill for CCIVR output: parse the ccivr_output/Table.csv of extracted sense-antisense (cis-NAT) pairs, classify each pair by its expression combination (down or up on each strand) using the fold_change and padj columns, and tabulate pair counts per structural type (EB, FO, HH, TT) to surface inversely correlated down-up pairs as in Figure 4B/4D of Ohhata et al. 2022.

## When to use

Use when a CCIVR run has produced Table.csv with cis-NAT pairs carrying fold_change and padj expression columns, and you need expression-combination counts (down-down, down-up, up-down, up-up) per EB/FO/HH/TT type or a down-up pair set.

## When NOT to use

- The input CSV carries only the five gene annotation columns (id, Chr, Strand, Start, End) with no expression profiling columns (fold_change/padj) — CCIVR will extract structural cis-NATs but no down/up combination classification is possible.
- You need de novo NAT discovery from unassembled transcriptomes (e.g. NATpipe-style) rather than tabulating pairs already extracted by CCIVR from genome annotation.
- The cis-NAT pair list was produced by a different tool (e.g. BEDTools annotation-based overlap only) and lacks the CCIVR Table.csv schema with structural types and expression columns.

## Inputs

- ccivr_output/Table.csv — CCIVR table listing all extracted cis-NAT pairs with structural type (EB/FO/HH/TT) and per-gene expression columns (TPM, FPKM, fold_change, padj)
- ccivr_output/Summary.csv — per-type counts and percentages for validation
- CCIVR input CSV with columns id, Chr, Strand, Start, End (e.g., the human Huh-7 TGFβ Test_file.csv)

## Outputs

- fig4b_cisnat_expression_combination_table.csv — pair counts per expression combination (down-down, down-up, up-down, up-up) for EB, FO, HH, TT
- Filtered list of down-up (negatively correlated) cis-NAT pairs, e.g. BCAR3-AS1/BCAR3 and GPX2/CHURC1
- Heatmap PNG of down-up pairs reproducing Figure 4D

## How to apply

First run CCIVR on the CSV input (five gene annotation columns id, Chr, Strand, Start, End, plus expression columns TPM, FPKM, fold_change, padj) so that ccivr_output/Table.csv lists every extracted cis-NAT pair tagged with its structural type (EB embedded, FO fully-overlapped, HH head-to-head, TT tail-to-tail). Parse Table.csv and, for each sense-antisense pair, assign an expression combination by reading the fold_change and padj columns of both strands — each strand is called down or up, giving down-down, down-up, up-down, or up-up. Tabulate the pair counts per combination separately for EB, FO, HH, and TT and write a combination table (fig4b_cisnat_expression_combination_table.csv in the source workflow). Focus on the down-up (inversely correlated) pairs — in the Huh-7 TGFβ analysis these included the fully-overlapped pairs BCAR3-AS1/BCAR3 and GPX2/CHURC1, which were subsequently validated by SETD2 knockdown with H3K36me3 promoter modification. Build a heatmap of the down-up pairs as the visual output reproducing Figure 4D.

## Related tools

- **CCIVR** (Generates the Table.csv/Summary.csv whose cis-NAT pairs (EB/FO/HH/TT) are parsed and counted by expression combination in this skill) — https://github.com/CCIVR/ccivr
- **DESeq2** (Upstream statistical analysis that supplies the fold_change and padj columns used to call each strand down or up)
- **RSEM** (Upstream expression profiling that supplies the TPM/FPKM values accompanying the pairs)

## Examples

```
ccivr data/Test_file.csv --output ccivr_output
```

## Evaluation signals

- Summary.csv reproduces the reported test-file composition: total genes 65065, genes with cisNats 27351 [42.04%], EB 8112 [12.47%], FO 12194 [18.74%], HH 5992 [9.21%], TT 6097 [9.37%].
- Summing the combination table (down-down + down-up + up-down + up-up) over all four structural types matches the total cis-NAT pair count in Table.csv.
- Every sense-antisense pair in Table.csv resolves to exactly one of the four expression combinations based on its fold_change and padj values (no unclassified rows).
- The down-up set for the Huh-7 TGFβ input contains the fully-overlapped (FO) pairs BCAR3-AS1/BCAR3 and GPX2/CHURC1 reported as validated in the paper.
- The heatmap PNG of down-up pairs matches the layout/content of Figure 4D of Ohhata et al. 2022.

## Limitations

- Prediction depends on the accuracy of gene annotation (structure and strand direction) deposited in the source databases such as Ensembl.
- CCIVR scripts do not cover the full analysis pipeline — expression profiling relies on third-party programs (STAR for mapping, RSEM for expression profiling, DESeq2 for statistics), so down/up calls inherit those tools' assumptions.
- A full CCIVR run on the test file can take from a few to dozens of minutes on a standard computer (e.g. 6 min 34 s on an iMac 2017, 3.4 GHz Core i5), so tabulation depends on a completed upstream run.

## Evidence

- [readme] Two CSV formatted output files **"Table.csv"** and **"Summary.csv"** will be generated and saved in a new directory named **"ccivr_output"**: "Two CSV formatted output files **"Table.csv"** and **"Summary.csv"** will be generated and saved in a new directory named **"ccivr_output"**"
- [readme] Summary.csv contains the absolute number of each type of extracted cis-NATs with its percentages of the total genes.: "Summary.csv contains the absolute number of each type of extracted cis-NATs with its percentages of the total genes."
- [methods] Attaching information for expression profiling obtained from RNA-seq analysis is also available during the CCIVR process by adding extra columns such as **"TPM"**, **"FPKM"**, **"fold_change"**: "Attaching information for expression profiling obtained from RNA-seq analysis is also available during the CCIVR process by adding extra columns such as **"TPM"**, **"FPKM"**, **"fold_change"**"
- [abstract] CCIVR identified cis-NAT pairs that demonstrate inversely correlated expression upon TGFβ stimulation: "CCIVR identified cis-NAT pairs that demonstrate inversely correlated expression upon TGFβ stimulation"
- [discussion] it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis: "it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis"
