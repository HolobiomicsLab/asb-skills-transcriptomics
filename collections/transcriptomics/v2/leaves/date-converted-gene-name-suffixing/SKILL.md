---
name: date-converted-gene-name-suffixing
description: Use when a gene-name column contains duplicate Mar-01 or Mar-02 date
  terms that each map to more than one gene (MTARC1/MARCHF1 or MTARC2/MARCHF2) and
  the instances must be made distinguishable before user resolution.
when_to_use_negative:
- The date-converted term is unambiguous (e.g. Mar-03 onward maps to a single gene)
  — these are auto-converted directly by Gene Updater without suffixing or manual
  assignment
- The gene-name column contains no date-converted values (e.g. it was never opened
  in Excel under default settings) — use the tool's plain old-name-to-new-HGNC-name
  conversion instead
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Gene Updater
  role: Web tool implementing duplicate-date-term suffixing and user resolution of
    Mar-01/Mar-02 to HGNC gene names
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Streamlit
  role: Framework hosting the app and the dropdown (selectbox/streamlit-tags) widgets
    used for manual gene assignment
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: pandas
  role: Loads the uploaded .csv/.xlsx gene table and the hgnc-symbol-check2.csv conversion
    file for duplicate detection and column rewriting
- name: XlsxWriter
  role: Exports the resolved gene expression table as an Excel output file
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:b0fb40f856560eac5d9e5eeefff8ef82f50016850410c187db242e5ebec1716e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/date-to-gene-converter
  tools:
  - Gene Updater
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
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8947368421052632
n_claims: 19
---

# date-converted-gene-name-suffixing

## Summary

Relabels duplicate Excel date-converted gene names (Mar-01, Mar-02) with ordinal suffixes (Mar-01_1st, Mar-01_2nd, etc.) so that each ambiguous instance can be manually assigned to its correct HGNC gene (MTARC1/MARCHF1 or MTARC2/MARCHF2) via Streamlit dropdown widgets in Gene Updater.

## When to use

Use when a gene-name column contains duplicate Mar-01 or Mar-02 date terms that each map to more than one gene (MTARC1/MARCHF1 or MTARC2/MARCHF2) and the instances must be made distinguishable before user resolution.

## When NOT to use

- The date-converted term is unambiguous (e.g. Mar-03 onward maps to a single gene) — these are auto-converted directly by Gene Updater without suffixing or manual assignment
- The gene-name column contains no date-converted values (e.g. it was never opened in Excel under default settings) — use the tool's plain old-name-to-new-HGNC-name conversion instead

## Inputs

- Excel-mangled gene expression table (.csv or .xlsx) with gene names in the first column containing duplicate Mar-01 and/or Mar-02 date-converted values
- Optional Gene Description column (second column) for mapping Mar-01/Mar-02 instances to genes
- HGNC symbol conversion file (hgnc-symbol-check2.csv)

## Outputs

- Gene-name column with duplicates relabeled as Mar-01_1st/Mar-01_2nd and Mar-02_1st/Mar-02_2nd
- User-resolved gene labels (MTARC1 or MARCHF1 for Mar-01; MTARC2 or MARCHF2 for Mar-02) written back into the table
- Downloadable Excel file with updated HUGO gene names for downstream analysis

## How to apply

Detect duplicate Mar-01 and Mar-02 values in the first gene-name column of the uploaded .csv or .xlsx file (loaded with pandas); unlike unambiguous date terms that convert directly, these are not auto-converted because each can map to two genes. Suffix each occurrence as Mar-01_1st, Mar-01_2nd (analogously Mar-02_1st, Mar-02_2nd) so the duplicate values become distinguishable. Build Streamlit selectbox/streamlit-tags widgets so the user manually assigns each suffixed value to its corresponding gene: Mar-01 to MTARC1 or MARCHF1, Mar-02 to MTARC2 or MARCHF2. If a Gene Description is present in the second column, the user matches the gene name to the description; otherwise they must check the raw dataset to ascertain what Mar-01_1st and Mar-01_2nd mean. Write the user-resolved gene labels back into the gene expression table and export the updated table as Excel (XlsxWriter/openpyxl) with updated HUGO gene names.

## Related tools

- **Gene Updater** (Web tool implementing duplicate-date-term suffixing and user resolution of Mar-01/Mar-02 to HGNC gene names) — https://github.com/kuanrongchan/date-to-gene-converter
- **Streamlit** (Framework hosting the app and the dropdown (selectbox/streamlit-tags) widgets used for manual gene assignment) — https://github.com/kuanrongchan/date-to-gene-converter
- **pandas** (Loads the uploaded .csv/.xlsx gene table and the hgnc-symbol-check2.csv conversion file for duplicate detection and column rewriting)
- **XlsxWriter** (Exports the resolved gene expression table as an Excel output file)

## Examples

```
streamlit run date_gene_tool.py
```

## Evaluation signals

- Duplicate Mar-01 values appear exactly as Mar-01_1st and Mar-01_2nd (and Mar-02 as Mar-02_1st/Mar-02_2nd) in the first gene-name column — no unsuffixed duplicates remain
- After widget assignment, no Mar-01*/Mar-02* date labels remain in the output table; they are replaced by MTARC1/MARCHF1 and MTARC2/MARCHF2
- Running 'streamlit run date_gene_tool.py' on demo.csv executes without error
- Keying in a gene of interest (e.g. MARCHF1) in the app displays expression data under the updated (non-date) gene name
- The downloaded Excel output opens in Excel and the resolved gene names are preserved as text, not re-converted to dates

## Limitations

- Mar-01 and Mar-02 cannot be auto-resolved: if no Gene Description column is provided, the user must consult the raw dataset to determine what Mar-01_1st and Mar-01_2nd mean, since MTARC1-2 and MARCHF1-2 have contrasting biological functions
- Manual date-to-text conversion and gene assignment can be tedious and error-prone if the raw dataset lacks gene descriptions
- Python package dependency versions are not pinned in the discussion ('compatible versions' unspecified), which may affect local reproducibility

## Evidence

- [readme] Duplicate Mar-01 terms are relabeled with ordinal suffixes: [evidence span withheld — anchor status: external_doc]
- [abstract] Ambiguous date terms map to two genes each and require user assignment: "as Mar-01 and Mar-02 can each be potentially mapped to 2 different gene names, users can assign the date terms to the appropriate gene names within the Gene Updater web tool."
- [intro] Dropdown widget resolution of Mar-01/Mar-02 to MTARC/MARCHF genes: "The conversion of Mar-01 to either MTARC1 or MARCHF1, as well as Mar-02 to either MTARC2 or MARCHF2 can be assigned by the user within the Gene Updater web tool."
- [discussion] Contrasting functions of the candidate genes make disambiguation critical: "if the genes are auto-converted to Mar-01 or Mar-02, it is challenging to interpret whether the genes are either MTARC1-2 or MARCHF1-2, which are known to have contrasting biological functions."
- [intro] Output is an Excel file with updated HUGO gene names: "The output is an Excel data file containing the updated HUGO gene names which can be downloaded for further downstream analysis."
