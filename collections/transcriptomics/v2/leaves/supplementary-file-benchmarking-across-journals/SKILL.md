---
name: supplementary-file-benchmarking-across-journals
description: Use when you need to benchmark a corpus of published supplementary files
  (e.g. from Mark Ziemann's gene_name_errors dashboard) for a recurring data-integrity
  defect, and report screened/affected/autocorrected counts per journal and in total.
when_to_use_negative:
- You are working with a single dataset you control and simply need to fix gene names
  — use Gene Updater directly rather than the journal-wide benchmarking workflow
- The supplementary files contain no gene symbols or gene-name columns, so the HGNC
  symbol screening and date-error quantification do not apply
- The analysis goal is pathway-level impact of date conversion (GO Biological Processes
  enrichment via Enrichr) rather than file-level prevalence statistics
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Gene Updater
  role: Applied to flagged supplementary files to determine which date-converted gene
    names (e.g. Oct-04, Mar-01) are autocorrected to new HGNC-approved symbols; failures
    are recorded for the benchmark table
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Streamlit
  role: Hosts/runs the Gene Updater web tool locally via 'streamlit run date_gene_tool.py'
    during the autocorrection step of the benchmark
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: pandas
  role: Aggregates per-journal and total counts (files screened, with gene symbols,
    with date errors, autocorrected fraction) into the summary table; recommended
    version pandas >= 1.2.5
- name: Mark Ziemann's gene name errors dashboard
  role: 'Source of the corpus: extracts supplementary text and Excel files from various
    journals (e.g. Report_2022-05 page)'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:580e7efa829a8cdc5bdf3df9a71b7f854d7046b924dd274238b56c109dd753a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/date-to-gene-converter
  tools:
  - Gene Updater
  - Mark Ziemann's gene name errors dashboard
  - Python
  - Streamlit
  - pandas
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# supplementary-file-benchmarking-across-journals

## Summary

A benchmarking skill that reproduces the Gene Updater paper's journal-level audit of supplementary text/Excel gene-list files, quantifying what proportion contain Excel date-converted gene name errors (e.g. Mar-01, Mar-02) and what fraction the Gene Updater tool can autocorrect.

## When to use

Use when you need to benchmark a corpus of published supplementary files (e.g. from Mark Ziemann's gene_name_errors dashboard) for a recurring data-integrity defect, and report screened/affected/autocorrected counts per journal and in total.

## When NOT to use

- You are working with a single dataset you control and simply need to fix gene names — use Gene Updater directly rather than the journal-wide benchmarking workflow
- The supplementary files contain no gene symbols or gene-name columns, so the HGNC symbol screening and date-error quantification do not apply
- The analysis goal is pathway-level impact of date conversion (GO Biological Processes enrichment via Enrichr) rather than file-level prevalence statistics

## Inputs

- List of supplementary text/Excel files published in major journals, extracted from Mark Ziemann's gene_name_errors dashboard (e.g. Report_2022-05 page)
- Downloaded supplementary .txt and .xlsx files containing gene expression gene lists
- hgnc-symbol-check2.csv (HGNC symbol check list) from the date-to-gene-converter repository

## Outputs

- Per-journal and total summary table (files screened, files with gene symbols, files with date-related errors and resulting percentage, fraction autocorrected by Gene Updater) built with pandas
- List of flagged files containing date-converted gene symbols (Mar-01, Mar-02)
- Record of which date-converted gene names Gene Updater autocorrected vs. left unresolved

## How to apply

First query Mark Ziemann's gene_name_errors dashboard (e.g. the June 2022 report page) to extract the list of supplementary text and Excel files published in major journals. Download each listed file and screen it for gene symbols using the HGNC symbol check list (hgnc-symbol-check2.csv). Sort each gene-name column in ascending order so that date-converted symbols (Mar-01, Mar-02) surface as numbers at the top of the column, flagging files with date-related gene name errors. Run the Gene Updater tool locally (pip install streamlit; streamlit run date_gene_tool.py) on flagged files and record which date-converted names are autocorrected to new HGNC-approved symbols; note that Mar-01/Mar-02 remain ambiguous (mapping to MTARC1/2 vs MARCHF1/2) unless a gene description is supplied. Aggregate counts per journal and in total with pandas into a summary table (files screened, files with gene symbols, files with date errors and percentage, fraction autocorrected).

## Related tools

- **Gene Updater** (Applied to flagged supplementary files to determine which date-converted gene names (e.g. Oct-04, Mar-01) are autocorrected to new HGNC-approved symbols; failures are recorded for the benchmark table) — https://github.com/kuanrongchan/date-to-gene-converter
- **Streamlit** (Hosts/runs the Gene Updater web tool locally via 'streamlit run date_gene_tool.py' during the autocorrection step of the benchmark) — https://github.com/kuanrongchan/date-to-gene-converter
- **pandas** (Aggregates per-journal and total counts (files screened, with gene symbols, with date errors, autocorrected fraction) into the summary table; recommended version pandas >= 1.2.5)
- **Mark Ziemann's gene name errors dashboard** (Source of the corpus: extracts supplementary text and Excel files from various journals (e.g. Report_2022-05 page))

## Examples

```
pip install streamlit; streamlit run date_gene_tool.py
```

## Evaluation signals

- Reproduced table matches the reported benchmark figures: 356 files found, 81 with gene symbols, 28 (34.6%) with date-related errors, and 78.6% autocorrected by Gene Updater
- Gene-name columns sorted in ascending order show date-converted symbols appearing as numbers at the top of the column, confirming correct flagging
- Only files containing Mar-01 and Mar-02 terms without an accompanying unique identifier (e.g. gene description) remain unresolved — any other unresolved file signals a screening error
- Percentage of files with gene symbols containing date errors is comparable to the paper's ~one-fifth baseline for published supplementary Excel gene lists
- Per-journal counts sum to the totals reported across all screened files

## Limitations

- Mar-01 and Mar-02 are the only failures without a unique identifier such as a gene description, since each can map to two genes (MTARC1/MARCHF1 and MTARC2/MARCHF2) with contrasting biological functions
- Benchmark figures are time-stamped to a specific dashboard report (June 2022); ~50–100 articles with gene name errors were still reported monthly, so counts drift over time
- Python package dependency versions are not pinned ('compatible versions' unspecified for pandas, numpy, regex, inflect, dateparser, xlrd, openpyxl, XlsxWriter, streamlit-tags)
- When there are duplicate Mar-01 values in a file they are annotated Mar-01_1st and Mar-01_2nd and require manual assignment via the tool's widgets, complicating fully automated benchmarking

## Evidence

- [discussion] Extract file list from dashboard report and screen for gene symbols using HGNC symbol check: [evidence span withheld — anchor status: not_found]
- [discussion] Sort ascending to spot date-converted symbols: "The fastest way to spot these errors is by sorting the column of gene names in ascending order, and the gene symbols that are converted to dates will appear as numbers at the top of the column."
- [discussion] Run the tool locally for the autocorrection benchmark step: "users can then simply type: streamlit run date_gene_tool.py. This will generate a new tab with the Gene Updater web tool appearing in the default browser."
- [discussion] Mar-01/Mar-02 ambiguity is the key failure mode in benchmarking: "if the genes are auto-converted to Mar-01 or Mar-02, it is challenging to interpret whether the genes are either MTARC1-2 or MARCHF1-2, which are known to have contrasting biological functions."
- [readme] README: gene description in second column aids mapping Mar-01/Mar-02, informing which files autocorrect: "Having a Gene Description on the second column is useful for mapping the Mar-01 and Mar-02 genes, although this step is not mandatory."
