---
name: enrichment-p-value-multiple-testing-interpretation
description: Use when you have a gene list submitted to an enrichment tool (e.g. Enrichr
  against GO Biological Processes) and must decide which returned terms are significant,
  using adjusted p-value < 0.05 rather than raw p-values.
when_to_use_negative:
- The gene list still contains Excel date terms (Mar-01, Mar-02) or old gene symbols
  that have not been mapped to HGNC names — pathway databases will not recognize them
  and enrichment results will be void
- You need rank-based or set-level statistics (e.g. GSEA permutation testing) rather
  than over-representation adjusted p-values from Enrichr
- Raw unadjusted p-values are the quantity of interest for a bespoke sensitivity analysis
  — this skill deliberately discards them in favor of corrected values
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Enrichr
  role: Web tool that performs over-representation analysis of the gene list against
    the GO Biological Processes database and returns adjusted p-values used for term
    selection
- name: Gene Updater
  role: Pre-processing step that restores date-converted and old gene names to HGNC-approved
    symbols so the enrichment query list is complete
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: Gene Ontology (GO) Biological Processes database
  role: Reference pathway database against which the gene terms are queried
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:c79afa186969f193693ca9d9adb6d6404278ee83c660fcbd907162367fa5db21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/date-to-gene-converter
  tools:
  - Enrichr
  - Gene Ontology (GO) Biological Processes database
  - Gene Updater
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Enrichment p-value multiple testing interpretation

## Summary

Interpret over-representation (pathway enrichment) results from tools like Enrichr by ranking terms on the multiple-testing-corrected p-value and retaining only terms with adjusted p < 0.05. In the Gene Updater study this defined which GO Biological Processes were deemed most affected by Excel date-conversion of gene names.

## When to use

Use when you have a gene list submitted to an enrichment tool (e.g. Enrichr against GO Biological Processes) and must decide which returned terms are significant, using adjusted p-value < 0.05 rather than raw p-values.

## When NOT to use

- The gene list still contains Excel date terms (Mar-01, Mar-02) or old gene symbols that have not been mapped to HGNC names — pathway databases will not recognize them and enrichment results will be void
- You need rank-based or set-level statistics (e.g. GSEA permutation testing) rather than over-representation adjusted p-values from Enrichr
- Raw unadjusted p-values are the quantity of interest for a bespoke sensitivity analysis — this skill deliberately discards them in favor of corrected values

## Inputs

- Gene list (e.g. Table 1 date-converted gene symbols restored via Gene Updater)
- Enrichr GO Biological Processes enrichment results table with adjusted p-values per term

## Outputs

- Filtered GO BP enrichment results containing only terms with adjusted p-value < 0.05 (e.g. enrichr_go_bp_results.csv)
- Ranked list of significant GO Biological Processes terms ordered by adjusted p-value

## How to apply

Submit the gene list (e.g. Table 1 date-converted genes such as MARCHF1) to Enrichr against the Gene Ontology (GO) Biological Processes database. Retrieve the full ranked term list with Enrichr's adjusted p-values (which account for the many GO BP terms tested), sort terms by adjusted p-value, and retain only terms with adjusted p-value < 0.05. Because many tools (Enrichr, GSEA, Ingenuity Pathway Analysis) rely on gene symbols, run this only after date-converted and old gene names have been restored to HGNC symbols, otherwise converted terms act as voids in the analysis. Validate that the retained significant terms cover the expected biology — here cell division, exocytosis, cilium assembly, protein ubiquitination and nitric oxide biosynthesis — before exporting (e.g. enrichr_go_bp_results.csv).

## Related tools

- **Enrichr** (Web tool that performs over-representation analysis of the gene list against the GO Biological Processes database and returns adjusted p-values used for term selection)
- **Gene Updater** (Pre-processing step that restores date-converted and old gene names to HGNC-approved symbols so the enrichment query list is complete) — https://github.com/kuanrongchan/date-to-gene-converter
- **Gene Ontology (GO) Biological Processes database** (Reference pathway database against which the gene terms are queried)

## Evaluation signals

- All terms in the exported results table have adjusted p-value < 0.05; no raw p-values below 0.05 without correction are retained
- Significant terms include those matching cell division, exocytosis, cilium assembly, protein ubiquitination and nitric oxide biosynthesis as reported in the paper
- Input gene list to Enrichr contains no residual date strings (Mar-01, Mar-02) or unrecognized symbols; every term maps to a current HGNC gene symbol
- The number of significant GO BP terms is far smaller than the number tested, confirming the multiple-testing correction was actually applied
- Results reproduce when the same gene list is re-submitted to Enrichr's GO BP library

## Limitations

- Date terms Mar-01 and Mar-02 can each map to two different genes (MTARC1/2 vs MARCHF1/2, which have contrasting biological functions); mis-assignment changes the enrichment input and hence the significant terms
- Enrichment tools rely on gene symbols, so date-converted or outdated names cause voids in pathway analysis that no p-value correction can recover
- The adjusted p-value threshold of 0.05 is a reporting convention from this study; other analyses may warrant different cutoffs or databases (e.g. Reactome)

## Evidence

- [abstract] Finding supports adjusted-p filtering on GO BP terms: "Molecular pathways related to cell division, exocytosis, cilium assembly, protein ubiquitination and nitric oxide biosynthesis were found to be most affected by Excel auto-conversion."
- [discussion] GO BP enrichment via Enrichr as the analysis substrate: "Gene terms annotated in Table 1 were used as the input data for pathway analysis. The genes were analysed against the Gene Ontology (GO) Biological Processes database6,7 using the Enrichr tool2."
- [intro] Gene-symbol dependency motivates corrected-symbol input before p-value interpretation: "many of the pathway enrichment tools such as Enrichr2, Gene set enrichment analysis (GSEA)3,4 and Ingenuity Pathway Analysis5 rely on gene symbols"
- [other] Task card threshold used to filter Enrichr GO BP results: [evidence span withheld — anchor status: not_found]
