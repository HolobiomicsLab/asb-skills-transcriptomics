---
name: rna-seq-differential-expression-analysis
description: Use when a normalized RNA-seq expression matrix with two-group labels
  (e.g., 46 critical vs 23 non-critical COVID-19 cases from SEMdata) must be turned
  into per-gene DEG P-values to seed network analyses.
when_to_use_negative:
- The expression data are raw counts requiring normalization first — limma linear
  modeling here assumes normalized RNA-seq data as input.
- The goal is already perturbation analysis (SEMrun node/edge models) on an existing
  graph rather than deriving the DEG P-values that seed it.
- The dataset is not the two-group benchmark (e.g., single-group or time-series designs)
  since the design matrix here encodes the critical-vs-non-critical contrast.
edam_operation: http://edamontology.org/operation_3563
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: limma
  role: Empirical Bayes gene-wise linear model fitting and moderated statistics for
    the critical-vs-non-critical differential expression contrast
- name: SEMdata
  role: Supplies the benchmark GSE172114 COVID-19 RNA-seq expression matrix and group
    labels shipped with the package
  repo: https://github.com/fernandoPalluzzi/SEMdata
- name: org.Hs.eg.db
  role: Gene ID conversion (e.g., Entrez to gene symbols via mapIds) so the DEG table
    matches downstream graph node labels
- name: R
  role: Runtime environment in which limma, SEMdata, and SEMgraph are executed
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: SEMgraph
  role: Downstream consumer of the exported DEG P-value/seed table for active subnetwork
    detection and SEMtree analyses
  repo: https://github.com/fernandoPalluzzi/SEMgraph
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:599ac445db1100904a8bc5694a127e31ebb2d82f68b7ce827f9b954317d75106
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fernandoPalluzzi/SEMdata
  tools:
  - R
  - SEMdata
  - SEMgraph
  - SEMtree()
  - limma
  - org.Hs.eg.db
  edam_operation: http://edamontology.org/operation_3563
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# rna-seq-differential-expression-analysis

## Summary

Computes gene-level differential expression P-values on the GSE172114 COVID-19 RNA-seq benchmark (46 critical cases vs 23 non-critical controls) using limma's empirical Bayes linear modeling with Benjamini–Hochberg adjustment, producing the seed/P-value table consumed by downstream SEMtree and active subnetwork analyses.

## When to use

Use when a normalized RNA-seq expression matrix with two-group labels (e.g., 46 critical vs 23 non-critical COVID-19 cases from SEMdata) must be turned into per-gene DEG P-values to seed network analyses.

## When NOT to use

- The expression data are raw counts requiring normalization first — limma linear modeling here assumes normalized RNA-seq data as input.
- The goal is already perturbation analysis (SEMrun node/edge models) on an existing graph rather than deriving the DEG P-values that seed it.
- The dataset is not the two-group benchmark (e.g., single-group or time-series designs) since the design matrix here encodes the critical-vs-non-critical contrast.

## Inputs

- Normalized COVID-19 RNA-seq expression matrix from SEMdata (GSE172114, Carapito R. et al. 2022)
- Binary group vector of experimental conditions (46 critical cases vs 23 non-critical controls)

## Outputs

- covid19_deg_table.csv with one row per gene (gene ID, log2FC, raw P-value, BH-adjusted P-value)
- Seed gene list / P-value vector feeding the five active subnetwork detection methods and SEMtree analyses

## How to apply

Load the normalized expression matrix and group labels (GSE172114: 46 critical vs 23 non-critical controls) from the SEMdata package in R. Build a limma design matrix encoding the critical-vs-non-critical contrast and fit gene-wise linear models, then apply limma's empirical Bayes variance moderation to extract per-gene moderated t-statistics, log2 fold changes, and raw P-values. Adjust P-values across all genes with Benjamini–Hochberg (method = 'BH') and export a per-gene table (gene ID, log2FC, P-value, adjusted P-value) to covid19_deg_table.csv. This DEG table is the explicit seed/P-value input for the five active subnetwork detection methods and SEMtree-based perturbation analyses; convert gene identifiers (e.g., Entrez to symbols) with org.Hs.eg.db mapIds when the downstream graph uses SYMBOL labels.

## Related tools

- **limma** (Empirical Bayes gene-wise linear model fitting and moderated statistics for the critical-vs-non-critical differential expression contrast)
- **SEMdata** (Supplies the benchmark GSE172114 COVID-19 RNA-seq expression matrix and group labels shipped with the package) — https://github.com/fernandoPalluzzi/SEMdata
- **org.Hs.eg.db** (Gene ID conversion (e.g., Entrez to gene symbols via mapIds) so the DEG table matches downstream graph node labels)
- **R** (Runtime environment in which limma, SEMdata, and SEMgraph are executed) — https://github.com/fernandoPalluzzi/SEMgraph
- **SEMgraph** (Downstream consumer of the exported DEG P-value/seed table for active subnetwork detection and SEMtree analyses) — https://github.com/fernandoPalluzzi/SEMgraph

## Examples

```
design <- model.matrix(~ group); fit <- eBayes(lmFit(expr, design)); deg <- topTable(fit, number = Inf, adjust.method = 'BH'); write.csv(deg, 'covid19_deg_table.csv')
```

## Evaluation signals

- The exported covid19_deg_table.csv contains one row per gene with BH-adjusted P-values within [0,1].
- The table is loadable as the P-value/seed input for the active subnetwork detection step (five state-of-the-art methods).
- Sample counts match the benchmark: 46 critical cases and 23 non-critical controls from GSE172114.
- The design matrix encodes the critical-vs-non-critical contrast, and per-gene output includes moderated t-statistics, log2 fold changes, and raw P-values.
- Gene identifiers are consistent with downstream graph node keys after org.Hs.eg.db conversion.

## Limitations

- Multiple testing adjustment is by Benjamini–Hochberg across all genes; other correction schemes are not part of this workflow as documented.
- The workflow is documented only for the two-group benchmark design (critical vs non-critical); applicability to other designs is not established in context.
- Quality of downstream SEMtree/subnetwork results depends on the DEG P-value table, and the article's discussion section text is absent, limiting stated caveats.

## Evidence

- [methods] limma EB design: [evidence span withheld — anchor status: not_found]
- [methods] BH adjustment: [evidence span withheld — anchor status: not_found]
- [readme] group sizes: [evidence span withheld — anchor status: not_found]
- [abstract] seed gene usage: "from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods."
- [methods] gene ID conversion: "the use of the R package **org.Hs.eg.db** in the Bioconductor project, for gene ID conversion"
