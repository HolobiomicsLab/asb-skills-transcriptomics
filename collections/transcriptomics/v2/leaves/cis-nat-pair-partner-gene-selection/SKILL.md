---
name: cis-nat-pair-partner-gene-selection
description: Use when you have CCIVR output from an RNA-seq dataset (e.g. Huh-7 cells
  with TGFβ stimulation) and need to partition differentially expressed cis-NAT partner
  genes by direction (fold_change/padj) into gene lists for enrichment analysis.
when_to_use_negative:
- Input is raw sequencing data (FASTQ) — CCIVR relies on processed RNA-seq output
  from third-party programs (STAR for mapping, RSEM for expression profiling, DESeq2
  for statistics), so map and quantify first.
- You need the total cis-NAT structural catalog (EB/FO/HH/TT counts) without any expression-differential
  filtering — plain CCIVR Summary.csv analysis suffices and no direction-resolved
  gene selection applies.
- No strand-aware or genome-annotated cis-NAT pairs are present in the dataset (CCIVR
  output contains no differentially expressed pairs), so no partner-gene lists can
  be formed.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: CCIVR
  role: Identifies cis-NAT pairs with structural characteristics (EB, FO, HH, TT)
    and expression profiles, producing Table.csv from which partner genes are extracted
  repo: https://github.com/CCIVR/ccivr
- name: Python
  role: Pandas-based extraction of up-/down-regulated partner-gene lists from Table.csv
    and conversion of Ensembl-style IDs for DAVID submission
- name: DAVID
  role: v6.8 Functional Annotation Tool used to run GOTERM_BP_FAT GO biological-process
    enrichment on each partner-gene list against the Huh-7 background
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:cc4199aa3485f26c5cf44a8d03fa6665058cbe25da7bd5b2e9f1dc82fb53db94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCIVR/ccivr
  tools:
  - CCIVR
  - DAVID
  - Python
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cis-nat-pair-partner-gene-selection

## Summary

Selects the partner genes of differentially expressed cis-NAT pairs from a CCIVR run on Huh-7 TGFβ-stimulated RNA-seq data and prepares up-regulated/down-regulated gene lists for GO (GOTERM_BP_FAT) enrichment in DAVID v6.8. This links CCIVR's Table.csv cis-NAT inventory to functional interpretation of cis-NAT regulation.

## When to use

Use when you have CCIVR output from an RNA-seq dataset (e.g. Huh-7 cells with TGFβ stimulation) and need to partition differentially expressed cis-NAT partner genes by direction (fold_change/padj) into gene lists for enrichment analysis.

## When NOT to use

- Input is raw sequencing data (FASTQ) — CCIVR relies on processed RNA-seq output from third-party programs (STAR for mapping, RSEM for expression profiling, DESeq2 for statistics), so map and quantify first.
- You need the total cis-NAT structural catalog (EB/FO/HH/TT counts) without any expression-differential filtering — plain CCIVR Summary.csv analysis suffices and no direction-resolved gene selection applies.
- No strand-aware or genome-annotated cis-NAT pairs are present in the dataset (CCIVR output contains no differentially expressed pairs), so no partner-gene lists can be formed.

## Inputs

- CCIVR output Table.csv (ccivr_output directory) listing extracted cis-NAT pairs with structural types (EB, FO, HH, TT) and expression columns
- CCIVR input CSV with five gene annotation columns (id, Chr, Strand, Start, End) plus RNA-seq expression columns (TPM, FPKM, fold_change, padj), e.g. the Huh-7 TGFβ Test_file.csv
- Huh-7 dataset gene list to serve as the DAVID background/universe

## Outputs

- go_bp_enrichment_top15.csv with 15 GOTERM_BP_FAT terms per condition (up- and down-regulated), each with term name, gene counts, and Benjamini/FDR-corrected p-value
- Up-regulated and down-regulated partner-gene identifier lists in DAVID-recognized format

## How to apply

Load the CCIVR output directory's Table.csv (produced alongside Summary.csv in ccivr_output) with Python/pandas and extract the gene identifiers of partner genes belonging to differentially expressed cis-NAT pairs, splitting them into up-regulated and down-regulated lists based on the expression-profiling columns (fold_change, padj) attached to the input file. Convert the Ensembl-style gene IDs to a DAVID-recognized format (official gene symbols or Ensembl Gene ID) using the annotation columns of the CCIVR input CSV (id, Chr, Strand, Start, End, plus extras such as GeneSymbol). Submit each list to DAVID v6.8 Functional Annotation Tool using the full Huh-7 test-file gene set as background/universe, running GOTERM_BP_FAT (GO biological-process) separately per direction. Rank enriched terms by Benjamini/FDR-corrected p-value and keep the top 15 terms per list, writing them with term names, gene counts, and corrected p-values to go_bp_enrichment_top15.csv. Rationale: cis-NATs repress partner genes via epigenetic alteration, so direction-resolved GO terms reveal which biological processes (e.g. TGFβ signaling/EMT vs. cell growth) are under cis-NAT regulation.

## Related tools

- **CCIVR** (Identifies cis-NAT pairs with structural characteristics (EB, FO, HH, TT) and expression profiles, producing Table.csv from which partner genes are extracted) — https://github.com/CCIVR/ccivr
- **Python** (Pandas-based extraction of up-/down-regulated partner-gene lists from Table.csv and conversion of Ensembl-style IDs for DAVID submission)
- **DAVID** (v6.8 Functional Annotation Tool used to run GOTERM_BP_FAT GO biological-process enrichment on each partner-gene list against the Huh-7 background)

## Examples

```
ccivr Test_file.csv --output ccivr_output
```

## Evaluation signals

- go_bp_enrichment_top15.csv contains exactly 15 GO terms per condition (up- and down-regulated), each with enrichment statistics and Benjamini/FDR-corrected p-values
- Up-regulated list enrichment recapitulates the Fig. 4C pattern: TGF-β signaling/EMT-related GO terms among up-regulated partner genes
- Down-regulated list enrichment shows cell-growth-related GO terms among down-regulated partner genes
- DAVID run used the Huh-7 dataset genes as background/universe (correct denominator, not genome-wide)
- Partner-gene lists derive from the differentially expressed subset of CCIVR Table.csv, split by direction (fold_change/padj), not all cis-NAT pair genes

## Limitations

- CCIVR prediction depends on the accuracy of gene annotation (structure and strand direction) deposited in databases such as Ensembl.
- CCIVR scripts do not cover the full analysis pipeline — expression profiling relies on third-party programs (STAR, RSEM, DESeq2), so upstream errors propagate into the partner-gene lists.
- No explicit version number for CCIVR is given in the article; pin the installed version yourself for reproducibility.

## Evidence

- [abstract] CCIVR identified cis-NAT pairs that demonstrate inversely correlated expression upon TGFβ stimulation: "CCIVR identified cis-NAT pairs that demonstrate inversely correlated expression upon TGFβ stimulation"
- [methods] Input CSV annotation columns: "The input file must contain five different gene annotations listed in their independent columns"
- [methods] CCIVR output files: "Two CSV formatted output files **"Table.csv"** and **"Summary.csv"** will be generated and saved in a new directory named **"ccivr_output"**"
- [readme] Huh-7 test file description: "The test file contains human gene datasets accompanied by expression profiling of human hepatoma Huh-7 cells with TGFß"
- [discussion] Upstream pipeline dependency: "it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis"
