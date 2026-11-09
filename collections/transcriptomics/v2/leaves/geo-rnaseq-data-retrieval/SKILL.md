---
name: geo-rnaseq-data-retrieval
description: Use when an analysis requires public embryo-development RNA-seq data
  — e.g. a specified GEO accession (GSE71434) or GEO-deposited scRNA-seq from cited
  references — that must be fetched and normalized before communication-strength or
  delta computation.
when_to_use_negative:
- The analysis targets maternal-embryo interactions or other non-autocrine settings
  for which the study's data were not designed
- Data are already provided as processed, quantile-normalized expression matrices
  (e.g. the demo seu_demo.rds/cds_demo.rds objects in the TimeTalk package) and only
  need the downstream RunTimeTalk call
- The needed dataset has no public GEO deposition and must instead be generated de
  novo by sequencing
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0204
tools:
- name: TimeTalk
  role: R analysis scripts that consume the retrieved and normalized pseudo-bulk expression
    to compute eLR interaction scores
  repo: https://github.com/ChengLiLab/TimeTalk
- name: R
  role: Custom R scripts for GEO data handling, pseudo-bulk calculation, and quantile
    normalization (all statistical analyses used R version 3.6.3)
  repo: https://github.com/ChengLiLab/TimeTalk
- name: preprocessCore
  role: Provides the normalize.quantiles function used for quantile normalization
    of the merged expression matrix (version 1.48.0)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:60a8b4addc57313501fc2381d238c55cafa86bc16dce10b28b92a6cb39a8c6a8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - TimeTalk
  - preprocessCore
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0204
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# geo-rnaseq-data-retrieval

## Summary

Retrieves public RNA-seq/scRNA-seq datasets (e.g. the ZGA-inhibition alpha-amanitin dataset GSE71434 and mouse pre-implantation scRNA-seq from GEO) and prepares them as normalized pseudo-bulk expression matrices for downstream TimeTalk eLR interaction-score analysis.

## When to use

Use when an analysis requires public embryo-development RNA-seq data — e.g. a specified GEO accession (GSE71434) or GEO-deposited scRNA-seq from cited references — that must be fetched and normalized before communication-strength or delta computation.

## When NOT to use

- The analysis targets maternal-embryo interactions or other non-autocrine settings for which the study's data were not designed
- Data are already provided as processed, quantile-normalized expression matrices (e.g. the demo seu_demo.rds/cds_demo.rds objects in the TimeTalk package) and only need the downstream RunTimeTalk call
- The needed dataset has no public GEO deposition and must instead be generated de novo by sequencing

## Inputs

- GEO accession identifiers (e.g. GSE71434 for alpha-amanitin ZGA-inhibition RNA-seq, ref. 30)
- GEO-deposited mouse pre-implantation scRNA-seq datasets (refs 12, 13)
- Gene expression count/raw matrices from downloaded GEO series

## Outputs

- Quantile-normalized merged expression matrix (preprocessCore normalize.quantiles output)
- Pseudo-bulk per-stage/per-condition expression profiles (early 2-cell, late 2-cell, alpha-amanitin-treated)
- Expression matrices ready for TimeTalk interaction-score computation

## How to apply

Identify the required datasets by their manuscript citations and GEO accessions: the ZGA-inhibition RNA-seq data come from the alpha-amanitin-treated embryo study (ref. 30, GSE71434), and mouse pre-implantation scRNA-seq datasets are downloaded from GEO per the paper's data-availability statement. After download, use custom R scripts to compute pseudo-bulk expression for an average single cell in each stage, since TimeTalk's interaction scores are computed on pseudo-bulk per-condition expression (early 2-cell, late 2-cell, alpha-amanitin-treated). Apply quantile normalization to the merged expression matrix using preprocessCore's 'normalize.quantiles' (version 1.48.0) so conditions are comparable before delta log-ratio interaction-score calculations. The rationale is that TimeTalk operates on normalized temporal-series expression, so correct retrieval, pseudo-bulk aggregation, and quantile normalization are prerequisites for reproducing results such as the 56 altered eLR pairs (Fig. 3e).

## Related tools

- **TimeTalk** (R analysis scripts that consume the retrieved and normalized pseudo-bulk expression to compute eLR interaction scores) — https://github.com/ChengLiLab/TimeTalk
- **R** (Custom R scripts for GEO data handling, pseudo-bulk calculation, and quantile normalization (all statistical analyses used R version 3.6.3)) — https://github.com/ChengLiLab/TimeTalk
- **preprocessCore** (Provides the normalize.quantiles function used for quantile normalization of the merged expression matrix (version 1.48.0))

## Examples

```
normalized <- preprocessCore::normalize.quantiles(merged_expr_matrix)
```

## Evaluation signals

- The downloaded GEO series matches the cited accession (GSE71434, alpha-amanitin-treated embryos from ref. 30) and contains the expected early 2-cell, late 2-cell, and treated conditions
- Merged expression matrix passes through normalize.quantiles from preprocessCore 1.48.0 without error and distributions are aligned across conditions
- Pseudo-bulk profiles are computed per stage for an average single cell, as in the paper's custom R scripts
- Downstream interaction scores on the prepared matrices reproduce the manuscript's ZGA-inhibition delta values, recovering the 56 altered eLR pairs (Fig. 3e, Supplementary Figs. 12-13)

## Limitations

- Granger causality conclusions are affected by the choice of parameter winsz, and the authors advise testing multiple parameter combinations on the prepared data
- The study's autocrine assumption precludes exploring maternal-embryo interactions, so retrieved embryo-only datasets cannot support that question
- Scarce high-temporal-resolution datasets limited validation of the framework built on these GEO inputs

## Evidence

- [other] Download the ZGA-inhibition RNA-seq dataset (GEO accession GSE71434, alpha-amanitin-treated embryos from ref. 30) from the GEO database.: [evidence span withheld — anchor status: not_found]
- [discussion] GEO sourcing statement: "scRNA-seq datasets were downloaded from the GEO database (see Availability of data and materials)."
- [discussion] pseudo-bulk computation: "Custom R scripts were used for the pre-implantation scRNA-seq data to calculate the pseudo-bulk expression for an average single cell in each stage."
- [discussion] quantile normalization: "Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0)."
