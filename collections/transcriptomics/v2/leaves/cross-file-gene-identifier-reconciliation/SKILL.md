---
name: cross-file-gene-identifier-reconciliation
description: Use when a gene expression dataset (.csv or .xlsx) opened in Excel contains
  date-like terms or outdated gene symbols in the first column that fail pathway-database
  lookups (Enrichr, GSEA, Ingenuity) and need mapping to HGNC-approved names.
when_to_use_negative:
- The gene list already uses current HGNC-approved symbols and contains no date-converted
  terms — reconciliation is unnecessary
- The input is not gene nomenclature data (e.g. protein IDs, Ensembl accessions) not
  covered by the hgnc-symbol-check2.csv previous-name mapping
- Ambiguous Mar-01/Mar-02 terms cannot be resolved because neither a gene description
  column nor the raw dataset is available to distinguish MTARC1/MARCHF1 or MTARC2/MARCHF2
edam_operation: http://edamontology.org/operation_3282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: Gene Updater
  role: Streamlit web tool that converts old gene names and Excel date terms back
    into HGNC-recommended gene names; hosts dropdown widgets for assigning ambiguous
    Mar-01/Mar-02
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Streamlit
  role: Framework for running date_gene_tool.py locally (streamlit run date_gene_tool.py)
    or via the shared web instance
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: pandas
  role: Loads hgnc-symbol-check2.csv and demo.csv to build and verify the date-term
    to HGNC-symbol mapping
- name: Microsoft Excel
  role: 'Source of the misidentification problem: opening gene lists under default
    settings converts symbols like SEPT1 and MARC1 to dates; also used to spot errors
    by sorting ascending and to set cell formats to text'
- name: streamlit-tags
  role: Widget package used by the Gene Updater app for entering genes of interest
    to inspect updated expression data
  repo: https://github.com/kuanrongchan/date-to-gene-converter
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:cd337a781312be8f492e09ed97a21f73d8cf92a8c67b61349deb282c53505abd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/date-to-gene-converter
  tools:
  - Gene Updater
  - Microsoft Excel
  - Python
  - Streamlit
  - pandas
  - streamlit-tags
  edam_operation: http://edamontology.org/operation_3282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# cross-file-gene-identifier-reconciliation

## Summary

Reconciles gene identifiers across files by converting Excel date-corrupted terms (Dec-01, Mar-01..Mar-11, Sep-01..Sep-15) and old gene names into current HGNC-approved symbols, using the Gene Updater tool's mapping built from hgnc-symbol-check2.csv.

## When to use

Use when a gene expression dataset (.csv or .xlsx) opened in Excel contains date-like terms or outdated gene symbols in the first column that fail pathway-database lookups (Enrichr, GSEA, Ingenuity) and need mapping to HGNC-approved names.

## When NOT to use

- The gene list already uses current HGNC-approved symbols and contains no date-converted terms — reconciliation is unnecessary
- The input is not gene nomenclature data (e.g. protein IDs, Ensembl accessions) not covered by the hgnc-symbol-check2.csv previous-name mapping
- Ambiguous Mar-01/Mar-02 terms cannot be resolved because neither a gene description column nor the raw dataset is available to distinguish MTARC1/MARCHF1 or MTARC2/MARCHF2

## Inputs

- .csv or .xlsx gene expression files with gene names (possibly Excel date-corrupted, e.g. Dec-01, Mar-01, Sep-01) in the first column
- hgnc-symbol-check2.csv (HGNC previous-name to approved-symbol correspondence table from the repository)
- demo.csv (pre-loaded demo dataset for exploring the web tool)
- optional second-column gene descriptions to disambiguate Mar-01/Mar-02

## Outputs

- Excel data file with updated HUGO/HGNC gene names for downstream analysis
- date-term-to-HGNC-gene mapping table (41 previous-name/Excel-date/HUGO-gene rows, e.g. DEC1→DELEC1, SEPT1-14→SEPTIN1-14, SEP15→SELENOF)
- user-assigned annotations for Mar-01 (MTARC1 or MARCHF1) and Mar-02 (MTARC2 or MARCHF2)

## How to apply

Obtain demo.csv and hgnc-symbol-check2.csv from the kuanrongchan/date-to-gene-converter GitHub deposit (Zenodo 10.5281/zenodo.6845701) and place them alongside date_gene_tool.py. Load hgnc-symbol-check2.csv with pandas to extract previous-name to new-HGNC-symbol correspondences for the affected families (SEPT1-14→SEPTIN1-14, MARC1/2→MTARC1/2, MARCH1/2→MARCHF1/2, DEC1→DELEC1), then cross-reference the date-like terms from the input file to build the mapping. Convert each file by uploading .csv or .xlsx with gene names in the first column; for the two ambiguous terms Mar-01 and Mar-02, which can each map to MTARC1/MARCHF1 and MTARC2/MARCHF2, use the Streamlit dropdown widgets (and duplicate Mar-01 values are annotated Mar-01_1st/Mar-01_2nd) — the gene description in the second column helps disambiguate, otherwise check the raw dataset. Spot residual errors by sorting the gene-name column ascending, since date-converted symbols appear as numbers at the top. Verify by keying in genes of interest (e.g. MARCHF1) to confirm the expression data updated, then download the output Excel file for downstream analysis.

## Related tools

- **Gene Updater** (Streamlit web tool that converts old gene names and Excel date terms back into HGNC-recommended gene names; hosts dropdown widgets for assigning ambiguous Mar-01/Mar-02) — https://github.com/kuanrongchan/date-to-gene-converter
- **Streamlit** (Framework for running date_gene_tool.py locally (streamlit run date_gene_tool.py) or via the shared web instance) — https://github.com/kuanrongchan/date-to-gene-converter
- **pandas** (Loads hgnc-symbol-check2.csv and demo.csv to build and verify the date-term to HGNC-symbol mapping)
- **Microsoft Excel** (Source of the misidentification problem: opening gene lists under default settings converts symbols like SEPT1 and MARC1 to dates; also used to spot errors by sorting ascending and to set cell formats to text)
- **streamlit-tags** (Widget package used by the Gene Updater app for entering genes of interest to inspect updated expression data) — https://github.com/kuanrongchan/date-to-gene-converter

## Examples

```
cd date_gene_tool && pip install -r requirements.txt && streamlit run date_gene_tool.py
```

## Evaluation signals

- The produced mapping contains all 41 rows of Table 1's previous-name/Excel-date/HUGO-gene correspondences (e.g. DEC1→DELEC1, MARCH3-11→MARCHF3-11, SEPT1-14→SEPTIN1-14, SEP15→SELENOF)
- Every mapped symbol matches an HGNC-approved name present in hgnc-symbol-check2.csv; only Mar-01 (MTARC1/MARCHF1) and Mar-02 (MTARC2/MARCHF2) remain user-assigned
- Sorting the output gene-name column ascending shows no numeric (date-derived) entries at the top
- Keying in genes of interest (e.g. MARCHF1) in the app confirms the gene expression data has been updated to the new gene names
- Downstream pathway analysis against GO Biological Processes via Enrichr no longer shows voids from unrecognized date terms

## Limitations

- Mar-01 and Mar-02 are inherently ambiguous: MTARC1-2 and MARCHF1-2 have contrasting biological functions, so users must manually assign; duplicate Mar-01 values become Mar-01_1st/Mar-01_2nd and require checking the raw dataset if no gene description column exists
- Local installs need compatible versions of many packages (pandas, numpy, regex, inflect, dateparser, xlrd, openpyxl, XlsxWriter, streamlit-tags) whose exact versions are not pinned
- Excel's own fix — manually converting date-formatted cells to text via cell formatting — is tedious and error-prone; errors recur in roughly one-fifth of published supplementary Excel gene lists and 50–100 articles per month

## Evidence

- [abstract] Herein, we developed a web tool with Streamlit that can convert old gene names and dates back into the new gene names recommended by HGNC: "Herein, we developed a web tool with Streamlit that can convert old gene names and dates back into the new gene names recommended by HGNC"
- [abstract] Mar-01/Mar-02 ambiguity: "as Mar-01 and Mar-02 can each be potentially mapped to 2 different gene names, users can assign the date terms to the appropriate gene names within the Gene Updater web tool."
- [methods] download files and place alongside tool: "In the GitHub address (https://github.com/kuanrongchan/date-to-gene-converter), download the following 2 files and place the 2 files within the date_gene_tool folder: 1. demo.csv 2."
- [discussion] spot errors by sorting: "The fastest way to spot these errors is by sorting the column of gene names in ascending order, and the gene symbols that are converted to dates will appear as numbers at the top of the column."
- [readme] Mar-01_1st/Mar-01_2nd handling: [evidence span withheld — anchor status: external_doc]
- [readme] verify updated genes of interest: "Finally, users can key in the genes of interest (e.g. MARCHF1) to inspect if the gene expression data has indeed been updated with the new gene names."
