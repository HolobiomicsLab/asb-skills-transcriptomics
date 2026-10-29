---
name: ambiguous-mar01-mar02-date-term-disambiguation
description: Use when a .csv or .xlsx gene expression file contains date-converted
  entries Mar-01 or Mar-02 (including duplicate Mar-01/Mar-02 values), which cannot
  be auto-converted because they map to two candidate genes each.
when_to_use_negative:
- Input contains old gene names or non-ambiguous date conversions only — these are
  auto-converted by Gene Updater without manual Mar-01/Mar-02 assignment
- The gene list is already in current HGNC nomenclature and contains no Excel date-converted
  terms, so no correction is needed
- No gene description is available AND the raw dataset cannot be checked, making correct
  MTARC/MARCHF assignment impossible
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: Gene Updater
  role: Web tool providing the dropdown/tag widgets used to manually assign Mar-01_1st/Mar-01_2nd
    and Mar-02 values to MTARC1/MARCHF1 or MTARC2/MARCHF2
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Streamlit
  role: Framework hosting the Gene Updater app locally via 'streamlit run date_gene_tool.py'
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Python
  role: Language in which date_gene_tool.py is written and executed (Python 3.7 or
    later required)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:4ecbc04ebd3bb0144b16cb84a9a1b2171627669e0bb93308d6e612eea23a45a8
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# ambiguous-mar01-mar02-date-term-disambiguation

## Summary

Resolves the uniquely ambiguous Excel date-converted terms Mar-01 and Mar-02 — each of which can map to two different HGNC genes (MTARC1/MARCHF1 and MTARC2/MARCHF2) — by manually assigning them to the correct gene name using Gene Updater's widget interface.

## When to use

Use when a .csv or .xlsx gene expression file contains date-converted entries Mar-01 or Mar-02 (including duplicate Mar-01/Mar-02 values), which cannot be auto-converted because they map to two candidate genes each.

## When NOT to use

- Input contains old gene names or non-ambiguous date conversions only — these are auto-converted by Gene Updater without manual Mar-01/Mar-02 assignment
- The gene list is already in current HGNC nomenclature and contains no Excel date-converted terms, so no correction is needed
- No gene description is available AND the raw dataset cannot be checked, making correct MTARC/MARCHF assignment impossible

## Inputs

- .csv or .xlsx gene expression file(s) with gene names in the first column containing Excel date-converted Mar-01/Mar-02 terms
- Optional second-column Gene Description for mapping Mar-01/Mar-02 to gene names
- Raw dataset used to ascertain Mar-01_1st/Mar-01_2nd identity when no gene description is provided

## Outputs

- Gene-name column with Mar-01_1st/Mar-01_2nd and Mar-02 values assigned to MTARC1/MARCHF1 and MTARC2/MARCHF2 respectively
- Downloadable Excel file containing updated HUGO/HGNC gene names for downstream analysis

## How to apply

Run Gene Updater locally with 'streamlit run date_gene_tool.py' (or via the hosted Streamlit instance) after installing Python 3.7+, Streamlit, pandas, numpy, regex, inflect, dateparser, openpyxl, xlrd, XlsxWriter, and streamlit-tags, with demo.csv and hgnc-symbol-check2.csv in the date_gene_tool folder. Upload the .csv/.xlsx file with gene names in the first column; the tool annotates duplicate Mar-01 values as Mar-01_1st and Mar-01_2nd (and likewise for Mar-02). If a Gene Description is present in the second column, match each Mar-01_1st/Mar-01_2nd value to its gene description to assign MTARC1 vs MARCHF1 (and MTARC2 vs MARCHF2 for Mar-02); if not, return to the raw dataset to determine the correct gene before assigning via the dropdown widgets. Finally, key in genes of interest (e.g. MARCHF1) to confirm the expression data were re-labelled under the new HGNC names before downloading the output Excel file.

## Related tools

- **Gene Updater** (Web tool providing the dropdown/tag widgets used to manually assign Mar-01_1st/Mar-01_2nd and Mar-02 values to MTARC1/MARCHF1 or MTARC2/MARCHF2) — https://github.com/kuanrongchan/date-to-gene-converter
- **Streamlit** (Framework hosting the Gene Updater app locally via 'streamlit run date_gene_tool.py') — https://github.com/kuanrongchan/date-to-gene-converter
- **Python** (Language in which date_gene_tool.py is written and executed (Python 3.7 or later required))

## Examples

```
streamlit run date_gene_tool.py
```

## Evaluation signals

- Duplicate Mar-01 values are annotated as Mar-01_1st and Mar-01_2nd before assignment, and none are left unlabelled
- Each assigned Mar-01/Mar-02 term maps to the correct gene (MTARC1/MARCHF1 or MTARC2/MARCHF2) when cross-checked against the gene description column or the raw dataset
- Keying in a gene of interest (e.g. MARCHF1) in the tool confirms the gene expression data have been updated with the new HGNC gene names
- The downloadable output Excel file contains updated HUGO gene names with no residual Mar-01/Mar-02 date terms

## Limitations

- Mar-01 and Mar-02 cannot be auto-converted because each maps to two genes (MTARC1/MARCHF1, MTARC2/MARCHF2), so manual assignment is always required
- MTARC1-2 and MARCHF1-2 have contrasting biological functions, so a wrong assignment changes biological interpretation of the dataset
- If no gene description is provided in the dataset, users must check the raw dataset themselves to ascertain what Mar-01_1st and Mar-01_2nd mean

## Evidence

- [abstract] Mar-01 and Mar-02 can each be potentially mapped to 2 different gene names, and users can assign the date terms to the appropriate gene names within the Gene Updater web tool.: "as Mar-01 and Mar-02 can each be potentially mapped to 2 different gene names, users can assign the date terms"
- [readme] If gene description is provided in the dataset, users will just need to match the gene name to the gene description. Otherwise, users will have to check their raw dataset to ascertain what the Mar-01_1st and Mar-01_2nd mean.: "If gene description is provided in the dataset, users will just need to match the gene name to the gene description"
- [discussion] Genes auto-converted to Mar-01 or Mar-02 are difficult to disambiguate between MTARC1-2 and MARCHF1-2, which have contrasting biological functions: "if the genes are auto-converted to Mar-01 or Mar-02, it is challenging to interpret whether the genes are either MTARC1-2 or MARCHF1-2"
- [readme] When there are duplicate Mar-01 values, Mar-01 will be annotated as Mar-01_1st and Mar-01_2nd. Users will have to manually assign the corresponding gene names to the values using the widgets provided.: "When there are duplicate Mar-01 values, Mar-01 will be annotated as Mar-01_1st and Mar-01_2nd."
