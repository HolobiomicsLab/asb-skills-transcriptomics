---
name: gene-weight-top-contributor-ranking
description: Use when a rROMA analysis flags a module as over-dispersed or shifted
  and you need to identify which individual genes carry the largest PC1 weights, e.g.
  to nominate a lead gene for follow-up in a CF vs healthy comparison.
when_to_use_negative:
- The module has not yet been run through rRoma.R, so no PC1 weights or ModuleMatrix
  statistics exist to rank.
- The goal is differential pathway activity (shift testing with 'ppv Median Exp' <=
  0.05) rather than identifying individual gene drivers within a module.
- The expression matrix is not normalized (rRoma requires cpm, vst, or equivalent
  normalized data).
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3523
tools:
- name: rROMA
  role: R package providing the GetTopContrib function that ranks genes by their PC1
    weights within a module
  repo: github.com/sysbio-curie/rROMA
- name: edgeR
  role: Supplies the cpm normalization applied to the expression matrix before the
    rRoma run whose weights are ranked
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:ccc6cf2547947cdf6d17de655b2a9a964fdffd67f3de0a00bac26c4133c046db
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - edgeR
  - irlba
  - rROMA
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3523
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# Gene-weight top-contributor ranking

## Summary

Ranks the genes driving a rROMA module's PC1 activity by their absolute weights in the first principal component, using the rRoma GetTopContrib function to identify lead drivers such as GSN in the COAGULATION hallmark module.

## When to use

Use when a rROMA analysis flags a module as over-dispersed or shifted and you need to identify which individual genes carry the largest PC1 weights, e.g. to nominate a lead gene for follow-up in a CF vs healthy comparison.

## When NOT to use

- The module has not yet been run through rRoma.R, so no PC1 weights or ModuleMatrix statistics exist to rank.
- The goal is differential pathway activity (shift testing with 'ppv Median Exp' <= 0.05) rather than identifying individual gene drivers within a module.
- The expression matrix is not normalized (rRoma requires cpm, vst, or equivalent normalized data).

## Inputs

- rRoma output object (rRoma.output) containing ModuleMatrix statistics
- cpm-normalized gene expression matrix (genes as rows, samples as columns) used in the rRoma.R run
- Selected geneset definition (e.g. HALLMARK module with prefix stripped via sub('HALLMARK_', '', Name))

## Outputs

- Ranked gene-weight table of top-contributing genes for a module (exportable as CSV)
- Visualization plot of per-gene PC1 weights (GetTopContrib Plot = TRUE)

## How to apply

After running rRoma.R on a cpm-normalized expression matrix (e.g. MatData) against MSigDB HALLMARK gene sets from MSigDB v6.2 selected with SelectFromMSIGdb, call GetTopContrib on the rRoma output to compute per-gene contributions to the module's PC1. Use nGenes = 0.1 to keep the top ~10% of genes, OrderType = 'Abs' to rank by absolute weight, Mode = 'Wei' to use gene weights, and Plot = TRUE for a weight visualization. The rationale is that PC1 weights quantify how strongly each gene aligns with the module's coordinated expression axis, so extreme-weight genes are the putative drivers of the module's activity. In the GSE176121 CF application, this reproduced GSN (weight ~15.6) as the top contributor of the COAGULATION module, which was flagged as over-dispersed ('ppv L1' <= 0.05) but not shifted ('ppv Median Exp' > 0.05). Save the resulting gene-weight table as a CSV for reproducibility.

## Related tools

- **rROMA** (R package providing the GetTopContrib function that ranks genes by their PC1 weights within a module) — github.com/sysbio-curie/rROMA
- **edgeR** (Supplies the cpm normalization applied to the expression matrix before the rRoma run whose weights are ranked)

## Examples

```
GetTopContrib(rRoma.output, nGenes = 0.1, OrderType = 'Abs', Mode = 'Wei', Plot = TRUE)
```

## Evaluation signals

- The top-ranked contributor of the COAGULATION module in the GSE176121 CF analysis is GSN with weight ~15.6.
- With nGenes = 0.1 the returned table contains roughly the top 10% of genes in the tested module.
- The ranked weights correspond to absolute PC1 loadings (OrderType = 'Abs', Mode = 'Wei'), so the highest-weight gene has the largest absolute loading on the module's first principal component.
- The gene-weight table can be exported as CSV and matches the module flagged as over-dispersed ('ppv L1' <= 0.05, 'ppv Median Exp' > 0.05) in the ModuleMatrix.

## Limitations

- The CF case study lacks statistical power; the authors note it would require additional transcriptional datasets and testing of several gene set reference databases.
- Gene-weight ranking depends on a fixed random seed in rRoma.R for the random gene set null distributions; unseeded runs may give slightly different statistics.
- Default hyperparameters such as the extreme-weight percentage used for weight computation are not reported in the article.

## Evidence

- [methods] To do this we use the `GetTopContrib` function: "To do this we use the `GetTopContrib` function"
- [full_text] Ranking procedure: [evidence span withheld — anchor status: not_found]
- [discussion] Findings quote: "out of the 50 hallmark pathways tested, 3 were significantly active: FATTY_ACID_METABOLISM, APICAL_SURFACE, and COAGULATION."
- [readme] Normalized data requirement: "Warning: rRoma works with **normalized** data. Here cpm from edgeR was used for normalisation"
