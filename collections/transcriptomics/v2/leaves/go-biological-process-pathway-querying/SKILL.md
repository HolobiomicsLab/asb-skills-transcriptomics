---
name: go-biological-process-pathway-querying
description: Use when you hold a curated list of human gene symbols (e.g. date-converted
  genes recovered from Excel auto-conversion) and need to identify which GO Biological
  Process terms they enrich at adjusted p < 0.05.
when_to_use_negative:
- The input gene list still contains date-formatted (Mar-01, Mar-02) or deprecated
  gene symbols that have not been corrected — Enrichr and other pathway tools rely
  on gene symbols and will return void results
- The goal is analysis against other pathway resources (e.g. Reactome) rather than
  the GO Biological Processes database
- The analysis is a ranked-list GSEA-style run over a full expression matrix rather
  than enrichment of a discrete gene list via Enrichr
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: Enrichr
  role: Web tool used to query the gene list against the GO Biological Processes database
    and return ranked enrichment terms with adjusted p-values
- name: Gene Ontology (GO) Biological Processes database
  role: Reference pathway database against which the gene terms are enriched
- name: Gene Updater
  role: Upstream tool that converts Excel date-converted gene terms (e.g. Mar-01/Mar-02)
    back to HGNC-approved gene symbols so they are recognized by Enrichr
  repo: https://github.com/kuanrongchan/date-to-gene-converter
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-17104-3
    title: 'Gene Updater: a web tool that autocorrects and updates for Excel misidentified
      gene names'
  source_content_hash: sha256:0228ec487a702745b0d1eef01a184db8914f9d065a427d252aeffc2b97268379
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
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# go-biological-process-pathway-querying

## Summary

Submit a curated list of human gene symbols to the Enrichr tool against the Gene Ontology (GO) Biological Processes database, rank returned terms by adjusted p-value, and retain significant terms (adjusted p < 0.05) to identify the biological processes a gene set modulates.

## When to use

Use when you hold a curated list of human gene symbols (e.g. date-converted genes recovered from Excel auto-conversion) and need to identify which GO Biological Process terms they enrich at adjusted p < 0.05.

## When NOT to use

- The input gene list still contains date-formatted (Mar-01, Mar-02) or deprecated gene symbols that have not been corrected — Enrichr and other pathway tools rely on gene symbols and will return void results
- The goal is analysis against other pathway resources (e.g. Reactome) rather than the GO Biological Processes database
- The analysis is a ranked-list GSEA-style run over a full expression matrix rather than enrichment of a discrete gene list via Enrichr

## Inputs

- Gene list of human HGNC gene symbols (e.g. from Table 1 of the paper, loaded via pandas)
- Gene Ontology (GO) Biological Processes database (queried through Enrichr)

## Outputs

- Ranked GO Biological Process enrichment results filtered to adjusted p-value < 0.05, exported as enrichr_go_bp_results.csv

## How to apply

Load the gene list (e.g. Table 1 date-converted terms such as Mar-01/Mar-02 resolved via the Gene Updater web tool to MARCHF1/MTARC1 etc.) with pandas. Submit the gene symbols to Enrichr, querying against the Gene Ontology (GO) Biological Processes database — Enrichr requires recognized gene symbols, so date-formatted or deprecated names must first be corrected. Rank the returned GO BP terms by Enrichr's adjusted p-value and retain terms with adjusted p-value < 0.05. Export the filtered results as enrichr_go_bp_results.csv. Validate by checking that reproduced terms include cell division, exocytosis, cilium assembly, protein ubiquitination and nitric oxide biosynthesis.

## Related tools

- **Enrichr** (Web tool used to query the gene list against the GO Biological Processes database and return ranked enrichment terms with adjusted p-values)
- **Gene Ontology (GO) Biological Processes database** (Reference pathway database against which the gene terms are enriched)
- **Gene Updater** (Upstream tool that converts Excel date-converted gene terms (e.g. Mar-01/Mar-02) back to HGNC-approved gene symbols so they are recognized by Enrichr) — https://github.com/kuanrongchan/date-to-gene-converter

## Evaluation signals

- The filtered enrichment results must include GO BP terms corresponding to cell division, exocytosis, cilium assembly, protein ubiquitination and nitric oxide biosynthesis as reported in the paper
- All retained terms have adjusted p-value < 0.05; terms at or above the cutoff are excluded
- enrichr_go_bp_results.csv is exported and contains the ranked significant GO BP terms
- Every submitted gene symbol is a recognized HGNC symbol (no date strings or old names reaching Enrichr, which would create voids in the enrichment)

## Limitations

- Pathway enrichment tools such as Enrichr rely on gene symbols, so any Excel auto-converted dates in the input cause unrecognized terms and voids in the pathway analysis
- Mar-01 and Mar-02 are ambiguous: they can each map to two different genes (MTARC1-2 or MARCHF1-2, which have contrasting biological functions), so misassignment distorts enrichment results
- This workflow covers the GO Biological Processes database only, as used in the paper; other pathway databases (e.g. Reactome) are not queried here

## Evidence

- [discussion] Enrichr queried against GO BP with Table 1 gene terms: "Gene terms annotated in Table 1 were used as the input data for pathway analysis. The genes were analysed against the Gene Ontology (GO) Biological Processes database6,7 using the Enrichr tool2."
- [intro] Enrichr depends on recognized gene symbols: "many of the pathway enrichment tools such as Enrichr2, Gene set enrichment analysis (GSEA)3,4 and Ingenuity Pathway Analysis5 rely on gene symbols"
- [abstract] Expected enriched pathways reported by the paper: "Molecular pathways related to cell division, exocytosis, cilium assembly, protein ubiquitination and nitric oxide biosynthesis were found to be most affected by Excel auto-conversion."
