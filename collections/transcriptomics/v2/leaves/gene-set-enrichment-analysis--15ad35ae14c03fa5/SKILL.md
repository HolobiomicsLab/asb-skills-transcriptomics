---
name: gene-set-enrichment-analysis
description: Use when a list of gene symbols — especially one recovered from Excel
  date-conversion errors (Mar-01, Mar-02) — must be functionally annotated via GO
  Biological Processes enrichment, with terms filtered at adjusted p < 0.05.
when_to_use_negative:
- The gene list still contains unresolved Excel date terms (Mar-01, Mar-02) or old
  gene symbols — run Gene Updater first, since Enrichr requires current gene symbols
- The input is a full ranked expression matrix rather than a discrete gene list of
  converted genes (GSEA-style ranking, not over-representation, is indicated)
- The question targets pathway sets outside GO Biological Processes (e.g. Reactome)
  without a stated GO BP query
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: Enrichr
  role: Web tool used to perform pathway enrichment of the gene list against the Gene
    Ontology (GO) Biological Processes database; returns adjusted p-values for ranking
    terms
- name: Gene Updater
  role: Upstream tool that converts Excel date terms (Mar-01, Mar-02) and old names
    back to HGNC-recommended symbols so the gene list is queryable by Enrichr
  repo: https://github.com/kuanrongchan/date-to-gene-converter
- name: pandas
  role: Used to load the Table 1 gene list file prior to submission to Enrichr (pandas
    >= 1.2.5 recommended)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:4e6442cd939a5e6f5f440a06bd8ba5e42f43834ffb0b7df2ed4c7750c814220b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kuanrongchan/date-to-gene-converter
  tools:
  - Enrichr
  - Gene Ontology (GO) Biological Processes database
  - Gene Updater
  - pandas
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# gene-set-enrichment-analysis

## Summary

Over-representation analysis of a gene list (here, Excel date-converted genes such as Mar-01/MARCHF1) against the Gene Ontology Biological Processes database using the Enrichr tool, ranking returned terms by adjusted p-value to identify which biological processes the genes modulate.

## When to use

Use when a list of gene symbols — especially one recovered from Excel date-conversion errors (Mar-01, Mar-02) — must be functionally annotated via GO Biological Processes enrichment, with terms filtered at adjusted p < 0.05.

## When NOT to use

- The gene list still contains unresolved Excel date terms (Mar-01, Mar-02) or old gene symbols — run Gene Updater first, since Enrichr requires current gene symbols
- The input is a full ranked expression matrix rather than a discrete gene list of converted genes (GSEA-style ranking, not over-representation, is indicated)
- The question targets pathway sets outside GO Biological Processes (e.g. Reactome) without a stated GO BP query

## Inputs

- Gene list of date-converted genes (Table 1 of the Gene Updater paper), e.g. as .csv or .xlsx with gene names in the first column
- HGNC-corrected gene symbols produced by the Gene Updater web tool (date terms Mar-01/Mar-02 resolved to assigned symbols)

## Outputs

- enrichr_go_bp_results.csv — GO Biological Process terms ranked by Enrichr adjusted p-value, filtered to adjusted p < 0.05

## How to apply

Load the gene list (e.g. Table 1 date-converted genes from the Gene Updater conversion mapping, with Mar-01 resolved to its assigned symbol such as MARCHF1) with pandas, ensuring gene names are in the first column of a .csv or .xlsx file. Submit the list to the Enrichr tool and query it against the Gene Ontology (GO) Biological Processes database — Enrichr is appropriate because pathway enrichment tools rely on gene symbols, so date-converted or stale symbols must first be corrected. Rank the returned GO BP terms by Enrichr's adjusted p-value and retain terms with adjusted p-value < 0.05. Export the filtered results as enrichr_go_bp_results.csv. Validate by checking that reproduced terms correspond to the expected processes: cell division, exocytosis, cilium assembly, protein ubiquitination, and nitric oxide biosynthesis.

## Related tools

- **Enrichr** (Web tool used to perform pathway enrichment of the gene list against the Gene Ontology (GO) Biological Processes database; returns adjusted p-values for ranking terms)
- **Gene Updater** (Upstream tool that converts Excel date terms (Mar-01, Mar-02) and old names back to HGNC-recommended symbols so the gene list is queryable by Enrichr) — https://github.com/kuanrongchan/date-to-gene-converter
- **pandas** (Used to load the Table 1 gene list file prior to submission to Enrichr (pandas >= 1.2.5 recommended))

## Examples

```
import pandas as pd; genes = pd.read_csv('table1_genes.csv').iloc[:,0].tolist()  # submit list to Enrichr vs GO Biological Processes, filter adjusted p < 0.05
```

## Evaluation signals

- The exported enrichr_go_bp_results.csv contains only GO BP terms with adjusted p-value < 0.05
- The reproduced enriched terms include processes matching the paper's findings: cell division, exocytosis, cilium assembly, protein ubiquitination, and nitric oxide biosynthesis
- Mar-01/Mar-02 entries in the input were explicitly resolved (e.g. to MARCHF1) rather than passed as dates to Enrichr
- Output file is valid CSV with term-name and adjusted-p-value columns traceable to the Enrichr response

## Limitations

- Mar-01 and Mar-02 can each map to two different genes (MTARC1 vs MARCHF1, MTARC2 vs MARCHF2), which have contrasting biological functions — misassignment changes the enrichment input
- Manual date-to-text conversion in Excel is tedious and error-prone, so date-converted lists can carry silent errors before enrichment
- Approximately one-fifth of published papers with supplementary Excel gene lists contain erroneous gene name conversions, so input provenance must be checked

## Evidence

- [abstract] Molecular pathways related to cell division, exocytosis, cilium assembly, protein ubiquitination and nitric oxide biosynthesis were found to be most affected by Excel auto-conversion.: "Molecular pathways related to cell division, exocytosis, cilium assembly, protein ubiquitination and nitric oxide biosynthesis were found to be most affected by Excel auto-conversion."
- [intro] many of the pathway enrichment tools such as Enrichr2, Gene set enrichment analysis (GSEA)3,4 and Ingenuity Pathway Analysis5 rely on gene symbols: "many of the pathway enrichment tools such as Enrichr2, Gene set enrichment analysis (GSEA)3,4 and Ingenuity Pathway Analysis5 rely on gene symbols"
- [discussion] Gene terms annotated in Table 1 were used as the input data for pathway analysis. The genes were analysed against the Gene Ontology (GO) Biological Processes database: "Gene terms annotated in Table 1 were used as the input data for pathway analysis. The genes were analysed against the Gene Ontology (GO) Biological Processes database"
- [readme] Users can upload their .csv or .xlsx file or files. Ensure that the first column contains the gene names.: "Users can upload their .csv or .xlsx file or files. Ensure that the first column contains the gene names."
