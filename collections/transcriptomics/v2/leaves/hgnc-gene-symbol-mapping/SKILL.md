---
name: hgnc-gene-symbol-mapping
description: Use when a gene-expression table (.csv/.xlsx) has a gene-name column
  containing Excel date artifacts (Dec-01, Mar-01..Mar-11, Sep-01..Sep-15) or pre-2017
  HGNC symbols that must be converted to current HGNC names before pathway analysis.
when_to_use_negative:
- The gene-name column already contains current HGNC-approved symbols and contains
  no Excel date artifacts (e.g. Mar-01, SEPT1) — no conversion is needed
- The data are non-human gene names — the mapping targets HGNC (HUGO Gene Nomenclature
  Committee) human gene nomenclature
- The first column of the uploaded file does not contain gene names — Gene Updater
  expects gene names in the first column
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Gene Updater
  role: Web tool (date_gene_tool.py) that autocorrects Excel date-converted gene names
    and updates old gene names to HGNC-recommended symbols; resolves duplicates via
    dropdown widgets
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Streamlit
  role: Framework hosting the Gene Updater app locally ('streamlit run date_gene_tool.py')
    and providing the widget interface for assigning ambiguous Mar-01/Mar-02 terms
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: pandas
  role: Loads demo.csv and hgnc-symbol-check2.csv to build and validate the previous-name
    to new-HGNC-symbol mapping table with Python
  repo: https://github.com/kuanrongchan/date-to-gene-converter
provenance:
  source_task_ids:
  - task_002
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:0265530c4e6fd1d9c48c24310ef50ac5728ed5b9b844bb98c7a3fc4e859d45dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/date-to-gene-converter
  tools:
  - Gene Updater
  - Mark Ziemann's gene name errors dashboard
  - Python
  - Streamlit
  - XlsxWriter
  - dateparser
  - inflect
  - numpy
  - openpyxl
  - pandas
  - regex
  - xlrd
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# hgnc-gene-symbol-mapping

## Summary

Maps Excel date-converted and outdated gene names (e.g. SEPT1, MARC1, Mar-01) back to current HGNC-approved human gene symbols using Gene Updater and the hgnc-symbol-check2.csv conversion file, resolving the ambiguous Mar-01/Mar-02 terms via manual user assignment.

## When to use

Use when a gene-expression table (.csv/.xlsx) has a gene-name column containing Excel date artifacts (Dec-01, Mar-01..Mar-11, Sep-01..Sep-15) or pre-2017 HGNC symbols that must be converted to current HGNC names before pathway analysis.

## When NOT to use

- The gene-name column already contains current HGNC-approved symbols and contains no Excel date artifacts (e.g. Mar-01, SEPT1) — no conversion is needed
- The data are non-human gene names — the mapping targets HGNC (HUGO Gene Nomenclature Committee) human gene nomenclature
- The first column of the uploaded file does not contain gene names — Gene Updater expects gene names in the first column

## Inputs

- .csv or .xlsx gene expression file with gene names in the first column (gene description in the second column is optional but helps disambiguate Mar-01/Mar-02)
- hgnc-symbol-check2.csv (HGNC previous-name to new-symbol conversion table from the date-to-gene-converter GitHub/Zenodo deposit)
- demo.csv (pre-loaded restricted gene list for exploring tool features)

## Outputs

- Excel (.xlsx) file with gene-name column converted to updated HGNC gene symbols
- Complete date-term-to-HGNC mapping table (41 rows; date_to_gene_mapping.csv) covering Dec-01, Mar-01..Mar-11, Sep-01..Sep-15

## How to apply

Upload the .csv or .xlsx file (or use the pre-loaded demo.csv) with gene names in the first column; Gene Updater auto-converts old names and date terms to new HGNC symbols using hgnc-symbol-check2.csv. Sort the gene column in ascending order to spot date-converted symbols appearing as numbers at the top. Treat Mar-01 and Mar-02 as ambiguous — each maps to two genes (MTARC1/MARCHF1 and MTARC2/MARCHF2) with contrasting biological functions — so suffix duplicates as Mar-01_1st/Mar-01_2nd (and Mar-02_1st/Mar-02_2nd) and assign the correct gene via Streamlit dropdown widgets, using an accompanying gene description column when available. Export the result as an Excel file and verify by keying in a gene of interest (e.g. MARCHF1) to confirm the expression data shows updated gene names.

## Related tools

- **Gene Updater** (Web tool (date_gene_tool.py) that autocorrects Excel date-converted gene names and updates old gene names to HGNC-recommended symbols; resolves duplicates via dropdown widgets) — https://github.com/kuanrongchan/date-to-gene-converter
- **Streamlit** (Framework hosting the Gene Updater app locally ('streamlit run date_gene_tool.py') and providing the widget interface for assigning ambiguous Mar-01/Mar-02 terms) — https://github.com/kuanrongchan/date-to-gene-converter
- **pandas** (Loads demo.csv and hgnc-symbol-check2.csv to build and validate the previous-name to new-HGNC-symbol mapping table with Python) — https://github.com/kuanrongchan/date-to-gene-converter

## Examples

```
streamlit run date_gene_tool.py
```

## Evaluation signals

- The produced mapping table contains all 41 previous-name/Excel-date/HUGO-gene rows and every mapped symbol matches an HGNC-approved name in hgnc-symbol-check2.csv
- Dec-01, MARCH3-11, SEPT1-14, and SEP15 map one-to-one to DELEC1, MARCHF3-11, SEPTIN1-14, and SELENOF respectively; only Mar-01 and Mar-02 remain two-candidate ambiguous
- Duplicate ambiguous values are distinctly suffixed (Mar-01_1st, Mar-01_2nd, Mar-02_1st, Mar-02_2nd) before user assignment
- Keying in a gene of interest (e.g. MARCHF1) in the running app displays expression data under the updated (non-date) gene labels
- The output Excel file downloads and opens with the converted gene-name column, suitable for downstream Enrichr/GSEA pathway analysis

## Limitations

- Mar-01 and Mar-02 cannot be autocorrected without an accompanying unique identifier such as a gene description; these were the only failure cases in the June 2022 benchmark (78.6% autocorrected)
- Mar-01/Mar-02 ambiguity is biologically consequential: MTARC1-2 and MARCHF1-2 have contrasting biological functions, so wrong assignment changes pathway interpretation
- Manual date-to-text conversion in Excel is tedious and error-prone; there is no automated safeguard within Excel itself
- Python package dependency versions are not pinned ('compatible versions' unspecified), which may complicate local reproducibility

## Evidence

- [abstract] Streamlit conversion of old names: "a web tool with Streamlit that can convert old gene names and dates back into the new gene names recommended by HGNC"
- [intro] Mar-01/Mar-02 user assignment: "The conversion of Mar-01 to either MTARC1 or MARCHF1, as well as Mar-02 to either MTARC2 or MARCHF2 can be assigned by the user within the Gene Updater web tool."
- [readme] Date-converted terms not auto-mappable: "If the first column contains dates, they will be converted to the updated gene names, with the exception of Mar-01 and Mar-02 as these terms can be mapped to more than one gene."
- [methods] Required files download: "In the GitHub address (https://github.com/kuanrongchan/date-to-gene-converter), download the following 2 files"
- [discussion] Ambiguity biology caveat: "if the genes are auto-converted to Mar-01 or Mar-02, it is challenging to interpret whether the genes are either MTARC1-2 or MARCHF1-2, which are known to have contrasting biological functions."
