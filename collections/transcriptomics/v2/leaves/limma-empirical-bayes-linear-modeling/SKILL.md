---
name: limma-empirical-bayes-linear-modeling
description: Use when a normalized RNA-seq expression matrix with two-group labels
  (e.g., 46 critical vs 23 non-critical COVID-19 samples from GSE172114) needs gene-level
  DEG P-values to seed active subnetwork detection or SEMtree perturbation analysis.
when_to_use_negative:
- The input is raw (unnormalized) counts or single-sample data with no group design
  — limma's linear model requires a normalized matrix and a defined contrast between
  groups
- The goal is pathway-level or network-level perturbation testing (use SEMrun/SEMpath
  node, edge, or path perturbation models instead of gene-level DE testing)
- The downstream method needs seed genes only (a curated disease-gene list) rather
  than genome-wide P-values
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: limma
  role: Empirical Bayes gene-wise linear modeling and BH multiple-testing adjustment
    producing the DEG P-values
- name: SEMdata
  role: Ships the normalized GSE172114 COVID-19 RNA-seq benchmark data (46 critical
    vs 23 non-critical cases) used as input
  repo: https://github.com/fernandoPalluzzi/SEMdata
- name: SEMgraph
  role: Consumes the gene P-values as seeds for active subnetwork detection and SEMtree/SEMrun
    perturbation analyses
  repo: https://github.com/fernandoPalluzzi/SEMgraph
- name: org.Hs.eg.db
  role: Gene ID conversion (Entrez to gene symbols) to keep the DEG table aligned
    with graph node labels
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad377
    title: 'SEMtree: tree-based structure learning methods with structural equation
      models'
  source_content_hash: sha256:2678aa14e35f6306b41f39db0e25a13fc0f673f7c457fee4a847eb8cf1b6f6ce
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
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# limma-empirical-bayes-linear-modeling

## Summary

Compute gene-level differential expression P-values on the GSE172114 COVID-19 RNA-seq benchmark (46 critical vs 23 non-critical cases) using limma's empirical Bayes moderated linear models with Benjamini–Hochberg adjustment; the resulting per-gene table supplies the P-values/seeds consumed by SEMtree's active subnetwork detection and downstream SEM analyses.

## When to use

Use when a normalized RNA-seq expression matrix with two-group labels (e.g., 46 critical vs 23 non-critical COVID-19 samples from GSE172114) needs gene-level DEG P-values to seed active subnetwork detection or SEMtree perturbation analysis.

## When NOT to use

- The input is raw (unnormalized) counts or single-sample data with no group design — limma's linear model requires a normalized matrix and a defined contrast between groups
- The goal is pathway-level or network-level perturbation testing (use SEMrun/SEMpath node, edge, or path perturbation models instead of gene-level DE testing)
- The downstream method needs seed genes only (a curated disease-gene list) rather than genome-wide P-values

## Inputs

- Normalized COVID-19 RNA-seq expression matrix (GSE172114, 46 critical + 23 non-critical samples) from the SEMdata package
- Binary group vector labeling critical vs non-critical samples per gene column
- Gene ID mapping resources (org.Hs.eg.db) to align gene IDs with KEGG/reference graph node IDs

## Outputs

- covid19_deg_table.csv with one row per gene: gene ID, log2 fold change, raw P-value, BH-adjusted P-value
- Gene P-value/seed list feeding the five active subnetwork detection methods and SEMtree perturbation models

## How to apply

Load the normalized expression matrix and group labels (46 critical cases vs 23 non-critical controls, GSE172114) from the SEMdata benchmark package. Build a limma design matrix encoding the critical-vs-non-critical contrast, then fit gene-wise linear models with lmFit and apply the contrast of interest. Apply limma's empirical Bayes variance moderation (eBayes) to extract per-gene moderated t-statistics, log2 fold changes, and raw P-values, and adjust for multiple testing across all genes with Benjamini–Hochberg (method = 'BH'). Write the per-gene DEG table (gene ID, log2FC, P-value, adjusted P-value) to covid19_deg_table.csv; this table is the P-value/seed input that the five active subnetwork detection methods and SEMtree-based analyses consume, so gene IDs must be kept consistent with the reference graph (Entrez IDs, convertible to symbols via org.Hs.eg.db).

## Related tools

- **limma** (Empirical Bayes gene-wise linear modeling and BH multiple-testing adjustment producing the DEG P-values)
- **SEMdata** (Ships the normalized GSE172114 COVID-19 RNA-seq benchmark data (46 critical vs 23 non-critical cases) used as input) — https://github.com/fernandoPalluzzi/SEMdata
- **SEMgraph** (Consumes the gene P-values as seeds for active subnetwork detection and SEMtree/SEMrun perturbation analyses) — https://github.com/fernandoPalluzzi/SEMgraph
- **org.Hs.eg.db** (Gene ID conversion (Entrez to gene symbols) to keep the DEG table aligned with graph node labels)

## Examples

```
fit <- eBayes(contrasts.fit(lmFit(expr, design), makeContrasts(critical - noncritical, levels = design))); deg <- topTable(fit, number = Inf, adjust.method = 'BH'); write.csv(deg, 'covid19_deg_table.csv')
```

## Evaluation signals

- Exported covid19_deg_table.csv contains exactly one row per gene in the expression matrix, with columns gene ID, log2FC, P-value, and BH-adjusted P-value
- All BH-adjusted P-values fall within [0, 1] and are monotonically non-decreasing relative to raw P-values
- The fitted design matrix uses all 46 critical and 23 non-critical samples with the critical-vs-non-critical contrast
- The table loads without error as the P-value/seed input for the active subnetwork detection step, with gene IDs matching reference-graph node IDs (Entrez, symbol-convertible via org.Hs.eg.db)

## Limitations

- The DEG P-values are computed on normalized RNA-seq data; the article applies a nonparanormal transform (transformData) downstream to relax normality for SEM fitting, so raw limma outputs should not be fed directly into SEMrun without that transformation step
- Gene IDs in the DEG table must match the reference interactome node IDs (e.g., the KEGG network with 5007 nodes/44755 edges); mismatched identifier types will silently drop seeds
- The article's discussion section is absent from the source document, so caveats about the benchmark comparison are not available here

## Evidence

- [abstract] from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods.: "from a list of seed (i.e. disease) genes or gene P-values, the perturbed modules with undirected edges are generated with five state-of-the-art active subnetwork detection methods."
- [abstract] We applied SEMtree() to both Coronavirus disease (COVID-19) RNA-seq dataset (GEO accession: GSE172114) and simulated datasets with various differential expression patterns.: "We applied SEMtree() to both Coronavirus disease (COVID-19) RNA-seq dataset (GEO accession: GSE172114) and simulated datasets with various differential expression patterns."
- [methods] the use of the R package **org.Hs.eg.db** in the Bioconductor project, for gene ID conversion: "the use of the R package **org.Hs.eg.db** in the Bioconductor project, for gene ID conversion"
- [readme] **COVID-19** RNA-seq dataset of 46 critical and 23 non-critical COVID-19 cases in young patients, from Carapito R. *et al.*, 2022 (GEO accession:: [evidence span withheld — anchor status: external_doc]
