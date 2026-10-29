---
name: excel-date-conversion-gene-list-recovery
description: Use when a .csv or .xlsx gene expression list opened in Excel contains
  gene symbols auto-converted to dates (e.g. Mar-01, Mar-02) or outdated gene names
  that pathway enrichment tools like Enrichr or GSEA cannot recognize.
when_to_use_negative:
- The gene list already contains valid, current HGNC symbols and never passed through
  Excel — no date conversion or name update is needed
- You need to run pathway enrichment on an already-corrected gene list — use pathway
  enrichment tools (e.g. Enrichr against GO Biological Processes) instead of the date-recovery
  workflow
- The 'date-like' entries are legitimate date values in a non-gene column — converting
  them would corrupt real data
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0199
tools:
- name: Gene Updater
  role: Open-source Streamlit web tool (local or hosted) that autocorrects Excel date-converted
    gene names and updates old gene names to HGNC-recommended symbols; users assign
    ambiguous Mar-01/Mar-02 terms via dropdown widgets
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Streamlit
  role: Framework used to build and run the Gene Updater web tool locally (pip install
    streamlit; streamlit run date_gene_tool.py)
- name: Microsoft Excel
  role: 'Source of the error being repaired: opening gene expression files under default
    Excel settings converts gene names to dates; manual cell-format date-to-text conversion
    is the tedious alternative this skill replaces'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:031b4e11d16c5d3f4a4a10c6b1e900d384c06ad8974723191cf4a4e51361ec03
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/date-to-gene-converter
  tools:
  - Enrichr
  - Gene Ontology (GO) Biological Processes database
  - Gene Updater
  - Microsoft Excel
  - Streamlit
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0199
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# excel-date-conversion-gene-list-recovery

## Summary

Recover gene lists corrupted by Excel's automatic conversion of gene symbols to dates (e.g. SEPT1, MARC1 becoming 1-Sep, 1-Mar) using the Gene Updater Streamlit web tool, and convert old gene names back to HGNC-recommended symbols so pathway databases can recognize them.

## When to use

Use when a .csv or .xlsx gene expression list opened in Excel contains gene symbols auto-converted to dates (e.g. Mar-01, Mar-02) or outdated gene names that pathway enrichment tools like Enrichr or GSEA cannot recognize.

## When NOT to use

- The gene list already contains valid, current HGNC symbols and never passed through Excel — no date conversion or name update is needed
- You need to run pathway enrichment on an already-corrected gene list — use pathway enrichment tools (e.g. Enrichr against GO Biological Processes) instead of the date-recovery workflow
- The 'date-like' entries are legitimate date values in a non-gene column — converting them would corrupt real data

## Inputs

- .csv or .xlsx file(s) with gene names in the first column, containing Excel date-converted symbols (Mar-01, Mar-02, etc.) and/or outdated gene names
- Optional: hgnc-symbol-check2.csv and demo.csv from github.com/kuanrongchan/date-to-gene-converter for local deployment and testing
- Optional: gene description column or raw dataset to disambiguate Mar-01/Mar-02 assignments

## Outputs

- Excel data file containing updated HGNC-recommended gene names, ready for downstream pathway enrichment (e.g. Enrichr) or data sharing
- User-assigned mapping of ambiguous Mar-01/Mar-02 terms to MTARC1/MARCHF1 and MTARC2/MARCHF2

## How to apply

First spot the damage by sorting the gene-name column in ascending order — date-converted symbols appear as numbers at the top of the column. Then load the list in the Gene Updater web tool (Streamlit app at https://share.streamlit.io/kuanrongchan/date-to-gene-converter/main/date_gene_tool.py, or run locally with `streamlit run date_gene_tool.py` after installing pandas, numpy, regex, inflect, dateparser, openpyxl, xlrd, XlsxWriter and streamlit-tags), ensuring gene names are in the first column; old names are automatically updated to new HGNC names. Ambiguous Mar-01 and Mar-02 terms must be manually assigned via dropdown widgets to MTARC1/MARCHF1 and MTARC2/MARCHF2 respectively — a gene description in the second column helps match names, and duplicate Mar-01 values are annotated as Mar-01_1st and Mar-01_2nd for individual assignment. Download the output Excel file with updated HUGO gene names, and verify by keying in genes of interest (e.g. MARCHF1) to confirm the data was updated before downstream pathway analysis.

## Related tools

- **Gene Updater** (Open-source Streamlit web tool (local or hosted) that autocorrects Excel date-converted gene names and updates old gene names to HGNC-recommended symbols; users assign ambiguous Mar-01/Mar-02 terms via dropdown widgets) — https://github.com/kuanrongchan/date-to-gene-converter
- **Streamlit** (Framework used to build and run the Gene Updater web tool locally (pip install streamlit; streamlit run date_gene_tool.py))
- **Microsoft Excel** (Source of the error being repaired: opening gene expression files under default Excel settings converts gene names to dates; manual cell-format date-to-text conversion is the tedious alternative this skill replaces)

## Examples

```
cd date_gene_tool && streamlit run date_gene_tool.py
```

## Evaluation signals

- After sorting the corrected gene-name column in ascending order, no numeric/date entries appear at the top of the column
- No unresolved Mar-01 or Mar-02 tokens remain unless the user has explicitly assigned each (including Mar-01_1st/Mar-01_2nd duplicates) to MTARC1/MARCHF1 or MTARC2/MARCHF2
- Keying in genes of interest (e.g. MARCHF1) in the Gene Updater checker confirms the expression data was updated with the new gene names
- All symbols in the output file are recognized by downstream pathway tools (Enrichr, GSEA, Ingenuity Pathway Analysis) which rely on gene symbols to query GO and Reactome
- Output Excel file contains updated HUGO gene names suitable for further downstream analysis and data sharing

## Limitations

- Mar-01 and Mar-02 can each map to two genes (MTARC1 vs MARCHF1, MTARC2 vs MARCHF2) with contrasting biological functions, so user judgment or gene descriptions/raw data are required for correct assignment
- Manual date-to-text conversion within Excel is a tedious and error-prone alternative, and Excel itself offers no automated safeguard
- Local installation requires Python 3.7 or later and multiple packages, but 'compatible versions' are not pinned (pandas >= 1.2.5 is the only version stated)
- If no gene description column is provided, users must check the raw dataset to ascertain what Mar-01_1st and Mar-01_2nd mean

## Evidence

- [abstract] Herein, we developed a web tool with Streamlit that can convert old gene names and dates back into the new gene names recommended by HGNC: "Herein, we developed a web tool with Streamlit that can convert old gene names and dates back into the new gene names recommended by HGNC"
- [discussion] Mar-01/Mar-02 ambiguity: "if the genes are auto-converted to Mar-01 or Mar-02, it is challenging to interpret whether the genes are either MTARC1-2 or MARCHF1-2, which are known to have contrasting biological functions."
- [discussion] spotting the errors: "The fastest way to spot these errors is by sorting the column of gene names in ascending order, and the gene symbols that are converted to dates will appear as numbers at the top of the column."
- [readme] input format requirements: "Users can upload their .csv or .xlsx file or files. Ensure that the first column contains the gene names. Having a Gene Description on the second column is useful for mapping the Mar-01 and Mar-02"
- [readme] verification step: "Finally, users can key in the genes of interest (e.g. MARCHF1) to inspect if the gene expression data has indeed been updated with the new gene names."
