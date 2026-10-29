---
name: toxic-chemical-gene-module-annotation
description: Use when you have a short list of differentially expressed genes (e.g.
  ~27 DE genes from a toxicant-exposure microarray study like cobalt-exposed rat liver
  cell lines) and want to test whether environmental toxic-chemical gene modules are
  enriched.
when_to_use_negative:
- The gene list comes from a non-environmental factor of interest outside the database's
  four environmental factors (cigarette smoking, diet, infections, toxic chemicals)
  — the database cannot cover other exposure types
- Input genetic data are SNPs, CNVs or DNA methylations rather than a gene list —
  these data types are not yet covered by the tool
- You need joint or interaction odds-ratio statistics — these statistical tests are
  not yet included
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3321
- http://edamontology.org/topic_0621
tools:
- name: E.PAGE
  role: R package performing the Fisher exact enrichment of toxic-chemical gene modules
    against the curated database; provides process_input_text, compute, compute_sets
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: E.PATH
  role: Companion R package providing the curated environmental pathways database
    (annotations tibble and gs_genes list)
  repo: https://github.com/AhmedMehdiLab/E.PATH
- name: R
  role: Implementation language (E.PAGE written in R version 4.0.3)
- name: tidyverse
  role: Dependency; enrichment results returned as tidyverse tibbles
- name: Seurat
  role: Dependency enabling Seurat object input (with clustering performed) for differential
    expression between clusters
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:9e30c4cc92eb06db73a259913c85fd5bd77576dd1d83742bda912c56d755a727
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - E.PATH
  - R
  - Seurat
  - tidyverse
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3321
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# toxic-chemical-gene-module-annotation

## Summary

Annotate a differentially-expressed gene list against the E.PAGE curated toxic-chemical gene modules using Fisher exact (hypergeometric) enrichment with Benjamini-Hochberg FDR correction, to link toxic-exposure signatures to modules such as Genotoxicity and Carcinogen.

## When to use

Use when you have a short list of differentially expressed genes (e.g. ~27 DE genes from a toxicant-exposure microarray study like cobalt-exposed rat liver cell lines) and want to test whether environmental toxic-chemical gene modules are enriched.

## When NOT to use

- The gene list comes from a non-environmental factor of interest outside the database's four environmental factors (cigarette smoking, diet, infections, toxic chemicals) — the database cannot cover other exposure types
- Input genetic data are SNPs, CNVs or DNA methylations rather than a gene list — these data types are not yet covered by the tool
- You need joint or interaction odds-ratio statistics — these statistical tests are not yet included

## Inputs

- List of differentially expressed genes (space-separated gene symbols string or Seurat object with clustering performed)
- E.PAGE curated toxic-chemical gene module database (TGE heading; 644 toxic-chemical-associated genes)
- Optional custom annotation (.csv) and database (.csv) files via import_annotations()/import_database()
- Optional org.Hs.eg.db for Gene Ontology-based analysis

## Outputs

- Enrichment table (tidyverse tibble, results$stats) with columns: gene module, number of modules, number of DE genes, p_adj, fold enrichment
- Per-category results list with CSGE, IGE, DGE and TGE (toxic-chemical) headings from compute_sets()

## How to apply

Install the E.PAGE R package from github.com/AhmedMehdiLab/E.PAGE and preprocess the gene list into E.PAGE input format with process_input_text (capitalize=TRUE for gene symbols). Run compute() against the default curated database — which spans four environmental factors (cigarette smoking, diet, infections, toxic chemicals) totalling 25,789 genes in 522 manually annotated modules, of which the toxic-chemical module set (TGE) contains 644 toxic-chemical-associated genes — testing each module against the query by Fisher exact test on the hypergeometric distribution. Correct module p-values with Benjamini-Hochberg FDR to obtain p_adj, and compute fold enrichment as the ratio of module genes in the query to module genes in the universe background. Filter on p_adj and assemble a table with columns gene module, number of modules, number of DE genes, p_adj, fold enrichment; results are returned as a tidyverse tibble viewable via results$stats, and toxic-chemical-specific results via compute_sets() under the TGE heading. Validation is demonstrated by the cobalt-exposure case: 27 DE genes from two cobalt-exposed rat liver derived cell lines recovered six modules (Genotoxic, Non-genotoxic, Carcinogen, Hepatocarcinogens, Liver-based in vitro models, Chemical induced gene expression), each with all 27 query genes, fold enrichment ~12 and p_adj = 1.84E-05.

## Related tools

- **E.PAGE** (R package performing the Fisher exact enrichment of toxic-chemical gene modules against the curated database; provides process_input_text, compute, compute_sets) — https://github.com/AhmedMehdiLab/E.PAGE
- **E.PATH** (Companion R package providing the curated environmental pathways database (annotations tibble and gs_genes list)) — https://github.com/AhmedMehdiLab/E.PATH
- **R** (Implementation language (E.PAGE written in R version 4.0.3))
- **tidyverse** (Dependency; enrichment results returned as tidyverse tibbles)
- **Seurat** (Dependency enabling Seurat object input (with clustering performed) for differential expression between clusters)

## Examples

```
library(E.PAGE); input <- process_input_text("GENE1 GENE2 GENE3", capitalize = TRUE); results <- compute(input); results$stats
```

## Evaluation signals

- Cobalt-exposure validation: the recovered table must contain the six reported modules (Genotoxic, Non-genotoxic, Carcinogen, Hepatocarcinogens, Liver-based in vitro models, Chemical induced gene expression), each with all 27 query genes
- Significance check: each validated module should show p_adj = 1.84E-05 with fold enrichment ~12
- All reported module p-values must be Benjamini-Hochberg FDR-corrected across the modules tested
- Output must be a tidyverse tibble accessible via results$stats, with the five columns: gene module, number of modules, number of DE genes, p_adj, fold enrichment
- compute_sets() must return a list with the TGE (toxin) heading populated when toxic-chemical modules are enriched

## Limitations

- Only four environmental variable types (cigarette smoking, diet, infections, toxic chemicals) are included; more environmental datasets would be needed for wider community use
- Possible dataset bias due to limited input data types such as gene lists, and similarities between infection and tissue damage-associated immune responses
- The study is restricted to MeSH-term queries of the GEO database for differential gene expression data
- The toxic-chemical module set contains 644 genes, so short query lists that intersect poorly with module genes may yield no significant enrichment

## Evidence

- [abstract] a database containing four environmental factors, cigarette smoking, diet, infections and toxic chemicals, along with a total of 25,789 genes: "a database containing four environmental factors, cigarette smoking, diet, infections and toxic chemicals, along with a total of 25,789 genes"
- [readme] This is a data analysis package allowing the user with a list of genes of interest to find enriched environmental pathways using a curated database.: "This is a data analysis package allowing the user with a list of genes of interest to find enriched environmental pathways using a curated database."
- [methods] The statistical analysis package E.PAGE (Environmental Pathways Affecting Gene Expression) (https://github.com/AhmedMehdiLab/E.PAGE) was written in R version 4.0.3: "The statistical analysis package E.PAGE (Environmental Pathways Affecting Gene Expression) (https://github.com/AhmedMehdiLab/E.PAGE) was written in R version 4.0.3"
- [discussion] On testing, we found the modules Genotoxicity and Carcinogen to be enriched.: "On testing, we found the modules Genotoxicity and Carcinogen to be enriched."
- [readme] Statistically enriched annotations are stored in a tidyverse tibble, and can be viewed with results$stats: "Statistically enriched annotations are stored in a `tidyverse` `tibble`, and can be viewed with results$stats"
- [discussion] Our study is limited to four types of environmental variables, therefore to increase usage towards wider community more environmental datasets need to be integrated.: [evidence span withheld — anchor status: not_found]
