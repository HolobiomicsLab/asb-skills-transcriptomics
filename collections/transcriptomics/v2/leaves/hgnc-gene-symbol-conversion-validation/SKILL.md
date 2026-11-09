---
name: hgnc-gene-symbol-conversion-validation
description: Use when Excel auto-conversion turned gene symbols (e.g. SEPT1, MARC1)
  into dates such as Mar-01/Mar-02 in .csv or .xlsx expression files, or when old
  gene symbols need updating to new HGNC names before pathway analysis or data sharing.
when_to_use_negative:
- The file has never been opened in Excel and already uses current HGNC symbols —
  there are no date-converted terms or old names to repair
- Gene names are not in the first column of the uploaded file, so the tool cannot
  identify which column to convert
- The task is pathway enrichment analysis itself (e.g. Enrichr/GSEA) rather than repairing
  gene symbol labels beforehand
edam_operation: http://edamontology.org/operation_0286
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Gene Updater
  role: Web tool that converts old gene names and Excel date-converted terms back
    to new HGNC-recommended gene names
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Streamlit
  role: Framework hosting the date_gene_tool.py app locally or in the browser for
    interactive Mar-01/Mar-02 assignment
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: dateparser
  role: Python package used locally to recognize the mixed date formats Excel converts
    gene names into
- name: openpyxl
  role: Python package for reading/writing the .xlsx input and output Excel files
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:29ae9b360909457806a9f89b60901d4fb8fe373d936c8dfe921c917ede270790
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
  edam_operation: http://edamontology.org/operation_0286
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# hgnc-gene-symbol-conversion-validation

## Summary

Repairs gene expression files (.csv/.xlsx) whose gene symbols were auto-converted to dates by Excel, or that carry old gene names, by converting them back to current HGNC-recommended symbols with the Gene Updater Streamlit tool — including explicit disambiguation of the ambiguous Mar-01/Mar-02 date terms.

## When to use

Use when Excel auto-conversion turned gene symbols (e.g. SEPT1, MARC1) into dates such as Mar-01/Mar-02 in .csv or .xlsx expression files, or when old gene symbols need updating to new HGNC names before pathway analysis or data sharing.

## When NOT to use

- The file has never been opened in Excel and already uses current HGNC symbols — there are no date-converted terms or old names to repair
- Gene names are not in the first column of the uploaded file, so the tool cannot identify which column to convert
- The task is pathway enrichment analysis itself (e.g. Enrichr/GSEA) rather than repairing gene symbol labels beforehand

## Inputs

- .csv or .xlsx gene expression files with gene names in the first column (one or many files)
- Optional second-column Gene Description to map Mar-01/Mar-02 to MTARC1-2 or MARCHF1-2
- hgnc-symbol-check2.csv (HGNC symbol conversion file from the repository)
- demo.csv (pre-loaded demo gene list for exploring the tool)

## Outputs

- Excel data file with updated HUGO/HGNC gene names, downloadable for downstream analysis
- Explicit user-assigned mappings for ambiguous Mar-01_1st/Mar-01_2nd and Mar-02 terms
- Verification query results confirming genes of interest (e.g. MARCHF1) are updated

## How to apply

Upload one or more .csv or .xlsx files to the Gene Updater web tool (or run it locally with 'streamlit run date_gene_tool.py'), ensuring the first column contains gene names; an optional Gene Description in the second column aids mapping. The tool automatically converts old gene names and date-converted terms to new HGNC names, except Mar-01 and Mar-02, which are ambiguous. When duplicate Mar-01 values occur, they are annotated Mar-01_1st and Mar-01_2nd, and the user assigns each to MTARC1 or MARCHF1 (and Mar-02 to MTARC2 or MARCHF2) via dropdown widgets — matching gene name to gene description if provided, otherwise checking the raw dataset, since MTARC1-2 and MARCHF1-2 have contrasting biological functions. Validate by keying in genes of interest (e.g. MARCHF1) to confirm the expression data were remapped, then download the output Excel file for downstream analysis.

## Related tools

- **Gene Updater** (Web tool that converts old gene names and Excel date-converted terms back to new HGNC-recommended gene names) — https://github.com/kuanrongchan/date-to-gene-converter
- **Streamlit** (Framework hosting the date_gene_tool.py app locally or in the browser for interactive Mar-01/Mar-02 assignment) — https://github.com/kuanrongchan/date-to-gene-converter
- **dateparser** (Python package used locally to recognize the mixed date formats Excel converts gene names into)
- **openpyxl** (Python package for reading/writing the .xlsx input and output Excel files)

## Examples

```
streamlit run date_gene_tool.py
```

## Evaluation signals

- Sort the output gene-name column in ascending order: date-converted symbols appear as numbers at the top, so a correctly repaired file shows none
- Key in a gene of interest (e.g. MARCHF1) in the tool's widget and confirm its expression values were remapped to the new HGNC name
- Duplicate Mar-01 values are annotated Mar-01_1st and Mar-01_2nd and each has an explicit user-assigned gene (MTARC1 or MARCHF1) rather than being left as dates
- All other date terms and old gene names are converted to new HGNC names automatically, consistent with hgnc-symbol-check2.csv
- The output is a downloadable Excel file whose first column contains valid HGNC symbols ready for pathway databases such as Gene Ontology or Reactome

## Limitations

- Mar-01 and Mar-02 cannot be auto-resolved because each maps to two genes (MTARC1-2 vs MARCHF1-2); manual widget assignment is required, and without a Gene Description column the user must check the raw dataset
- Python package versions (pandas, numpy, regex, inflect, dateparser, openpyxl, xlrd, XlsxWriter, streamlit-tags) are not pinned — only 'compatible versions' is specified
- Excel itself offers no automated safeguard; manual cell-formatting date-to-text conversion is tedious and error-prone, motivating use of this tool before publication or data sharing

## Evidence

- [readme] If the first column contains dates, they will be converted to the updated gene names, with the exception of Mar-01 and Mar-02 as these terms can be mapped to more than one gene.: "If the first column contains dates, they will be converted to the updated gene names, with the exception of Mar-01 and Mar-02 as these terms can be mapped to more than one gene."
- [intro] The conversion of Mar-01 to either MTARC1 or MARCHF1, as well as Mar-02 to either MTARC2 or MARCHF2 can be assigned by the user within the Gene Updater web tool.: "The conversion of Mar-01 to either MTARC1 or MARCHF1, as well as Mar-02 to either MTARC2 or MARCHF2 can be assigned by the user within the Gene Updater web tool."
- [discussion] This will generate a new tab with the Gene Updater web tool appearing in the default browser.: "streamlit run date_gene_tool.py. This will generate a new tab with the Gene Updater web tool appearing in the default browser."
- [readme] Finally, users can key in the genes of interest (e.g. MARCHF1) to inspect if the gene expression data has indeed been updated with the new gene names.: "Finally, users can key in the genes of interest (e.g. MARCHF1) to inspect if the gene expression data has indeed been updated with the new gene names."
