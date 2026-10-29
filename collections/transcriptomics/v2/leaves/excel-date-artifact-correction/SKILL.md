---
name: excel-date-artifact-correction
description: Use when a gene expression dataset opened in Excel contains date-like
  tokens (Dec-01, Mar-01–Mar-11, Sep-01–Sep-15) or pre-2017 HGNC symbols (SEPT1, MARC1)
  in its gene-name column that must be restored to updated HGNC names before pathway
  analysis.
when_to_use_negative:
- The dataset has no date-like tokens and uses only current HGNC symbols — no conversion
  is needed
- The gene identifiers are not human HGNC symbols (e.g. mouse symbols, Ensembl IDs,
  protein names) — the mapping targets human HGNC nomenclature
- The gene-name column is not in the first column of the uploaded file and cannot
  be moved there
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Gene Updater
  role: Core web tool that converts old gene names and Excel date-converted terms
    back to new HGNC-recommended gene names
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Streamlit
  role: Framework hosting the date_gene_tool.py app locally (streamlit run date_gene_tool.py)
    or at share.streamlit.io
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: pandas
  role: Python dependency for processing uploaded .csv/.xlsx gene expression data
- name: streamlit-tags
  role: Widget package enabling the user to key in genes of interest and assign ambiguous
    Mar-01/Mar-02 terms
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:fbefc3b441c724b0a3229a71db43265d23f07deb075a23a36bc11da21542e70b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/date-to-gene-converter
  tools:
  - Gene Updater
  - Microsoft Excel
  - Python
  - Streamlit
  - XlsxWriter
  - dateparser
  - inflect
  - numpy
  - openpyxl
  - pandas
  - regex
  - streamlit-tags
  - xlrd
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7714285714285715
n_claims: 35
---

# excel-date-artifact-correction

## Summary

Detects and reverses Excel auto-conversion of human gene names (SEPT1–14, MARC1/2, MARCH1/2, DEC1) into date terms (Sep-01–Sep-15, Mar-01–Mar-11, Dec-01), restoring current HGNC-approved symbols in .csv/.xlsx gene expression files using Gene Updater.

## When to use

Use when a gene expression dataset opened in Excel contains date-like tokens (Dec-01, Mar-01–Mar-11, Sep-01–Sep-15) or pre-2017 HGNC symbols (SEPT1, MARC1) in its gene-name column that must be restored to updated HGNC names before pathway analysis.

## When NOT to use

- The dataset has no date-like tokens and uses only current HGNC symbols — no conversion is needed
- The gene identifiers are not human HGNC symbols (e.g. mouse symbols, Ensembl IDs, protein names) — the mapping targets human HGNC nomenclature
- The gene-name column is not in the first column of the uploaded file and cannot be moved there

## Inputs

- .csv or .xlsx gene expression data files with gene names in the first column
- Optional second column with Gene Description (aids Mar-01/Mar-02 disambiguation)
- demo.csv (pre-loaded demo gene list when no file is uploaded)
- hgnc-symbol-check2.csv (HGNC previous-name to new-symbol conversion file)

## Outputs

- Excel data file with updated HUGO/HGNC gene names replacing date-converted and old symbols
- User-assigned mappings for ambiguous Mar-01 (MTARC1 or MARCHF1) and Mar-02 (MTARC2 or MARCHF2) terms

## How to apply

Load the gene-name column from the .csv/.xlsx file (gene names must be in the first column) and identify date-converted terms by sorting the column in ascending order — date symbols appear as numbers at the top. Run date_gene_tool.py (Streamlit) locally with demo.csv and hgnc-symbol-check2.csv in the same folder, or use the hosted instance; old gene names are automatically mapped to new HGNC symbols via the hgnc-symbol-check2.csv previous-name to new-symbol correspondences. For the ambiguous terms Mar-01 and Mar-02, which can each map to two genes with contrasting biological functions (MTARC1/MARCHF1 and MTARC2/MARCHF2), use the Streamlit dropdown/streamlit-tags widgets to assign the correct gene — use the Gene Description column if present, otherwise check the raw dataset; duplicate Mar-01 values are labeled Mar-01_1st and Mar-01_2nd for individual assignment. Verify the correction by keying in genes of interest (e.g. MARCHF1) to confirm the expression data updated to the new gene names, then download the output Excel file for downstream analysis.

## Related tools

- **Gene Updater** (Core web tool that converts old gene names and Excel date-converted terms back to new HGNC-recommended gene names) — https://github.com/kuanrongchan/date-to-gene-converter
- **Streamlit** (Framework hosting the date_gene_tool.py app locally (streamlit run date_gene_tool.py) or at share.streamlit.io) — https://github.com/kuanrongchan/date-to-gene-converter
- **pandas** (Python dependency for processing uploaded .csv/.xlsx gene expression data)
- **streamlit-tags** (Widget package enabling the user to key in genes of interest and assign ambiguous Mar-01/Mar-02 terms)

## Examples

```
pip install -r requirements.txt; streamlit run date_gene_tool.py
```

## Evaluation signals

- All 41 previous-name/Excel-date/HUGO-gene correspondences from Table 1 are covered, e.g. DEC1→DELEC1, MARCH3-11→MARCHF3-11, SEPT1-14→SEPTIN1-14, SEP15→SELENOF
- Every mapped symbol matches HGNC-approved names listed in hgnc-symbol-check2.csv
- All date terms except Mar-01 and Mar-02 map one-to-one to a unique HGNC gene
- Keying in a gene of interest (e.g. MARCHF1) confirms the gene expression data has been updated with the new gene names
- Duplicate Mar-01 values are correctly distinguished as Mar-01_1st and Mar-01_2nd and individually assigned

## Limitations

- Mar-01 and Mar-02 are inherently ambiguous — each can map to two genes (MTARC1/MARCHF1 and MTARC2/MARCHF2) with contrasting biological functions, so user adjudication using gene descriptions or the raw dataset is mandatory
- If no Gene Description column is provided, users must manually check the raw dataset to ascertain what Mar-01_1st/Mar-01_2nd mean
- Excel date formatting is heterogeneous (e.g. Truke only handles dd/mm/yy and one file at a time); mixed date formats across multiple files require the tool's built-in format handling
- Approximately 50–100 research articles still carry gene name errors monthly, indicating the correction must be re-checked for any Excel-processed supplementary file

## Evidence

- [abstract] a web tool with Streamlit that can convert old gene names and dates back into the new gene names recommended by HGNC: "a web tool with Streamlit that can convert old gene names and dates back into the new gene names recommended by HGNC"
- [intro] The conversion of Mar-01 to either MTARC1 or MARCHF1, as well as Mar-02 to either MTARC2 or MARCHF2 can be assigned by the user within the Gene Updater web tool.: "The conversion of Mar-01 to either MTARC1 or MARCHF1, as well as Mar-02 to either MTARC2 or MARCHF2 can be assigned by the user within the Gene Updater web tool."
- [methods] download the following 2 files and place the 2 files within the date_gene_tool folder: "download the following 2 files and place the 2 files within the date_gene_tool folder: 1. demo.csv"
- [discussion] The fastest way to spot these errors is by sorting the column of gene names in ascending order: "The fastest way to spot these errors is by sorting the column of gene names in ascending order, and the gene symbols that are converted to dates will appear as numbers at the top of the column."
- [readme] Users can upload their .csv or .xlsx file or files. Ensure that the first column contains the gene names.: "Users can upload their .csv or .xlsx file or files. Ensure that the first column contains the gene names."
- [readme] When there are duplicate Mar-01 values, Mar-01 will be annotated as Mar-01_1st and Mar-01_2nd.: [evidence span withheld — anchor status: external_doc]
