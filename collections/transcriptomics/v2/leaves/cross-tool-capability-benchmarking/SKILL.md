---
name: cross-tool-capability-benchmarking
description: 'Use when selecting among gene-name/date correction tools: you need an
  apples-to-apples capability matrix covering multi-file processing, dd/mm/yy vs mixed
  date formats, and HGNC name updating.'
when_to_use_negative:
- Input gene lists contain no Excel date auto-conversions or outdated aliases, so
  tool selection is moot — any tool would be a no-op pass-through.
- You only need to run Gene Updater itself to correct files, not to choose among competing
  tools — use the correction workflow directly instead of benchmarking.
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Gene Updater
  role: 'Tool under test: Streamlit web app converting dates and old gene names to
    new HGNC-recommended names'
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Streamlit
  role: Framework for launching Gene Updater locally as date_gene_tool.py during benchmarking
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Python
  role: Runtime environment (Python 3.7 or later) in which Gene Updater and the benchmark
    harness are executed
- name: Oct4th
  role: Competing tool run on the identical corpus as a comparison baseline
- name: Truke
  role: Competing tool run on the identical corpus; known limited to dd/mm/yy dates
    and one file at a time
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:8c688d4007892145576bd81e8fbea51353917f68f7adbc3dc6628826e11b1280
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/date-to-gene-converter
  tools:
  - Gene Updater
  - Oct4th
  - Python
  - Streamlit
  - Truke
  - XlsxWriter
  - dateparser
  - inflect
  - numpy
  - openpyxl
  - pandas
  - regex
  - xlrd
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# cross-tool-capability-benchmarking

## Summary

Benchmarking method for comparing gene-name/date correction tools (e.g., Gene Updater, Oct4th, Truke) on an identical corpus of Excel-corrupted .csv/.xlsx files, scoring multi-file support, mixed date-format handling, and conversion to updated HGNC names.

## When to use

Use when selecting among gene-name/date correction tools: you need an apples-to-apples capability matrix covering multi-file processing, dd/mm/yy vs mixed date formats, and HGNC name updating.

## When NOT to use

- Input gene lists contain no Excel date auto-conversions or outdated aliases, so tool selection is moot — any tool would be a no-op pass-through.
- You only need to run Gene Updater itself to correct files, not to choose among competing tools — use the correction workflow directly instead of benchmarking.

## Inputs

- Corpus of .csv and .xlsx files with Excel date-converted gene names (mixed date formats)
- Files containing duplicate Mar-01/Mar-02 terms (annotated as Mar-01_1st, Mar-01_2nd)
- demo.csv (pre-loaded restricted gene list from the date-to-gene-converter repo)
- hgnc-symbol-check2.csv (HGNC symbol conversion reference)

## Outputs

- tool_comparison.csv with per-tool capability flags (multi-file, per-date-format handling, HGNC-updated output)
- Corrected gene expression files with updated HGNC gene names from each tool for side-by-side verification

## How to apply

Assemble an identical test corpus of .csv/.xlsx files containing mixed Excel date-converted gene names and duplicate Mar-01/Mar-02 terms, using the paper's demo.csv and hgnc-symbol-check2.csv as reference inputs with gene names in the first column. Run each tool under the same date-format and duplicate-term conditions: Gene Updater locally via 'streamlit run date_gene_tool.py' (after installing Python 3.7+, Streamlit, pandas, numpy, regex, inflect, dateparser, openpyxl, xlrd, XlsxWriter, streamlit-tags), plus each competing tool (Oct4th, Truke). For each tool, record three capability flags: whether it processes multiple files, whether it handles every date format present in the corpus, and whether it converts to updated HGNC-recommended gene names. Resolve the Mar-01/Mar-02 ambiguity step by mapping date terms to MTARC1/MARCHF1 and MTARC2/MARCHF2 via the tool's widgets, matching gene names to gene descriptions where provided and otherwise checking the raw dataset to assign Mar-01_1st/Mar-01_2nd values. Write the capability flags to tool_comparison.csv so the comparative finding (Gene Updater handles multiple files and mixed date formats; Truke is limited to dd/mm/yy dates and one file at a time) is machine-checkable.

## Related tools

- **Gene Updater** (Tool under test: Streamlit web app converting dates and old gene names to new HGNC-recommended names) — https://github.com/kuanrongchan/date-to-gene-converter
- **Streamlit** (Framework for launching Gene Updater locally as date_gene_tool.py during benchmarking) — https://github.com/kuanrongchan/date-to-gene-converter
- **Python** (Runtime environment (Python 3.7 or later) in which Gene Updater and the benchmark harness are executed)
- **Oct4th** (Competing tool run on the identical corpus as a comparison baseline)
- **Truke** (Competing tool run on the identical corpus; known limited to dd/mm/yy dates and one file at a time)

## Examples

```
streamlit run date_gene_tool.py
```

## Evaluation signals

- tool_comparison.csv contains one row per tool with three boolean capability flags (multi-file, date-format coverage, HGNC update) filled for every tool
- Corrected output keyed on genes of interest (e.g., MARCHF1) confirms gene expression data updated with new HGNC gene names
- Duplicate Mar-01 values are annotated as Mar-01_1st and Mar-01_2nd and mapped to the correct gene names (MTARC1 vs MARCHF1) via the widgets
- All three tools were run on the identical corpus under identical date-format and duplicate-term conditions
- Benchmark outcome should reproduce the published comparative result: Gene Updater processes multiple .csv/.xlsx files and multiple date kinds, while Truke handles only dd/mm/yy and single files

## Limitations

- Python package dependency versions are not pinned ('compatible versions' unspecified), so local reproductions of Gene Updater may drift across environments
- Truke is limited to dd/mm/yy dates and one file at a time, restricting the benchmark's ability to compare rich date-format handling across all tools
- Mar-01/Mar-02 disambiguation between MTARC1-2 and MARCHF1-2 (which have contrasting biological functions) requires manual widget assignment or a gene description column; without it, the benchmark's mapping step is error-prone
- Approximately 50-100 research articles still have gene name errors reported monthly, so benchmarks against a fixed corpus may not generalize to all date formats seen in the wild

## Evidence

- [readme] Users can upload their .csv or .xlsx file or files. Ensure that the first column contains the gene names.: "Users can upload their .csv or .xlsx file or files. Ensure that the first column contains the gene names."
- [readme] When there are duplicate Mar-01 values, Mar-01 will be annotated as Mar-01_1st and Mar-01_2nd.: [evidence span withheld — anchor status: external_doc]
- [intro] The conversion of Mar-01 to either MTARC1 or MARCHF1, as well as Mar-02 to either MTARC2 or MARCHF2 can be assigned by the user.: "The conversion of Mar-01 to either MTARC1 or MARCHF1, as well as Mar-02 to either MTARC2 or MARCHF2 can be assigned by the user within the Gene Updater web tool."
- [abstract] This user-friendly web tool ensures that the accuracy and integrity of gene expression data is preserved by minimizing errors.: "This user-friendly web tool ensures that the accuracy and integrity of gene expression data is preserved by minimizing errors in labelling gene names due to Excel auto-conversions."
- [readme] Finally, users can key in the genes of interest (e.g. MARCHF1) to inspect if the gene expression data has indeed been updated.: "Finally, users can key in the genes of interest (e.g. MARCHF1) to inspect if the gene expression data has indeed been updated with the new gene names."
