---
name: network-regularized-enrichment-scoring
description: Use when scoring differential-expression p-values against a large, redundant
  pathway database (KEGG, Reactome, GO) and you want enrichment estimates that exploit
  inter-pathway relations via a pathway-similarity network.
when_to_use_negative:
- The input is a small, non-redundant pathway database where inter-pathway dependencies
  are negligible and simpler over-representation methods suffice
- You lack a pathway similarity measure or network structure to supply as term_network,
  so the regularized model has no inter-pathway information to exploit
- The task is generic GO/ORA visualization of non-pareg enrichment results rather
  than fitting a network-regularized model
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0085
tools:
- name: pareg
  role: 'Core method: regularized generalized linear model that computes pathway enrichment
    scores with network regularization on a pathway similarity matrix'
  repo: https://github.com/cbg-ethz/pareg
- name: Snakemake
  role: Orchestrates the synthetic-benchmark workflows (grid of noise levels eta and
    similarity factors rho) shipped in the pareg repository
  repo: https://github.com/cbg-ethz/pareg
- name: enrichplot
  role: Receives pareg fits via as_enrichplot_object for pathway-network visualization
    of enrichment results
  repo: https://github.com/YuLab-SMU/enrichplot
- name: tidyverse
  role: Data manipulation and scoring of pareg enrichment scores against known active
    terms in R
provenance:
  source_task_ids:
  - task_001
  - task_008
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad522
    title: Coherent pathway enrichment estimation by modeling inter-pathway dependencies
      using regularized regression
  source_content_hash: sha256:9be45d9c995ea2f30fdee88c6d7369b04be8dc71d899b3931cfe7e61858228ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cbg-ethz/pareg
  tools:
  - R
  - Snakemake
  - enrichplot
  - pareg
  - tidyverse
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# network-regularized-enrichment-scoring

## Summary

Estimates pathway enrichment scores with pareg, a regularized generalized linear model that incorporates inter-pathway dependencies (e.g. shared genes, Jaccard similarity) directly in the enrichment computation, making it more robust to noise than competing methods like MGSA, FET, topGO, and blitzGSEA.

## When to use

Use when scoring differential-expression p-values against a large, redundant pathway database (KEGG, Reactome, GO) and you want enrichment estimates that exploit inter-pathway relations via a pathway-similarity network.

## When NOT to use

- The input is a small, non-redundant pathway database where inter-pathway dependencies are negligible and simpler over-representation methods suffice
- You lack a pathway similarity measure or network structure to supply as term_network, so the regularized model has no inter-pathway information to exploit
- The task is generic GO/ORA visualization of non-pareg enrichment results rather than fitting a network-regularized model

## Inputs

- Gene-level differential-expression p-value table (df_study) from a synthetic or real study
- Pathway database as a term–gene data frame (df_terms)
- Pathway similarity matrix, e.g. Jaccard via compute_term_similarities(df_terms, similarity_function = jaccard), or a pre-computed matrix from pathway_similarities
- Known active (true-positive) pathway list for benchmark scoring

## Outputs

- Ranked pathway-level enrichment scores from the pareg fit
- Precision-recall AUC and precision-at-80%-recall benchmark tables (e.g. benchmark_pr_auc.csv, similarity_benchmark.csv)
- enrichplot object (via as_enrichplot_object) for pathway network visualization

## How to apply

Install pareg from Bioconductor via BiocManager::install("pareg"), then build (or load the pre-computed) pathway-similarity matrix, e.g. compute_term_similarities(df_terms, similarity_function = jaccard) for a term table. Prepare the study as a data frame of gene-level differential-expression p-values, and run pareg on df_study with network_param = 1 and term_network = the Jaccard (or alternative) similarity matrix; the network regularization term makes this a modular enrichment analysis tool. To evaluate on synthetic benchmarks, draw true-positive active pathways with similarity_sample(mat_similarities, 5), assign DE genes Beta(0.1, 1) p-values and background genes Uniform(0, 1) p-values, and score ranked enrichment scores against the known active terms using PR-AUC and precision-at-80%-recall. Orchestrate multi-setting runs (noise levels eta = 0, 0.25, 0.5, 0.75; similarity factors rho = 0, 0.5, 1) with the Snakemake workflows in the pareg repository's inst/scripts/, and convert fits to enrichplot objects for pathway-network visualization.

## Related tools

- **pareg** (Core method: regularized generalized linear model that computes pathway enrichment scores with network regularization on a pathway similarity matrix) — https://github.com/cbg-ethz/pareg
- **Snakemake** (Orchestrates the synthetic-benchmark workflows (grid of noise levels eta and similarity factors rho) shipped in the pareg repository) — https://github.com/cbg-ethz/pareg
- **enrichplot** (Receives pareg fits via as_enrichplot_object for pathway-network visualization of enrichment results) — https://github.com/YuLab-SMU/enrichplot
- **tidyverse** (Data manipulation and scoring of pareg enrichment scores against known active terms in R)

## Examples

```
compute_term_similarities(df_terms, similarity_function = jaccard); active_terms <- similarity_sample(mat_similarities, 5)
```

## Evaluation signals

- The pareg fit returns ranked enrichment scores for all pathways that can be evaluated against the known active terms from similarity_sample
- In the synthetic benchmark (eta = 0.25), pareg with the Jaccard similarity matrix reaches a median precision of ~94% at 80% recall, vs MGSA 37%, FET 12%, topGO 5%
- pareg achieves the highest mean PR-AUC among all benchmarked methods across the (eta, rho) grid, reproducing Fig. 1a/1c trends
- PR-AUC and precision-at-80%-recall values per method are aggregated into a summary table (e.g. benchmark_pr_auc.csv) matching reported replicate variability

## Limitations

- Performance depends on the chosen pathway similarity measure: similarity measures differ substantially (as observed on GO Biological Process), so replacing the Jaccard matrix with an alternative requires re-benchmarking against the Jaccard baseline
- Benchmark claims rest on synthetic data with Beta(0.1, 1) DE p-values and Uniform background p-values; real-study behavior (e.g. TCGA breast cancer) is exploratory rather than a controlled ground-truth validation
- pareg classifies as a modular enrichment analysis tool, so it inherits the assumptions of network-regularization approaches rather than acting as a strict statistical significance test

## Evidence

- [abstract] pareg is a regularized GLM enrichment method: "develop a novel gene set enrichment method, called pareg, which is based on a regularized generalized linear model"
- [methods] Network regularization uses pathway similarity matrix: "The network regularization term is based on a pathway similarity matrix (e.g., defined by Jaccard similarity) and thus classifies this method as a modular enrichment analysis tool"
- [methods] pareg fits regressed on DE p-values: "It estimates pathway enrichment scores by regressing differential expression p-values"
- [readme] README describes the approach: "Pathway enrichment computations using a regularized regression approach to incorporate inter-pathway relations in the statistical model."
- [methods] Synthetic benchmark p-value distributions: "The p-values of genes considered to be differentially expressed are sampled from a Beta distribution centered at $0$. The p-values for all other genes are drawn from a Uniform distribution."
