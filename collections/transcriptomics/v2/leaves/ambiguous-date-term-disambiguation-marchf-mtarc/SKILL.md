---
name: ambiguous-date-term-disambiguation-marchf-mtarc
description: Use when an Excel-processed gene list contains date-converted symbols
  Mar-01 or Mar-02 that cannot be uniquely mapped to a single HGNC gene, and MTARC
  vs MARCHF assignment must be resolved from context.
when_to_use_negative:
- The date-converted term is not Mar-01 or Mar-02 (e.g. Dec-01, Sep-01..Sep-15) —
  these have unique HGNC mappings and are auto-corrected without user assignment.
- The input file has already been processed with Gene Updater and contains only HGNC-approved
  symbols.
- The dataset contains no Excel auto-conversion errors, only old-but-valid gene names
  needing HGNC updates.
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_0659
tools:
- name: Gene Updater
  role: Streamlit web tool that flags duplicate Mar-01/Mar-02 values (Mar-01_1st,
    Mar-01_2nd) and provides dropdown widgets for user assignment to MTARC/MARCHF
    genes
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Streamlit
  role: Framework hosting the Gene Updater app and its disambiguation dropdown widgets
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: pandas >= 1.2.5
  role: Loads the gene list and hgnc-symbol-check2.csv used to build the date-term-to-HGNC
    mapping
- name: Microsoft Excel
  role: Source of the date auto-conversion that produced the ambiguous Mar-01/Mar-02
    terms; also manual date-to-text fallback (tedious and error-prone)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:31b4acb61f456dfaaade5962cfbd280a197700cf5825170f2085b05d44a5feb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/date-to-gene-converter
  tools:
  - Gene Updater
  - Microsoft Excel
  - Python
  - Streamlit
  - pandas
  - pandas >= 1.2.5
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# ambiguous-date-term-disambiguation-marchf-mtarc

## Summary

Resolves the two Excel date-converted gene terms (Mar-01, Mar-02) that each map ambiguously to two HGNC genes (MTARC1/MARCHF1 and MTARC2/MARCHF2), using user-assigned dropdown widgets in the Gene Updater Streamlit app and gene-description matching.

## When to use

Use when an Excel-processed gene list contains date-converted symbols Mar-01 or Mar-02 that cannot be uniquely mapped to a single HGNC gene, and MTARC vs MARCHF assignment must be resolved from context.

## When NOT to use

- The date-converted term is not Mar-01 or Mar-02 (e.g. Dec-01, Sep-01..Sep-15) — these have unique HGNC mappings and are auto-corrected without user assignment.
- The input file has already been processed with Gene Updater and contains only HGNC-approved symbols.
- The dataset contains no Excel auto-conversion errors, only old-but-valid gene names needing HGNC updates.

## Inputs

- .csv or .xlsx gene expression file with gene names in the first column containing Mar-01 and/or Mar-02 date-converted terms
- hgnc-symbol-check2.csv (previous-name to new-HGNC-symbol correspondences from the date-to-gene-converter repo)
- optional second-column Gene Description for disambiguating Mar-01_1st / Mar-01_2nd duplicates

## Outputs

- Excel output file with updated HUGO gene names where each Mar-01/Mar-02 instance is assigned to one of MTARC1/MARCHF1 or MTARC2/MARCHF2
- Verified mapping record (e.g. date_to_gene_mapping.csv) with both candidate assignments noted for Mar-01 and Mar-02

## How to apply

First confirm the term is one of the two ambiguous date terms; all other date-converted terms (Dec-01, Mar-03..Mar-11, Sep-01..Sep-15) map one-to-one to HGNC symbols and need no disambiguation. In Gene Updater, upload the .csv/.xlsx file with gene names in the first column; when duplicate Mar-01 values exist they are annotated as Mar-01_1st and Mar-01_2nd, and the user assigns each to either MTARC1/MARCHF1 (and Mar-02 to MTARC2/MARCHF2) via the Streamlit dropdown widgets. Providing a Gene Description in the second column is recommended because users can match the gene name to its description; without it, the raw dataset must be checked manually. The rationale is that MTARC1-2 and MARCHF1-2 have contrasting biological functions, so a wrong assignment invalidates downstream pathway enrichment (e.g. Enrichr against GO Biological Processes). After assignment, verify by keying in genes of interest (e.g. MARCHF1) in the app to confirm the expression data updated to the new gene names before downloading the output Excel file.

## Related tools

- **Gene Updater** (Streamlit web tool that flags duplicate Mar-01/Mar-02 values (Mar-01_1st, Mar-01_2nd) and provides dropdown widgets for user assignment to MTARC/MARCHF genes) — https://github.com/kuanrongchan/date-to-gene-converter
- **Streamlit** (Framework hosting the Gene Updater app and its disambiguation dropdown widgets) — https://github.com/kuanrongchan/date-to-gene-converter
- **pandas >= 1.2.5** (Loads the gene list and hgnc-symbol-check2.csv used to build the date-term-to-HGNC mapping)
- **Microsoft Excel** (Source of the date auto-conversion that produced the ambiguous Mar-01/Mar-02 terms; also manual date-to-text fallback (tedious and error-prone))

## Examples

```
streamlit run date_gene_tool.py
```

## Evaluation signals

- Every non-ambiguous date term in the mapping has a unique HGNC target; only Mar-01 and Mar-02 retain two candidate genes each (MTARC1/MARCHF1, MTARC2/MARCHF2).
- Duplicate Mar-01 values are correctly annotated as Mar-01_1st and Mar-01_2nd and each assigned exactly one gene.
- Keying in a gene of interest (e.g. MARCHF1) in the app shows the gene expression data updated to the new gene name.
- The full mapping table contains all 41 previous-name/Excel-date/HUGO-gene rows of Table 1, with every mapped symbol matching HGNC-approved names in hgnc-symbol-check2.csv.

## Limitations

- Mar-01 and Mar-02 cannot be auto-resolved algorithmically; without a Gene Description column or raw-dataset context the assignment is a manual user decision.
- MTARC1-2 and MARCHF1-2 have contrasting biological functions, so mis-assignment propagates to downstream pathway interpretation (e.g. Enrichr GO Biological Processes).
- Package dependency versions are not pinned ('compatible versions' unspecified), which may complicate local reproduction of the disambiguation widgets.

## Evidence

- [abstract] Ambiguity is user-resolved in app: "as Mar-01 and Mar-02 can each be potentially mapped to 2 different gene names, users can assign the date terms to the appropriate gene names within the Gene Updater web tool."
- [readme] Duplicate annotation and widget assignment: [evidence span withheld — anchor status: external_doc]
- [readme] Gene description aids matching: "Having a Gene Description on the second column is useful for mapping the Mar-01 and Mar-02 genes, although this step is not mandatory."
- [discussion] Biological stakes of mis-assignment: "if the genes are auto-converted to Mar-01 or Mar-02, it is challenging to interpret whether the genes are either MTARC1-2 or MARCHF1-2, which are known to have contrasting biological functions."
- [intro] Verification step: "The conversion of Mar-01 to either MTARC1 or MARCHF1, as well as Mar-02 to either MTARC2 or MARCHF2 can be assigned by the user within the Gene Updater web tool."
