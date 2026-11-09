---
name: journal-level-error-count-tabulation
description: Use when quantifying the prevalence of Excel date-converted gene symbols
  across published supplementary files, or benchmarking what fraction of date-related
  gene name errors a correction tool can resolve per journal.
when_to_use_negative:
- Input is a single gene expression file that itself needs old gene names or dates
  corrected — that is the Gene Updater correction skill, not the journal-level prevalence
  tabulation
- Goal is pathway enrichment analysis of corrected gene lists (e.g. against GO Biological
  Processes with Enrichr) rather than error-prevalence statistics
- Supplementary files contain no gene symbols in the first column, so they cannot
  be screened for date-converted gene names
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Gene Updater
  role: Runs on flagged files to test which date-converted gene names are autocorrected
    to HGNC-approved symbols; the autocorrection fraction is a key tabulated metric
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Streamlit
  role: Framework used to run Gene Updater locally during the screening round (streamlit
    run date_gene_tool.py)
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Python + pandas
  role: Aggregates per-journal and total counts (files screened, with gene symbols,
    with date errors, percent autocorrected) into the summary table
- name: Mark Ziemann's gene name errors dashboard
  role: Source of the journal-level list of published supplementary text and Excel
    files to download and screen
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:31a1ef53e9af8a00ee682e9b89432c2af0274983bb9c4792d8370e52bddf2446
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/date-to-gene-converter
  tools:
  - Gene Updater
  - Mark Ziemann's gene name errors dashboard
  - Python
  - Python + pandas
  - Streamlit
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# journal-level-error-count-tabulation

## Summary

A benchmarking workflow that tabulates, per journal and in total, how many supplementary text/Excel files contain gene symbols and Excel date-conversion errors, and what fraction Gene Updater can autocorrect — reproducing the paper's Table 2 statistics.

## When to use

Use when quantifying the prevalence of Excel date-converted gene symbols across published supplementary files, or benchmarking what fraction of date-related gene name errors a correction tool can resolve per journal.

## When NOT to use

- Input is a single gene expression file that itself needs old gene names or dates corrected — that is the Gene Updater correction skill, not the journal-level prevalence tabulation
- Goal is pathway enrichment analysis of corrected gene lists (e.g. against GO Biological Processes with Enrichr) rather than error-prevalence statistics
- Supplementary files contain no gene symbols in the first column, so they cannot be screened for date-converted gene names

## Inputs

- List of supplementary text/Excel files from Mark Ziemann's gene_name_errors dashboard (journal-level report page)
- Downloaded supplementary .txt and .xlsx files from major journals
- HGNC symbol check list (hgnc-symbol-check2.csv)

## Outputs

- Journal-level summary table (files screened, files with gene symbols, files with date errors, error percentage, fraction autocorrected by Gene Updater)
- Per-file flags for date-related gene name errors (e.g. Mar-01, Mar-02 appearing as numbers at top of a sorted gene-name column)

## How to apply

Start from Mark Ziemann's gene_name_errors dashboard (e.g. the June 2022 report page) to extract the list of supplementary text and Excel files published in major journals, then download and screen each file for gene symbols using the HGNC symbol check list (hgnc-symbol-check2.csv). Sort each gene-name column in ascending order so date-converted symbols (Mar-01, Mar-02) surface as numbers at the top, flagging those files as date-error cases. Run Gene Updater locally (pip install streamlit; streamlit run date_gene_tool.py) on the flagged files and record which date-converted names are autocorrected to HGNC-approved symbols. Aggregate counts per journal and overall (files screened, files with gene symbols, files with date errors and the resulting percentage, and the fraction autocorrected) into a summary table with pandas. Validate against the reported benchmark: 356 files found, 81 with gene symbols, 28 (34.6%) with date-related errors, and 78.6% autocorrected.

## Related tools

- **Gene Updater** (Runs on flagged files to test which date-converted gene names are autocorrected to HGNC-approved symbols; the autocorrection fraction is a key tabulated metric) — https://github.com/kuanrongchan/date-to-gene-converter
- **Streamlit** (Framework used to run Gene Updater locally during the screening round (streamlit run date_gene_tool.py)) — https://github.com/kuanrongchan/date-to-gene-converter
- **Python + pandas** (Aggregates per-journal and total counts (files screened, with gene symbols, with date errors, percent autocorrected) into the summary table)
- **Mark Ziemann's gene name errors dashboard** (Source of the journal-level list of published supplementary text and Excel files to download and screen)

## Examples

```
pip install streamlit; streamlit run date_gene_tool.py
```

## Evaluation signals

- Reproduced table matches the reported benchmark figures: 356 files found, 81 with gene symbols, 28 (34.6%) with date-related errors, and 78.6% autocorrected
- All Gene Updater failures in the tabulation are attributable to files containing Mar-01 and Mar-02 terms without an accompanying unique identifier such as a gene description
- Date-converted gene symbols appear as numbers at the top of gene-name columns sorted in ascending order, confirming the flagging rule
- Counts per journal sum consistently to the totals (files screened ≥ files with gene symbols ≥ files with date errors ≥ autocorrected fraction)

## Limitations

- Mar-01 and Mar-02 can each map to two genes (MTARC1/MARCHF1 and MTARC2/MARCHF2, which have contrasting biological functions), so files with these terms and no gene description cannot be autocorrected automatically and require manual user assignment
- Analysis depends on the currency of Mark Ziemann's dashboard report pages (e.g. the 2022-05 report), and approximately 50–100 research articles per month still carry gene name errors, so tabulations are snapshots in time

## Evidence

- [intro] Leveraging Ziemann dashboard as file source: [evidence span withheld — anchor status: not_found]
- [discussion] Ascending-sort flagging rule: "The fastest way to spot these errors is by sorting the column of gene names in ascending order, and the gene symbols that are converted to dates will appear as numbers at the top of the column."
- [abstract] Mar-01/Mar-02 ambiguity limits autocorrection: "as Mar-01 and Mar-02 can each be potentially mapped to 2 different gene names, users can assign the date terms to the appropriate gene names within the Gene Updater web tool."
- [readme] Local execution route for screening runs: "Users can upload their .csv or .xlsx file or files. Ensure that the first column contains the gene names. Having a Gene Description on the second column is useful for mapping the Mar-01 and Mar-02"
