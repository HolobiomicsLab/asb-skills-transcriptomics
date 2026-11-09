---
name: cross-disease-module-intersection-analysis
description: Use when two or more disease gene lists have each been run through E.PAGE
  enrichment and you need the set of significantly enriched environmental gene modules
  common to all queries.
when_to_use_negative:
- You have only one query signature and need enrichment itself, not a cross-disease
  intersection — use E.PAGE's compute() directly.
- The enrichment results are not significance-filtered and you want to intersect raw
  ranked module lists without an FDR decision point.
- The queries target different module databases or annotation versions, making module
  names non-comparable across diseases.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
tools:
- name: E.PAGE
  role: Computes the per-disease enrichment results (results$stats tibble with padj
    and fold enrichment) that are filtered and intersected
  repo: https://github.com/AhmedMehdiLab/E.PAGE
- name: E.PATH
  role: Provides the curated gene-module database (gs_genes, annotations) against
    which enrichment is performed
  repo: https://github.com/AhmedMehdiLab/E.PATH
- name: dplyr/tidyverse
  role: Filtering enrichment tibbles to significant modules and inner_join on gene-module
    names
- name: R (4.0.3)
  role: Runtime environment for E.PAGE and the intersection workflow
  repo: https://github.com/AhmedMehdiLab/E.PAGE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-022-21988-6
    title: Environmental pathways affecting gene expression (E.PAGE) as an R package
      to predict gene–environment associations
  source_content_hash: sha256:1b95e25f2b5a35985580b58ebc3e51ba1c53db849d3f643bc3b7003d000bf4c6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/AhmedMehdiLab/E.PAGE
  tools:
  - E.PAGE
  - E.PATH
  - R
  - R (4.0.3)
  - dplyr
  - dplyr/tidyverse
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# cross-disease-module-intersection-analysis

## Summary

Identifies gene modules enriched in common across two disease query signatures by intersecting the statistically significant (FDR-adjusted) E.PAGE enrichment results for each disease, e.g. the infection-associated modules shared by T1D and RA DE gene lists.

## When to use

Use when two or more disease gene lists have each been run through E.PAGE enrichment and you need the set of significantly enriched environmental gene modules common to all queries.

## When NOT to use

- You have only one query signature and need enrichment itself, not a cross-disease intersection — use E.PAGE's compute() directly.
- The enrichment results are not significance-filtered and you want to intersect raw ranked module lists without an FDR decision point.
- The queries target different module databases or annotation versions, making module names non-comparable across diseases.

## Inputs

- E.PAGE enrichment output tibbles (results$stats) for each disease query signature
- Disease DE gene lists (e.g. 291 T1D DE genes; 229 RA DE genes from GSE12021, GSE55457, GSE55584, GSE55235)
- E.PAGE padj (Benjamini–Hochberg) and fold-enrichment columns per module

## Outputs

- Tidy table of shared significant gene modules with padj and fold enrichment per disease query (e.g. shared_modules.csv)
- Set of common modules (e.g. Immune response, Interferons, IL-12, Transcription regulation)

## How to apply

Run E.PAGE (R 4.0.3, tidyverse/Seurat dependencies) separately on each disease DE gene list — e.g. 291 T1D DE genes and 229 RA DE genes from GEO/MSigDB C7 microarray datasets — and inspect the enrichment tibble in results$stats. Filter each tibble to statistically significant gene modules using the Benjamini–Hochberg FDR-adjusted p-value (padj) reported by E.PAGE. Intersect the two significant-module lists with a dplyr inner_join on gene-module names to obtain the shared set (for T1D/RA this yields Immune response, Interferons, IL-12 and Transcription regulation, dominated by infection-associated modules). Assemble a tidy table of shared module names with padj and fold enrichment from each disease query and export (e.g. shared_modules.csv).

## Related tools

- **E.PAGE** (Computes the per-disease enrichment results (results$stats tibble with padj and fold enrichment) that are filtered and intersected) — https://github.com/AhmedMehdiLab/E.PAGE
- **E.PATH** (Provides the curated gene-module database (gs_genes, annotations) against which enrichment is performed) — https://github.com/AhmedMehdiLab/E.PATH
- **dplyr/tidyverse** (Filtering enrichment tibbles to significant modules and inner_join on gene-module names)
- **R (4.0.3)** (Runtime environment for E.PAGE and the intersection workflow) — https://github.com/AhmedMehdiLab/E.PAGE

## Examples

```
library(E.PAGE); results_t1d <- compute(process_input_text(t1d_genes, capitalize=TRUE)); results_ra <- compute(process_input_text(ra_genes, capitalize=TRUE)); shared <- dplyr::inner_join(dplyr::filter(results_t1d$stats, padj<0.05), dplyr::filter(results_ra$stats, padj<0.05), by="module")
```

## Evaluation signals

- The shared-module table contains the modules reported as common in the paper (Interferons, IL-12, Transcription regulation, Immune response).
- Both T1D and RA padj and fold-enrichment values are populated for every row of the shared table (no missing columns from either query).
- Every retained module has a significant FDR-adjusted padj in BOTH disease queries before the inner_join.
- Shared modules are dominated by infection-associated gene modules, consistent with the paper's finding for T1D and RA.
- Intersection row count equals the size of the inner_join of the two filtered tibbles on module names (no lost or duplicated modules).

## Limitations

- The curated database covers only four environmental factors (cigarette smoking, diet, infections, toxic chemicals); shared enrichment outside these categories cannot be detected.
- Possible dataset bias due to limited input data types and similarities between infection and tissue damage-associated immune responses can confound cross-disease interpretation.
- The study is limited to MeSH-term queries of GEO for differential gene expression data, restricting which disease gene lists can be compared.
- E.PAGE's stats tibble reflects over-representation only; joint odds ratio and interaction odds ratio tests are not yet included.

## Evidence

- [discussion] Filter to significant modules and intersect: "The T1D and RA associated DE genes were found to be primarily enriched in infection-associated gene modules"
- [readme] E.PAGE produces enrichment stats tibble: "Statistically enriched annotations are stored in a `tidyverse` `tibble`, and can be viewed with:

``` r
results$stats"
- [intro] E.PAGE purpose: "This is a data analysis package allowing the user with a list of genes of interest to find enriched environmental pathways using a curated database."
- [methods] R version and dependencies: "The statistical analysis package E.PAGE (Environmental Pathways Affecting Gene Expression) (https://github.com/AhmedMehdiLab/E.PAGE) was written in R version 4.0.3"
- [discussion] Scope limitation to four environmental factors: [evidence span withheld — anchor status: not_found]
- [discussion] Dataset bias caveat: "there is a possibility of dataset bias due to limited type of input data such as gene list, similarities between infection and tissue damage -associated immune responses."
