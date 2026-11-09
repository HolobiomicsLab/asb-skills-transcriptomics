---
name: excel-date-conversion-detection
description: Use when supplementary .csv or .xlsx gene-expression files contain date-like
  tokens (Mar-01, Mar-02, or numeric values at the top of an ascending-sorted gene-name
  column) that must be mapped back to HGNC symbols.
when_to_use_negative:
- The input file contains no gene symbols in the first column (e.g. plain metadata
  or image files), so there is nothing to date-screen
- Gene names are already current HGNC symbols with no date-like tokens present — conversion
  would add no value
- The task requires updating stale-but-valid aliases (not Excel date errors) without
  dates involved; still useable but the date-detection screen is unnecessary
edam_topics:
- http://edamontology.org/topic_0202
tools:
- name: Gene Updater
  role: Web/local tool that autocorrects Excel date-converted gene names and old gene
    names to new HGNC-recommended symbols
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Streamlit
  role: Framework used to run the Gene Updater web tool locally via 'streamlit run
    date_gene_tool.py'
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: pandas >= 1.2.5
  role: Recommended Python dependency for reading files and aggregating per-journal
    error counts into a summary table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:1c1434c1f3db56fac0b96cc1172b73c99b3720c9d967b2e1c5ebfa2bab68da51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/date-to-gene-converter
  tools:
  - Gene Updater
  - Python
  - Streamlit
  - pandas
  - pandas >= 1.2.5
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0202
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# excel-date-conversion-detection

## Summary

Detects gene symbols inadvertently auto-converted to dates when .csv/.xlsx gene expression files are opened in Excel (e.g. SEPT1 → Sep-01, MARC1 → Mar-01), and converts the date-like terms and old names back to current HGNC-approved gene symbols using the Gene Updater tool.

## When to use

Use when supplementary .csv or .xlsx gene-expression files contain date-like tokens (Mar-01, Mar-02, or numeric values at the top of an ascending-sorted gene-name column) that must be mapped back to HGNC symbols.

## When NOT to use

- The input file contains no gene symbols in the first column (e.g. plain metadata or image files), so there is nothing to date-screen
- Gene names are already current HGNC symbols with no date-like tokens present — conversion would add no value
- The task requires updating stale-but-valid aliases (not Excel date errors) without dates involved; still useable but the date-detection screen is unnecessary

## Inputs

- .csv or .xlsx supplementary gene-expression files with gene names in the first column
- hgnc-symbol-check2.csv (HGNC symbol check list)
- demo.csv (pre-loaded demo gene list for exploring the tool)

## Outputs

- Excel data file containing updated HGNC-recommended gene names for downstream analysis
- List of files containing date-related gene name errors (flagged by ascending-sort screening)
- Counts of files screened, files with gene symbols, files with date errors, and fraction autocorrected (aggregatable in pandas)

## How to apply

Screen each supplementary text/Excel file for gene symbols against the HGNC symbol check list (hgnc-symbol-check2.csv), then sort the gene-name column in ascending order — date-converted symbols appear as numbers at the top of the column, which is the fastest way to flag affected files. Run Gene Updater (pip install streamlit; streamlit run date_gene_tool.py) with demo.csv and hgnc-symbol-check2.csv in the date_gene_tool folder; the tool uploads .csv/.xlsx files with gene names in the first column and converts dates to updated HGNC gene names. Mar-01 and Mar-02 are ambiguous (each maps to two genes: MTARC1/MARCHF1 and MTARC2/MARCHF2), so assign them via dropdown widgets; a Gene Description in the second column helps disambiguation, and duplicate Mar-01 values are annotated Mar-01_1st and Mar-01_2nd for manual assignment. Finally key in genes of interest (e.g. MARCHF1) to confirm the output Excel file contains the updated HUGO gene names for downstream pathway analysis with tools such as Enrichr, GSEA, or Ingenuity Pathway Analysis.

## Related tools

- **Gene Updater** (Web/local tool that autocorrects Excel date-converted gene names and old gene names to new HGNC-recommended symbols) — https://github.com/kuanrongchan/date-to-gene-converter
- **Streamlit** (Framework used to run the Gene Updater web tool locally via 'streamlit run date_gene_tool.py') — https://github.com/kuanrongchan/date-to-gene-converter
- **pandas >= 1.2.5** (Recommended Python dependency for reading files and aggregating per-journal error counts into a summary table)

## Examples

```
streamlit run date_gene_tool.py
```

## Evaluation signals

- Benchmark reproduction check: June 2022 dashboard files yield 356 files found, 81 with gene symbols, 28 (34.6%) with date-related errors, and 78.6% autocorrected by Gene Updater
- Ascending-sort invariant: in a correctly flagged file, date-converted gene symbols appear as numbers at the top of the sorted gene-name column
- Known-failure check: files whose only date errors are Mar-01/Mar-02 without an accompanying unique identifier (e.g. gene description) are exactly the cases Gene Updater fails to autocorrect
- Duplicate-handling check: duplicated Mar-01 values must appear as Mar-01_1st and Mar-01_2nd pending manual gene-name assignment via widgets
- Output validation: key in a gene of interest (e.g. MARCHF1) and confirm the output Excel contains its updated HGNC name

## Limitations

- Mar-01 and Mar-02 are ambiguous — they map to MTARC1/MTARC2 versus MARCHF1/MARCHF2, which have contrasting biological functions, and disambiguation requires a gene description or manual inspection of the raw dataset
- Users with duplicate Mar-01/Mar-02 values but no gene description must manually check their raw dataset to assign Mar-01_1st / Mar-01_2nd
- Python package versions are not pinned ('compatible versions' unspecified for pandas, numpy, regex, inflect, dateparser, xlrd, openpyxl, XlsxWriter, streamlit-tags)
- Manual date-to-text conversion inside Excel itself is tedious and error-prone, so Excel offers no automated safeguard

## Evidence

- [intro] Excel converts gene names to dates when opened with default settings: "When gene expression datasets are opened with Excel under default settings (Microsoft Corp., Redmond, WA), a recurring problem where gene names are converted to dates occurs."
- [discussion] Fastest error spotting method is ascending sort: "The fastest way to spot these errors is by sorting the column of gene names in ascending order, and the gene symbols that are converted to dates will appear as numbers at the top of the column."
- [abstract] Tool converts dates and old names to HGNC symbols via Streamlit: "Herein, we developed a web tool with Streamlit that can convert old gene names and dates back into the new gene names recommended by HGNC"
- [readme] Mar-01 and Mar-02 cannot be auto-converted without extra identifiers: "If the first column contains dates, they will be converted to the updated gene names, with the exception of Mar-01 and Mar-02 as these terms can be mapped to more than one gene."
- [readme] Duplicate Mar-01 values are annotated for manual assignment: [evidence span withheld — anchor status: external_doc]
- [discussion] Mar-01/Mar-02 ambiguity affects MTARC and MARCHF genes with contrasting functions: "if the genes are auto-converted to Mar-01 or Mar-02, it is challenging to interpret whether the genes are either MTARC1-2 or MARCHF1-2, which are known to have contrasting biological functions."
