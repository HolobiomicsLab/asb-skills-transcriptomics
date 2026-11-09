---
name: gene-ontology-enrichment-analysis
description: Use when you have a small detected gene set (e.g., EDGE feature genes
  below the μ − 1.5 × σ entropy cutoff) and need to test whether it is enriched for
  GO biological processes relative to an appropriate background universe.
when_to_use_negative:
- Input is already an enrichment results table or a full ranked genome-wide gene list
  rather than a small detected feature gene set
- The goal is gene set enrichment analysis (GSEA) on a continuous ranking rather than
  over-representation testing on a discrete gene list
- No appropriate gene-universe background can be defined for the dataset, making hypergeometric
  P-values uninterpretable
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0089
tools:
- name: EDGE
  role: Upstream ensemble dimensionality reduction method whose entropy-based feature
    gene selection (μ − 1.5 × σ cutoff) produces the gene list submitted to GO enrichment
  repo: https://github.com/shawnstat/EDGE
- name: R
  role: Implementation environment for running the hypergeometric over-representation
    test, FDR adjustment, and table export
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-19465-7
    title: Ensemble dimensionality reduction and feature gene extraction for single-cell
      RNA-seq data
  source_content_hash: sha256:0748d29893ea5ef952e61c1497428465df98dc9766570d33a8a1e39abe0d50f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/shawnstat/EDGE
  tools:
  - EDGE
  - GOstats
  - R
  - UniBic
  - scHiCDiff
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0089
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# gene-ontology-enrichment-analysis

## Summary

Hypergeometric Gene Ontology biological-process over-representation testing on a short list of feature genes (e.g., the 35 EDGE-detected PBMC feature genes), producing a ranked table of enriched immune-response terms with fold enrichment, raw P-values, and FDR.

## When to use

Use when you have a small detected gene set (e.g., EDGE feature genes below the μ − 1.5 × σ entropy cutoff) and need to test whether it is enriched for GO biological processes relative to an appropriate background universe.

## When NOT to use

- Input is already an enrichment results table or a full ranked genome-wide gene list rather than a small detected feature gene set
- The goal is gene set enrichment analysis (GSEA) on a continuous ranking rather than over-representation testing on a discrete gene list
- No appropriate gene-universe background can be defined for the dataset, making hypergeometric P-values uninterpretable

## Inputs

- Set of detected feature genes (gene symbols/IDs), e.g., the 35 EDGE-selected PBMC feature genes
- Gene-universe background (all genes considered detectable in the scRNA-seq dataset)
- GO biological-process ontology annotations (gene-to-term mapping)

## Outputs

- Ranked enrichment table (ten rows for the EDGE PBMC case) with columns: GO term, fold enrichment, raw P-value, FDR
- Written table file, e.g., table3_go_enrichment.csv

## How to apply

Take the detected feature gene list (e.g., the 35 PBMC genes selected by EDGE at the μ − 1.5 × σ cutoff of averaged entropy values) and run a hypergeometric over-representation test for GO biological-process terms against an explicitly chosen gene-universe background. Compute fold enrichment for each term, adjust raw P-values for multiple testing with FDR, and rank terms by significance, keeping the top ten most enriched processes. For the EDGE PBMC case, all ten retained terms were immune-response- and response-to-stimulus-related (e.g., defense response, leukocyte-mediated immunity, immune effector process), which serves as a biological sanity check for the feature gene set.

## Related tools

- **EDGE** (Upstream ensemble dimensionality reduction method whose entropy-based feature gene selection (μ − 1.5 × σ cutoff) produces the gene list submitted to GO enrichment) — https://github.com/shawnstat/EDGE
- **R** (Implementation environment for running the hypergeometric over-representation test, FDR adjustment, and table export)

## Examples

```
enrich <- enrichment_analysis(genes = feature_genes_35, universe = gene_universe, ontology = "BP"); write.csv(top10_terms, "table3_go_enrichment.csv")
```

## Evaluation signals

- Output CSV contains exactly ten GO biological-process rows, each with fold enrichment, raw P-value, and FDR columns populated
- Enriched terms correspond to the immune-response processes reported in Table 3 (defense response, leukocyte-mediated immunity, immune effector process, response to stimulus)
- All reported FDR values are monotone with (≥) the corresponding raw P-values
- Feature gene input size matches the expected 35 genes from the μ − 1.5 × σ averaged-entropy cutoff
- Fold enrichment values are consistent with the hypergeometric expected/observed term counts under the stated universe

## Limitations

- Results depend on the choice of gene-universe background; an inappropriate universe inflates or deflates enrichment significance
- Only the top ten terms are retained by ranking, so weaker but real enrichments may be omitted
- The article does not report the specific GO annotation database version or the R package used for the hypergeometric test

## Evidence

- [other] workflow step: hypergeometric over-representation test against a gene-universe background: [evidence span withheld — anchor status: not_found]
- [other] finding: immune-related enriched processes: [evidence span withheld — anchor status: not_found]
- [discussion] feature gene cutoff definition: "We choose μ −  1.5 × σ as the cutoff value to select top feature genes"
- [abstract] EDGE purpose supporting feature gene extraction feeding enrichment: "EDGE is well suited for searching for meaningful organization of cells, detecting rare cell types, and identifying essential feature genes"
