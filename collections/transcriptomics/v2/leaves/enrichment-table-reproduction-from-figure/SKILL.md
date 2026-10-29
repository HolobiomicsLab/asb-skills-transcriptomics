---
name: enrichment-table-reproduction-from-figure
description: Use when a paper reports GO enrichment as a figure (e.g., GO BP terms
  among up- vs down-regulated cis-NAT partner genes) and the goal is a reproducible
  enrichment table rather than just a visual interpretation.
when_to_use_negative:
- The input is not differentially expressed partner genes from a CCIVR-style cis-NAT
  table but raw RNA-seq counts — first run mapping (STAR), expression profiling (RSEM),
  and DESeq2 differential analysis to obtain fold_change/padj.
- The target figure reports non-GO enrichment (e.g., KEGG pathways or motif analysis),
  since this skill reproduces GOTERM_BP_FAT biological-process terms specifically.
- No matched background gene set exists; DAVID enrichment requires the Huh-7 dataset
  genes (or equivalent) as the background universe, and results are not interpretable
  without it.
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3170
tools:
- name: CCIVR
  role: Produces the Table.csv of cis-NAT pairs whose differentially expressed partner
    genes are extracted for enrichment analysis
  repo: https://github.com/CCIVR/ccivr
- name: DAVID
  role: Functional Annotation Tool used to run GOTERM_BP_FAT enrichment on up- and
    down-regulated gene lists with the Huh-7 background
- name: Python
  role: Used with pandas to load the CCIVR output table, filter DE cis-NAT partner
    genes, and write the top-15 CSV
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-022-19782-5
    title: CCIVR facilitates comprehensive identification of cis-natural antisense
      transcripts with their structural characteristics and expression profiles
  source_content_hash: sha256:95c62429ba5e35e1389d669d324f2f94d08b42f19292068a1939431f2ab67a3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CCIVR/ccivr
  tools:
  - CCIVR
  - DAVID
  - Python
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# enrichment-table-reproduction-from-figure

## Summary

Reproduce figure-level GO enrichment tables (e.g., Fig. 4C of Ohhata et al. 2022) by submitting gene lists extracted from CCIVR's differentially expressed cis-NAT partner genes to DAVID v6.8 GOTERM_BP_FAT with a matched background, and writing a ranked top-15-per-condition CSV.

## When to use

Use when a paper reports GO enrichment as a figure (e.g., GO BP terms among up- vs down-regulated cis-NAT partner genes) and the goal is a reproducible enrichment table rather than just a visual interpretation.

## When NOT to use

- The input is not differentially expressed partner genes from a CCIVR-style cis-NAT table but raw RNA-seq counts — first run mapping (STAR), expression profiling (RSEM), and DESeq2 differential analysis to obtain fold_change/padj.
- The target figure reports non-GO enrichment (e.g., KEGG pathways or motif analysis), since this skill reproduces GOTERM_BP_FAT biological-process terms specifically.
- No matched background gene set exists; DAVID enrichment requires the Huh-7 dataset genes (or equivalent) as the background universe, and results are not interpretable without it.

## Inputs

- ccivr_output/Table.csv listing cis-NAT pairs with structural characteristics (EB/FO/HH/TT) and expression columns (TPM, FPKM, fold_change, padj)
- Gene identifier lists for up-regulated and down-regulated differentially expressed cis-NAT partner genes (Ensembl-style IDs)
- Huh-7 test file gene annotation columns (for ID conversion and background/universe definition)
- DAVID v6.8 Functional Annotation Tool access

## Outputs

- go_bp_enrichment_top15.csv with exactly 15 GOTERM_BP_FAT terms per condition (up- and down-regulated), each with term name, gene count, and Benjamini/FDR-corrected p-value

## How to apply

Load the prior CCIVR output (ccivr_output/Table.csv generated from the Huh-7 TGFβ-stimulation Test_file.csv) with Python/pandas and extract Ensembl-style gene IDs of the differentially expressed cis-NAT partner genes, split into up-regulated and down-regulated lists using fold_change/padj columns. Map the Ensembl IDs to DAVID-recognized identifiers (official gene symbols or Ensembl Gene ID) using the annotation columns of the Huh-7 test file. Submit each list to the DAVID v6.8 Functional Annotation Tool with the Huh-7 dataset genes as the background/universe, run GOTERM_BP_FAT separately per direction, rank by Benjamini/FDR-corrected p-value, and keep the top 15 terms per list. Write term names, gene counts, and corrected p-values to go_bp_enrichment_top15.csv, validating against the figure's biological pattern (TGF-β signaling/EMT terms enriched among up-regulated genes; cell-growth-related terms among down-regulated genes).

## Related tools

- **CCIVR** (Produces the Table.csv of cis-NAT pairs whose differentially expressed partner genes are extracted for enrichment analysis) — https://github.com/CCIVR/ccivr
- **DAVID** (Functional Annotation Tool used to run GOTERM_BP_FAT enrichment on up- and down-regulated gene lists with the Huh-7 background)
- **Python** (Used with pandas to load the CCIVR output table, filter DE cis-NAT partner genes, and write the top-15 CSV)

## Examples

```
ccivr data/Test_file.csv --output ccivr_output
```

## Evaluation signals

- The output table contains exactly 15 GOTERM_BP_FAT terms per condition (up-regulated and down-regulated), each with enrichment statistics
- The up-regulated term list reproduces the Fig. 4C pattern of TGF-β signaling/EMT-related GO terms
- The down-regulated term list reproduces the Fig. 4C pattern of cell-growth-related GO terms
- Every reported term carries a Benjamini/FDR-corrected p-value and a gene count, and the analysis used the Huh-7 dataset genes as background
- Gene IDs in the DAVID submission were converted from Ensembl-style IDs to a DAVID-recognized format using the test file's annotation columns

## Limitations

- CCIVR's expression analysis depends on third-party processed RNA-seq inputs (STAR mapping, RSEM expression, DESeq2 statistics); the quality of the DE gene lists feeding DAVID inherits those dependencies.
- CCIVR predictions depend on the accuracy of gene annotation (structure and strand direction) deposited in databases such as Ensembl.
- CCIVR output uses Ensembl-style gene IDs (e.g., ENSMUSG00000085715), which must be converted before DAVID submission; incorrect ID mapping or a mismatched background universe will distort enrichment results.

## Evidence

- [results] GO analysis of differentially expressed cis-NAT pair genes upon TGFβ stimulation showed that TGFβ signaling-related genes (including EMT genes) were enriched among up-regulated genes and cell growth-related genes among down-regulated genes.: [evidence span withheld — anchor status: not_found]
- [readme] The test file contains human gene datasets accompanied by expression profiling of human hepatoma Huh-7 cells with TGFß stimulation: "The test file contains human gene datasets accompanied by expression profiling of human hepatoma Huh-7 cells with TGFß stimulation"
- [readme] Two CSV formatted output files "Table.csv" and "Summary.csv" will be generated and saved in a new directory named "ccivr_output": "Two CSV formatted output files **"Table.csv"** and **"Summary.csv"** will be generated and saved in a new directory named **"ccivr_output"**"
- [discussion] it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis: "it uses processed RNA-seq data using third-party programs such as STAR51 for mapping, RSEM32 for expression profiling, and DESeq252 for statistical analysis"
