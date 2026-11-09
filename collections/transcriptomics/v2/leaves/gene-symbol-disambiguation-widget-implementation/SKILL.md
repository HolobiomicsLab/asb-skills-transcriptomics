---
name: gene-symbol-disambiguation-widget-implementation
description: Use when building or extending Gene Updater–style date-to-gene conversion
  and input data contains duplicate Mar-01 or Mar-02 terms that could each map to
  two genes (MTARC1/MARCHF1, MTARC2/MARCHF2), requiring user-assigned resolution.
when_to_use_negative:
- The input gene column contains no date-converted values or no duplicate Mar-01/Mar-02
  terms — automatic conversion of old gene names suffices and no widget-based disambiguation
  is needed.
- The data was never opened in Excel and gene symbols are already valid HGNC names
  — re-labeling would add no value.
- The goal is bulk programmatic conversion of unambiguous date terms (all except Mar-01/Mar-02)
  — the tool converts these automatically without user widgets.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Gene Updater
  role: Web/local tool whose date-to-gene conversion pipeline hosts the duplicate
    Mar-01/Mar-02 relabeling and user-assignment widgets
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Streamlit
  role: Framework for date_gene_tool.py, providing the checkbox/data display and the
    dropdown widget (streamlit-tags/selectbox) used to assign each suffixed date term
    to its gene
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: streamlit-tags
  role: Streamlit package installed locally to support the widget components of date_gene_tool.py
- name: pandas
  role: Loads demo.csv / hgnc-symbol-check2.csv and the user table; used to detect
    and suffix duplicate Mar-01/Mar-02 values in the first gene-name column
- name: XlsxWriter
  role: Exports the user-resolved gene expression table as the updated Excel output
    file
- name: openpyxl
  role: Local package supporting Excel reading/writing in date_gene_tool.py
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:b45d2f709d1ab900b3dbc0665b7db8b19917c2814b9f906e6f7199b911169799
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
  - streamlit-tags
  - xlrd
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 15
---

# gene-symbol-disambiguation-widget-implementation

## Summary

Implements the duplicate Mar-01/Mar-02 disambiguation mechanism of Gene Updater: duplicate date-converted gene terms are relabeled (e.g. Mar-01_1st, Mar-01_2nd) and presented to the user through Streamlit widgets so each instance can be assigned to the correct HGNC gene (MARCHF1/MTARC1 or MARCHF2/MTARC2) before export.

## When to use

Use when building or extending Gene Updater–style date-to-gene conversion and input data contains duplicate Mar-01 or Mar-02 terms that could each map to two genes (MTARC1/MARCHF1, MTARC2/MARCHF2), requiring user-assigned resolution.

## When NOT to use

- The input gene column contains no date-converted values or no duplicate Mar-01/Mar-02 terms — automatic conversion of old gene names suffices and no widget-based disambiguation is needed.
- The data was never opened in Excel and gene symbols are already valid HGNC names — re-labeling would add no value.
- The goal is bulk programmatic conversion of unambiguous date terms (all except Mar-01/Mar-02) — the tool converts these automatically without user widgets.

## Inputs

- User gene expression table in .csv or .xlsx with gene names in the first column (a Gene Description in the second column is useful for mapping Mar-01/Mar-02 but is not mandatory)
- HGNC symbol conversion file (hgnc-symbol-check2.csv)
- Demo dataset (demo.csv) pre-loaded when no user file is uploaded
- date_gene_tool.py script and Python packages: pandas, numpy, regex, inflect, dateparser, openpyxl, xlrd, XlsxWriter, streamlit-tags, Streamlit

## Outputs

- Gene expression table with updated HUGO gene names where every duplicate Mar-01_1st/Mar-01_2nd and Mar-02_1st/Mar-02_2nd instance has been replaced by its user-assigned gene (MARCHF1/MTARC1 or MARCHF2/MTARC2)
- Downloadable Excel output file for further downstream analysis

## How to apply

In date_gene_tool.py, load the user's .csv or .xlsx (gene names in the first column) plus hgnc-symbol-check2.csv, and detect duplicate date terms in the first gene-name column. Suffix each occurrence to make ambiguous values distinguishable (Mar-01_1st/Mar-01_2nd and analogously Mar-02_1st/Mar-02_2nd). Build Streamlit widgets (selectbox/streamlit-tags) so the user manually assigns each suffixed value to its gene (Mar-01_1st -> MARCHF1 or MTARC1; Mar-02_1st -> MARCHF2 or MTARC2); if a gene description exists in column 2, the user matches gene name to description, otherwise they must check the raw dataset. Write the user-resolved gene column back into the gene expression table and export it as an Excel file with XlsxWriter/openpyxl. Validate by letting the user key in a gene of interest (e.g. MARCHF1) and confirming the expression data shows the updated non-date gene labels.

## Related tools

- **Gene Updater** (Web/local tool whose date-to-gene conversion pipeline hosts the duplicate Mar-01/Mar-02 relabeling and user-assignment widgets) — https://github.com/kuanrongchan/date-to-gene-converter
- **Streamlit** (Framework for date_gene_tool.py, providing the checkbox/data display and the dropdown widget (streamlit-tags/selectbox) used to assign each suffixed date term to its gene) — https://github.com/kuanrongchan/date-to-gene-converter
- **streamlit-tags** (Streamlit package installed locally to support the widget components of date_gene_tool.py)
- **pandas** (Loads demo.csv / hgnc-symbol-check2.csv and the user table; used to detect and suffix duplicate Mar-01/Mar-02 values in the first gene-name column)
- **XlsxWriter** (Exports the user-resolved gene expression table as the updated Excel output file)
- **openpyxl** (Local package supporting Excel reading/writing in date_gene_tool.py)

## Examples

```
cd path/to/folder; pip install -r requirements.txt; streamlit run date_gene_tool.py
```

## Evaluation signals

- Run 'streamlit run date_gene_tool.py' with demo.csv: the app executes without errors and the demo dataset with duplicate Mar-01/Mar-02 values is annotated as Mar-01_1st/Mar-01_2nd (and Mar-02_1st/Mar-02_2nd).
- Each suffixed value is resolvable to exactly one gene via the dropdown widget (Mar-01_1st -> MARCHF1 or MTARC1; Mar-02_1st -> MARCHF2 or MTARC2) and no suffixed label remains in the final output table.
- Querying a gene of interest (e.g. MARCHF1) in the app returns expression data under the updated (non-date) gene name.
- The exported Excel file contains updated HUGO gene names for all formerly date-converted entries.
- If a Gene Description column was provided, each assigned gene matches its description (e.g. MARCHF1/MTARC1 functions), since these genes have contrasting biological functions.

## Limitations

- Mar-01 and Mar-02 can each map to two genes (MTARC1/MARCHF1 and MTARC2/MARCHF2) with contrasting biological functions, so disambiguation cannot be automated — user assignment is mandatory.
- If no gene description is provided in column 2, users must check the raw dataset to ascertain what Mar-01_1st and Mar-01_2nd mean.
- Python package versions are not pinned ('compatible versions' unspecified), which may cause local installation issues.
- The disambiguation only covers Mar-01/Mar-02; other date-converted gene names are converted automatically and their correctness depends on the mapping file.

## Evidence

- [readme] Duplicate Mar-01 terms relabeled with ordinal suffixes and user assignment via widgets: [evidence span withheld — anchor status: external_doc]
- [readme] Gene description column aids mapping of Mar-01/Mar-02: "Having a Gene Description on the second column is useful for mapping the Mar-01 and Mar-02 genes, although this step is not mandatory."
- [intro] Article workflow step: dropdown assignment of Mar-01/Mar-02: "The conversion of Mar-01 to either MTARC1 or MARCHF1, as well as Mar-02 to either MTARC2 or MARCHF2 can be assigned by the user within the Gene Updater web tool."
- [discussion] Biological rationale: MARCHF/MTARC genes have contrasting functions: "if the genes are auto-converted to Mar-01 or Mar-02, it is challenging to interpret whether the genes are either MTARC1-2 or MARCHF1-2, which are known to have contrasting biological functions."
- [discussion] Local run command for validation: "users can then simply type: streamlit run date_gene_tool.py. This will generate a new tab with the Gene Updater web tool appearing in the default browser."
