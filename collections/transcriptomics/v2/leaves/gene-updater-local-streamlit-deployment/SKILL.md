---
name: gene-updater-local-streamlit-deployment
description: Use when csv or .xlsx gene expression files contain gene symbols auto-converted
  to dates by Excel (e.g. SEPT1, MARC1 becoming date strings, or ambiguous Mar-01/Mar-02
  terms) and need to be converted back to HGNC-approved gene names.
when_to_use_negative:
- Input files contain no Excel date-converted gene names and already use current HGNC
  gene symbols — the tool is designed to rectify date conversions and old names, not
  general gene expression analysis.
- Ambiguous Mar-01/Mar-02 terms cannot be resolved because no gene description is
  provided and the raw dataset cannot disambiguate MTARC1/MARCHF1 from MTARC2/MARCHF2
  — manual assignment via the widgets would be guesswork with contrasting biological
  functions at stake.
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Gene Updater
  role: Web tool being deployed locally; converts old gene names and Excel date-converted
    terms back to new HGNC-recommended gene names
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Streamlit
  role: Framework used to build and run the date_gene_tool.py web app locally via
    'streamlit run'
- name: Python
  role: Programming language in which date_gene_tool.py is written and executed locally
- name: pandas
  role: Python package installed as a dependency for reading and processing .csv/.xlsx
    gene expression files
- name: dateparser
  role: Python package used to handle the different kinds of date formatting converted
    by Excel
- name: openpyxl
  role: Python package for reading/writing .xlsx input and output files
- name: xlrd
  role: Python package for reading Excel files
- name: XlsxWriter
  role: Python package for writing the output Excel file with updated HUGO gene names
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:28c2a496a632365fb557c7ad38c1f8b930c9caa2bbaffd40f300e618159e8f6d
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# gene-updater-local-streamlit-deployment

## Summary

Deploy the Gene Updater web tool locally with Streamlit to autocorrect Excel date-converted gene names and update old gene symbols to new HGNC-recommended names, using the date_gene_tool.py script from the kuanrongchan/date-to-gene-converter repository.

## When to use

Use when .csv or .xlsx gene expression files contain gene symbols auto-converted to dates by Excel (e.g. SEPT1, MARC1 becoming date strings, or ambiguous Mar-01/Mar-02 terms) and need to be converted back to HGNC-approved gene names.

## When NOT to use

- Input files contain no Excel date-converted gene names and already use current HGNC gene symbols — the tool is designed to rectify date conversions and old names, not general gene expression analysis.
- Ambiguous Mar-01/Mar-02 terms cannot be resolved because no gene description is provided and the raw dataset cannot disambiguate MTARC1/MARCHF1 from MTARC2/MARCHF2 — manual assignment via the widgets would be guesswork with contrasting biological functions at stake.

## Inputs

- .csv or .xlsx gene expression files with gene names in the first column (optional Gene Description in second column)
- date_gene_tool.py source from https://github.com/kuanrongchan/date-to-gene-converter/blob/main/date_gene_tool.py
- demo.csv (pre-loaded demo gene list)
- hgnc-symbol-check2.csv (HGNC symbol conversion file)

## Outputs

- Excel data file containing updated HUGO gene names for downstream analysis
- On-screen inspection of key genes of interest (e.g. MARCHF1) confirming date-converted names were restored

## How to apply

Install Python 3.7 or later, then pip install streamlit plus the dependency packages (pandas, numpy, regex, inflect, dateparser, openpyxl, xlrd, XlsxWriter, streamlit-tags). Create a date_gene_tool folder, copy the date_gene_tool.py source from the GitHub repository, and place demo.csv and hgnc-symbol-check2.csv in the same folder. Launch with 'streamlit run date_gene_tool.py', which opens the app in the default browser; upload .csv or .xlsx files with gene names in the first column (a Gene Description in the second column aids Mar-01/Mar-02 mapping but is optional). Resolve duplicate Mar-01/Mar-02 values — annotated as Mar-01_1st/Mar-01_2nd and Mar-02_1st/Mar-02_2nd — by assigning them to MTARC1/MARCHF1 and MTARC2/MARCHF2 via the provided dropdown widgets, matching gene descriptions where available or checking the raw dataset otherwise. Finally, key in genes of interest (e.g. MARCHF1) to confirm the expression data were updated, then download the output Excel file with updated HUGO gene names.

## Related tools

- **Gene Updater** (Web tool being deployed locally; converts old gene names and Excel date-converted terms back to new HGNC-recommended gene names) — https://github.com/kuanrongchan/date-to-gene-converter
- **Streamlit** (Framework used to build and run the date_gene_tool.py web app locally via 'streamlit run')
- **Python** (Programming language in which date_gene_tool.py is written and executed locally)
- **pandas** (Python package installed as a dependency for reading and processing .csv/.xlsx gene expression files)
- **dateparser** (Python package used to handle the different kinds of date formatting converted by Excel)
- **openpyxl** (Python package for reading/writing .xlsx input and output files)
- **xlrd** (Python package for reading Excel files)
- **XlsxWriter** (Python package for writing the output Excel file with updated HUGO gene names)

## Examples

```
cd path/to/folder; pip install -r requirements.txt; streamlit run date_gene_tool.py
```

## Evaluation signals

- Mar-01 and Mar-02 duplicate values are annotated as Mar-01_1st/Mar-01_2nd (and Mar-02 equivalents) and each is assigned to a chosen gene (MTARC1 or MARCHF1; MTARC2 or MARCHF2) via the widgets.
- Keying in a gene of interest such as MARCHF1 shows the gene expression data updated with the new HGNC gene name.
- Date-converted gene symbols no longer appear as numbers when the gene name column is sorted in ascending order.
- The output Excel file contains updated HUGO gene names and can be downloaded for further downstream analysis.
- The app launches successfully in the default browser tab after 'streamlit run date_gene_tool.py' with demo.csv and hgnc-symbol-check2.csv present in the date_gene_tool folder.

## Limitations

- Mar-01 and Mar-02 can each map to two different gene names (MTARC1/MARCHF1 and MTARC2/MARCHF2), so users must manually assign them; without a Gene Description column they must check the raw dataset.
- MTARC1-2 and MARCHF1-2 have contrasting biological functions, making wrong assignment biologically consequential.
- Python package dependency versions are not pinned ('compatible versions' unspecified in the article).
- Running locally requires Python 3.7+ and manual installation of nine packages plus Streamlit; no changelog is available for the repository.

## Evidence

- [readme] Install Python 3.7 (or later): "Install Python 3.7 (or later) at https://www.python.org/downloads/"
- [readme] streamlit run command: "You can run the program locally by typing in terminal:

streamlit run date_gene_tool.py
"
- [readme] duplicate Mar-01 annotation: [evidence span withheld — anchor status: external_doc]
- [readme] verify gene update: "Finally, users can key in the genes of interest (e.g. MARCHF1) to inspect if the gene expression data has indeed been updated with the new gene names."
- [abstract] Mar-01/Mar-02 ambiguity in article: "as Mar-01 and Mar-02 can each be potentially mapped to 2 different gene names, users can assign the date terms to the appropriate gene names within the Gene Updater web tool."
- [intro] output file description: "The output is an Excel data file containing the updated HUGO gene names which can be downloaded for further downstream analysis."
- [discussion] contrasting biological functions: "it is challenging to interpret whether the genes are either MTARC1-2 or MARCHF1-2, which are known to have contrasting biological functions."
