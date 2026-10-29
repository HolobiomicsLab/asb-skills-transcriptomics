---
name: smoking-gene-module-interpretation
description: Use when you have a DE gene list or a clustered Seurat object (e.g. smokers
  vs non-smokers airway epithelium) and want to find statistically enriched environmental
  gene modules, especially cigarette smoking, airway epithelium, or COPD modules.
when_to_use_negative:
- Your question concerns genetic data types such as SNPs, copy number variants, or
  DNA methylations — the E.PAGE database does not yet cover these
- You need joint odds ratio or interaction odds ratio statistics — the authors state
  these tests are not yet included
- Your environmental exposure is outside the four curated factors (cigarette smoking,
  diet, infections, toxic chemicals) — the paper notes more environmental datasets
  need integrating
edam_operation: http://edamontology.org/operation_3945
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: E.PAGE
  role: Performs enrichment of annotated environmental gene modules against the query
    via Fisher exact / hypergeometric test with Benjamini-Hochberg FDR correction;
    provides process_input_seurat, compute, results$stats
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: E.PATH
  role: Companion package supplying the curated environmental database (gs_genes)
    and annotations tibble consumed by E.PAGE
  repo: https://github.com/AhmedMehdiLab/E.PATH
- name: Seurat
  role: Provides the clustered single-cell Seurat object that E.PAGE's process_input_seurat
    converts into a query gene list via inter-cluster differential expression
- name: R 4.0.3 / tidyverse
  role: Runtime and tidyverse tibble handling for E.PAGE results and writing the enrichment
    CSV
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:3858fe9e4cee8c472e6b1615bba6d53a0d8bf5b7b88e41a780767c01c6c93aa7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - E.PATH
  - R 4.0.3
  - R 4.0.3 / tidyverse
  - RStudio
  - Seurat
  edam_operation: http://edamontology.org/operation_3945
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# smoking-gene-module-interpretation

## Summary

Interpret a differentially expressed gene list (e.g. from smoking vs non-smoking airway epithelium single-cell data) as enriched cigarette-smoking-related gene modules using the E.PAGE R package and its curated 237-dataset / 25,789-gene environmental database.

## When to use

Use when you have a DE gene list or a clustered Seurat object (e.g. smokers vs non-smokers airway epithelium) and want to find statistically enriched environmental gene modules, especially cigarette smoking, airway epithelium, or COPD modules.

## When NOT to use

- Your question concerns genetic data types such as SNPs, copy number variants, or DNA methylations — the E.PAGE database does not yet cover these
- You need joint odds ratio or interaction odds ratio statistics — the authors state these tests are not yet included
- Your environmental exposure is outside the four curated factors (cigarette smoking, diet, infections, toxic chemicals) — the paper notes more environmental datasets need integrating

## Inputs

- Seurat object (with clustering performed) containing smokers vs non-smokers airway epithelium single-cell RNAseq data, plus the cluster identifier
- Alternatively, a text list of differentially expressed genes of interest
- E.PATH::annotations tibble and E.PATH::database (gs_genes list) mapping gene sets to annotations

## Outputs

- results$stats: tidyverse tibble of enriched gene modules with number of DE genes, adjusted p-value, fold enrichment, and odds ratio
- smoking_enrichment_results.csv: enrichment table (gene module, number of DE genes, adjusted p-value, fold enrichment)
- Optional plots from plot_stats/plot_overlap confirming module rankings

## How to apply

Load the single-cell RNAseq atlas (e.g. smokers vs non-smokers airway epithelium) as a Seurat object with graph-based clustering already performed, then call E.PAGE's process_input_seurat supplying the object and cluster identifier so differential expression between Seurat clusters yields the query gene list. Run compute(), which tests each annotated gene module against the curated environmental database (four environmental factors — cigarette smoking, diet, infections, toxic chemicals — and 25,789 genes) via Fisher exact / hypergeometric testing with Benjamini-Hochberg FDR correction, returning adjusted p-values, fold enrichment, and odds ratios. Inspect the tidyverse tibble in results$stats filtered on adjusted p-value, and optionally confirm module rankings with plot_stats/plot_overlap. Validate against the paper's Table 7, where the smoking single-cell query ranked Cigarette smoking, Airway epithelium, Epithelial gene expression, and Chronic obstructive pulmonary disease as top modules (e.g. Epithelial gene expression padj 3.07E-87).

## Related tools

- **E.PAGE** (Performs enrichment of annotated environmental gene modules against the query via Fisher exact / hypergeometric test with Benjamini-Hochberg FDR correction; provides process_input_seurat, compute, results$stats) — https://github.com/AhmedMehdiLab/E.PAGE
- **E.PATH** (Companion package supplying the curated environmental database (gs_genes) and annotations tibble consumed by E.PAGE) — https://github.com/AhmedMehdiLab/E.PATH
- **Seurat** (Provides the clustered single-cell Seurat object that E.PAGE's process_input_seurat converts into a query gene list via inter-cluster differential expression)
- **R 4.0.3 / tidyverse** (Runtime and tidyverse tibble handling for E.PAGE results and writing the enrichment CSV)

## Examples

```
seurat_obj <- readRDS("ex_seurat.rds"); input <- process_input_seurat(seurat_obj, 0); results <- compute(input); results$stats
```

## Evaluation signals

- Top-ranked modules in results$stats must include Cigarette smoking, Airway epithelium, Epithelial gene expression, and Chronic obstructive pulmonary disease, matching Table 7 of the paper
- Epithelial gene expression module should show an extremely significant adjusted p-value on the order of 3.07E-87 for the smoking single-cell query
- All reported p-values are Benjamini-Hochberg adjusted; fold enrichment and odds ratios must be present for each enriched module
- Enrichment is computed against a database of 237 datasets / 25,789 genes; zero enriched modules suggests the input gene list or cluster identifier was malformed
- Output CSV contains the required columns: gene module, number of DE genes, adjusted p-value, fold enrichment

## Limitations

- Database limited to four environmental factor types; the authors note more environmental datasets need to be integrated for wider community use
- Possible dataset bias due to limited input data types (gene lists) and similarities between infection and tissue damage-associated immune responses
- Study restricted to MeSH-term queries of GEO for differential gene expression data
- Genetic data types (SNPs, CNVs, DNA methylations) not yet covered

## Evidence

- [discussion] smoking modules in single-cell query: "we found gene modules for Cigarette smoking, Airway epithelium, Epithelial gene expression, and Chronic obstructive pulmonary disease."
- [methods] Seurat input handling: "E.PAGE used a Seurat object (with clustering performed) as an input and performs differential expression analyses between the clusters"
- [abstract] database scope: "a database containing four environmental factors, cigarette smoking, diet, infections and toxic chemicals, along with a total of 25,789 genes"
- [readme] results format: "Statistically enriched annotations are stored in a `tidyverse` `tibble`, and can be viewed with:

``` r
results$stats"
- [abstract] statistics: "After title and abstract screening against pre-set criteria, a total of 237 datasets were selected and 522 gene modules were manually annotated"
