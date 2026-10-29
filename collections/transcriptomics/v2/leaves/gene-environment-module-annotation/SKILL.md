---
name: gene-environment-module-annotation
description: Use when you have a list of differentially expressed human genes (from
  microarray, bulk RNA-seq, or a clustered Seurat single-cell object) and want to
  test which curated environmental gene modules (infection, diet, cigarette smoking,
  toxic chemicals) are statistically over-represented.
when_to_use_negative:
- The question concerns genetic data types such as SNPs, copy number variants or DNA
  methylations — E.PAGE does not yet cover these.
- You need joint odds ratios or interaction odds ratios, which the paper states are
  not yet included in the statistical tests.
- The gene list is non-human or the environmental context falls outside the four curated
  factors (cigarette smoking, diet, infections, toxic chemicals).
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: E.PAGE
  role: 'R package performing the enrichment: process_input_text/process_input_seurat
    build the query, compute()/compute_sets() run the Fisher exact test with BH correction
    against the curated module database'
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: E.PATH
  role: Supplies the curated environmental pathways database (annotations tibble and
    gs_genes gene-set list) consumed by E.PAGE
  repo: https://github.com/AhmedMehdiLab/E.PATH
- name: Seurat
  role: Provides the clustered single-cell object that process_input_seurat() accepts
    to extract cluster DE gene lists
- name: R 4.0.3
  role: Runtime language in which E.PAGE is written and executed
provenance:
  source_task_ids:
  - task_006
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:e86dc2a61a860e9626953362cc071b1d667cdaa5712863dfa1aab1fcc2e07a38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - E.PATH
  - Gene Expression Omnibus (GEO)
  - R
  - R 4.0.3
  - Seurat
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# gene-environment-module-annotation

## Summary

Enrich a list of differentially expressed genes against the E.PATH/E.PAGE curated database of 522 manually annotated gene modules (25,789 genes, four environmental factors: cigarette smoking, diet, infections, toxic chemicals) using a Fisher exact (hypergeometric) test with Benjamini-Hochberg FDR correction, to reveal gene–environment associations behind a disease gene set.

## When to use

Use when you have a list of differentially expressed human genes (from microarray, bulk RNA-seq, or a clustered Seurat single-cell object) and want to test which curated environmental gene modules (infection, diet, cigarette smoking, toxic chemicals) are statistically over-represented.

## When NOT to use

- The question concerns genetic data types such as SNPs, copy number variants or DNA methylations — E.PAGE does not yet cover these.
- You need joint odds ratios or interaction odds ratios, which the paper states are not yet included in the statistical tests.
- The gene list is non-human or the environmental context falls outside the four curated factors (cigarette smoking, diet, infections, toxic chemicals).

## Inputs

- Text string or file of differentially expressed human gene symbols (e.g. 291 T1D PBMC DE genes, 229 RA DE genes from GSE12021/GSE55457/GSE55584/GSE55235)
- Seurat object with graph-based clusters already computed (single-cell RNA-seq, e.g. severe vs healthy COVID-19)
- E.PATH curated database: annotations tibble plus gs_genes list (522 modules, 25,789 genes, four environmental factors)
- Optional custom annotation and database .csv files via import_annotations()/import_database()

## Outputs

- tidyverse tibble of statistically enriched modules (results$stats) with padj, fold enrichment and odds ratios
- Category-split enrichment results (CSGE, IGE, DGE, TGE headings) from compute_sets()
- Collated CSV table of enriched modules (module name, module size, number of DE genes, padj, fold enrichment)

## How to apply

Prepare the gene list with E.PAGE's process_input_text(capitalize = TRUE) for text/gene-symbol input, or process_input_seurat() on a Seurat object that has already been clustered (the function performs the differential expression between clusters). Call compute() in R 4.0.3, which tests each of the 522 curated modules from the E.PATH database (25,789 genes mapped to four environmental factors) against the query using a Fisher exact test on the hypergeometric distribution, then applies Benjamini-Hochberg FDR correction to give adjusted p-values. Compute fold enrichment as (genes in module / query size) / (module genes / universe size) and report module size, number of DE genes, padj, and fold enrichment. Filter on padj to obtain significantly enriched modules (e.g. Inflammation, Infection type: Acute, Immune response, Insulin resistance, Cigarette smoking) and export the collated table (e.g. t1d_enrichment_table.csv or covid19_severe_enriched_modules.csv). Use compute_sets() instead of compute() when results must be split into the CSGE (cigarette smoking), IGE (infection), DGE (diet) and TGE (toxin) category headings. Validate by comparing reproduced rows against the paper's published tables within rounding tolerance (e.g. T1D Immune response padj 1.20E-04, FE 1.59; RA Infection type: Acute padj 8.49E-34, FE 2.63; COVID-19 severe Inflammation padj 1.18E-60, FE 3.39).

## Related tools

- **E.PAGE** (R package performing the enrichment: process_input_text/process_input_seurat build the query, compute()/compute_sets() run the Fisher exact test with BH correction against the curated module database) — https://github.com/AhmedMehdiLab/E.PAGE
- **E.PATH** (Supplies the curated environmental pathways database (annotations tibble and gs_genes gene-set list) consumed by E.PAGE) — https://github.com/AhmedMehdiLab/E.PATH
- **Seurat** (Provides the clustered single-cell object that process_input_seurat() accepts to extract cluster DE gene lists)
- **R 4.0.3** (Runtime language in which E.PAGE is written and executed)

## Examples

```
genes <- "ftl ApoE CTSZ"; input <- process_input_text(genes, capitalize = TRUE); results <- compute(input); results$stats
```

## Evaluation signals

- Reproduced rows match the paper's reported values within rounding tolerance, e.g. T1D 'Immune response' padj 1.20E-04 (FE 1.59) and 'Infection type: Acute' padj 7.46E-07 (FE 1.61).
- RA validation: 'Infection type: Acute' padj 8.49E-34 (FE 2.63) and 'Immune response' padj 1.03E-27 (FE 2.87).
- COVID-19 severe-case validation: top modules Inflammation (padj 1.18E-60, FE 3.39), Infection type: Acute (padj 1.02E-56), Immune response (padj 7.61E-49).
- Output object is a tidyverse tibble accessible via results$stats, with BH-adjusted p-values and fold enrichment columns present.
- Fold enrichment equals (genes in module / query size) / (module genes / universe size) when recomputed by hand.

## Limitations

- The study is limited to four types of environmental variables; more environmental datasets need to be integrated for wider community use.
- Possible dataset bias due to limited input data types (gene list only) and similarities between infection- and tissue damage-associated immune responses.
- Data collection is restricted to MeSH-term queries of the GEO database for differential gene expression data.
- Additional statistical tests such as joint odds ratio and interaction odds ratio are not yet included.

## Evidence

- [abstract] Curated database scope: "a database containing four environmental factors, cigarette smoking, diet, infections and toxic chemicals, along with a total of 25,789 genes"
- [abstract] Enrichment finding: "On testing, we uncovered statistically enriched biological processes, which revealed pathways associated with environmental factors and the genes"
- [methods] Package provenance: "The statistical analysis package E.PAGE (Environmental Pathways Affecting Gene Expression) (https://github.com/AhmedMehdiLab/E.PAGE) was written in R version 4.0.3"
- [methods] Seurat input mode: "E.PAGE used a Seurat object (with clustering performed) as an input and performs differential expression analyses between the clusters"
- [readme] README usage: "This is a data analysis package allowing the user with a list of genes of interest to find enriched environmental pathways using a curated database."
- [readme] Category split: "result will be a list with CSGE (cigarette smoking), IGE (infection), DGE (diet) and TGE (toxin) headings"
