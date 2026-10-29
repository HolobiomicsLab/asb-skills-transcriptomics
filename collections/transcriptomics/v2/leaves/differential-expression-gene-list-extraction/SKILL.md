---
name: differential-expression-gene-list-extraction
description: Use when a CCIVR Table.csv/Summary.csv of cis-NAT pairs carries RNA-seq
  expression columns (TPM, fold_change, padj) and the goal is to split partner genes
  into up/down lists for GO enrichment (e.g., DAVID GOTERM_BP_FAT).
when_to_use_negative:
- Input is raw RNA-seq data (FASTQ) — CCIVR requires processed RNA-seq data produced
  by third-party programs (STAR for mapping, RSEM for expression profiling, DESeq2
  for statistical analysis), so raw reads must be processed first.
- The analysis goal is only cis-NAT structural characterization (EB/FO/HH/TT counts)
  without expression profiling — CCIVR handles this directly, and no differential
  gene lists can be extracted.
- Genes of interest are not members of cis-NAT pairs from a CCIVR Table.csv run —
  using this list-extraction workflow would misdefine the gene universe; use whole-transcriptome
  DE gene lists instead.
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: CCIVR
  role: Produces Table.csv listing total extracted cis-NAT pairs with expression columns
    (TPM, FPKM, fold_change, padj) that serve as the source for the up/down gene lists
  repo: github.com/CCIVR/ccivr
- name: DAVID
  role: Functional Annotation Tool (v6.8) used to run GOTERM_BP_FAT GO enrichment
    on each gene list against the Huh-7 dataset background
- name: Python
  role: pandas-based loading of the CCIVR output table, extraction of up/down partner-gene
    lists, and identifier conversion before DAVID submission
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:60618dabc4ac84cae2a3735da24771145caaca04aabdf23604fbe384bcf4300b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CCIVR
  - DAVID
  - Python
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# differential-expression-gene-list-extraction

## Summary

Extracts up-regulated and down-regulated gene identifier lists from CCIVR's cis-NAT pair output (Table.csv) using expression columns (fold_change, padj), converts Ensembl-style IDs to a DAVID-recognized format, and runs GOTERM_BP_FAT enrichment against a cell-line-specific background.

## When to use

Use when a CCIVR Table.csv/Summary.csv of cis-NAT pairs carries RNA-seq expression columns (TPM, fold_change, padj) and the goal is to split partner genes into up/down lists for GO enrichment (e.g., DAVID GOTERM_BP_FAT).

## When NOT to use

- Input is raw RNA-seq data (FASTQ) — CCIVR requires processed RNA-seq data produced by third-party programs (STAR for mapping, RSEM for expression profiling, DESeq2 for statistical analysis), so raw reads must be processed first.
- The analysis goal is only cis-NAT structural characterization (EB/FO/HH/TT counts) without expression profiling — CCIVR handles this directly, and no differential gene lists can be extracted.
- Genes of interest are not members of cis-NAT pairs from a CCIVR Table.csv run — using this list-extraction workflow would misdefine the gene universe; use whole-transcriptome DE gene lists instead.

## Inputs

- CCIVR output Table.csv (list of extracted cis-NAT pairs with structural types EB/FO/HH/TT plus expression columns TPM, FPKM, fold_change, padj)
- CCIVR input/test CSV with five gene annotation columns (id, Chr, Strand, Start, End) and human Huh-7 RNA-seq expression profiling under TGFβ stimulation
- Gene annotation columns (e.g., GeneSymbol, gene_source) for Ensembl-to-DAVID identifier conversion

## Outputs

- go_bp_enrichment_top15.csv — combined table of 15 GO biological-process terms per condition (up-regulated and down-regulated) with term names, gene counts, and Benjamini/FDR-corrected p-values
- Up-regulated and down-regulated gene ID lists (DAVID-recognized format) derived from differentially expressed cis-NAT partner genes

## How to apply

Load the prior CCIVR run's ccivr_output/Table.csv with Python/pandas and extract the gene identifier lists of the up-regulated and down-regulated differentially expressed cis-NAT partner genes, using the fold_change and padj columns that CCIVR accepts as extra input columns. Convert the Ensembl-style gene IDs (e.g., ENSG/ENSMUSG format) to a DAVID-recognized identifier type — official gene symbols or Ensembl Gene ID — using the annotation columns of the Huh-7 test file (the README documents extra columns such as 'GeneSymbol' and 'gene_source' for this purpose). Submit each list separately to the DAVID v6.8 Functional Annotation Tool, with the full Huh-7 dataset genes as the background/universe, and run GOTERM_BP_FAT (GO biological-process) analysis. Rank enriched terms by Benjamini/FDR-corrected significance and take the top 15 terms per list, writing term names, gene counts, and corrected p-values to a combined CSV (go_bp_enrichment_top15.csv). The rationale is that CCIVR identifies inversely correlated cis-NAT pairs upon TGFβ stimulation in Huh-7 cells, so functional interpretation of those partner genes requires list-wise enrichment rather than single-gene inspection.

## Related tools

- **CCIVR** (Produces Table.csv listing total extracted cis-NAT pairs with expression columns (TPM, FPKM, fold_change, padj) that serve as the source for the up/down gene lists) — github.com/CCIVR/ccivr
- **DAVID** (Functional Annotation Tool (v6.8) used to run GOTERM_BP_FAT GO enrichment on each gene list against the Huh-7 dataset background)
- **Python** (pandas-based loading of the CCIVR output table, extraction of up/down partner-gene lists, and identifier conversion before DAVID submission)

## Examples

```
import pandas as pd; t = pd.read_csv("ccivr_output/Table.csv"); up = t[t.fold_change > 1].id.tolist(); down = t[t.fold_change < 1].id.tolist()
```

## Evaluation signals

- go_bp_enrichment_top15.csv contains exactly 15 GO terms per condition (up-regulated and down-regulated), each with enrichment statistics (gene count and corrected p-value).
- The up-regulated list's top terms reproduce the Fig. 4C pattern: TGF-β signaling-related terms including EMT genes are enriched.
- The down-regulated list's top terms are cell-growth-related GO biological-process terms.
- Enrichment is ranked by Benjamini/FDR-corrected p-values, not raw p-values.
- Gene lists contain only partner genes of differentially expressed cis-NAT pairs from the CCIVR Table.csv, and DAVID used the Huh-7 dataset genes as the background universe.

## Limitations

- CCIVR depends on the accuracy of gene annotation (structure and strand direction) deposited in databases such as Ensembl, which propagates to the extracted partner-gene lists.
- CCIVR scripts do not cover the full pipeline — they rely on processed RNA-seq data from third-party programs (STAR, RSEM, DESeq2), so the quality of fold_change/padj values depends on that upstream analysis.
- Ensembl-style gene IDs are not directly DAVID-recognized; conversion must use the gene annotation columns of the Huh-7 test file, and mismatches will shrink the submitted gene list.

## Evidence

- [results] GO analysis of differentially expressed cis-NAT pair genes upon TGFβ stimulation showed that TGFβ signaling-related genes (including EMT genes) were enriched: [evidence span withheld — anchor status: not_found]
- [methods] Huh-7 test file with TGFβ expression profiling: "The test file contains human gene datasets accompanied by expression profiling of human hepatoma Huh-7 cells with TGFß stimulation"
- [readme] CCIVR output table contents: "a table file containing a list of total extracted cis-NATs accompanied by their structural characteristics"
- [readme] Expression columns accepted by CCIVR: "adding extra columns such as **"TPM"**, **"FPKM"**, **"fold_change"**, **"padj"**, and so on to the input file"
- [discussion] CCIVR relies on third-party processed RNA-seq data: "it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis"
