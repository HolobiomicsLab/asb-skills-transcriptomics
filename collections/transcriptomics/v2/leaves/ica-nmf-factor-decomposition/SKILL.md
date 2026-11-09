---
name: ica-nmf-factor-decomposition
description: Use when scRNA-seq gene programs must be discovered by independent component
  analysis or NMF — e.g., as ICA/NMF comparators in a benchmark against SSN gene programs
  scored by GO and PPI enrichment.
when_to_use_negative:
- The goal is cell-type annotation rather than gene program discovery — the Miko scoring
  pipeline or sc-type are the paper's annotation tools, not ICA/NMF factorization.
- The input is already a set of gene programs; this skill generates programs from
  expression data and cannot recover GO/PPI statistics for externally supplied gene
  sets without the getAnnotationPathways enrichment step.
- The expected best-performing method is needed for program discovery — the paper's
  finding is that scale-free SSN achieves superior GO-term recovery and STRING PPI
  enrichment compared to ICA and NMF, so factorization is primarily a baseline.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scMiko
  role: R package supplying getICAGenes, getNMFGenes, consolidateNMF, findNetworkFeatures,
    and getAnnotationPathways used to run the ICA/NMF factorizations and evaluate
    the resulting gene programs
  repo: https://github.com/NMikolajewicz/scMiko
- name: Seurat
  role: Framework on which scMiko is built; provides the Seurat object holding the
    scRNA-seq expression data fed to the factorization functions
  repo: https://satijalab.org/seurat/
- name: scPipeline
  role: Modular Rmarkdown workflow collection (gene program discovery module) that
    packages the SSN/ICA/NMF comparison into reproducible dashboard reports
  repo: https://github.com/NMikolajewicz/scPipeline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-022-04093-2
    title: Multi-level cellular and functional annotation of single-cell transcriptomes
      using scPipeline
  source_content_hash: sha256:1f840e401978e1ba61dbec84328ee141df13d24959096759d4fc53d4e6ade2a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/NMikolajewicz/scMiko
  tools:
  - Seurat
  - scMiko
  - scPipeline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# ica-nmf-factor-decomposition

## Summary

Discovers gene programs from scRNA-seq data by ICA and NMF factorization (scMiko getICAGenes/getNMFGenes, with consolidateNMF to merge redundant NMF factors), and evaluates program quality by GO-term recovery and STRING PPI enrichment as comparator methods to scale-free SSN analysis.

## When to use

Use when scRNA-seq gene programs must be discovered by independent component analysis or NMF — e.g., as ICA/NMF comparators in a benchmark against SSN gene programs scored by GO and PPI enrichment.

## When NOT to use

- The goal is cell-type annotation rather than gene program discovery — the Miko scoring pipeline or sc-type are the paper's annotation tools, not ICA/NMF factorization.
- The input is already a set of gene programs; this skill generates programs from expression data and cannot recover GO/PPI statistics for externally supplied gene sets without the getAnnotationPathways enrichment step.
- The expected best-performing method is needed for program discovery — the paper's finding is that scale-free SSN achieves superior GO-term recovery and STRING PPI enrichment compared to ICA and NMF, so factorization is primarily a baseline.

## Inputs

- Processed Seurat object (normalized scRNA-seq expression matrix, e.g. so.query)
- Feature sets from findNetworkFeatures: expression-selected features (method='expr', min_pct=0.5), HVGs, or deviance-based sets with n_features=2000

## Outputs

- ICA-derived gene programs (gene sets) from getICAGenes
- Consolidated NMF gene programs from getNMFGenes + consolidateNMF
- Per-method GO-term recovery (max column-wise Jaccard similarity) and STRING PPI enrichment statistics
- Benchmark comparison table (ssn_benchmark_results.csv)

## How to apply

Start from the same network feature sets used by the SSN workflow — features selected with findNetworkFeatures using the expression criterion (method='expr', min_pct=0.5), highly-variable genes, or deviance-based sets with n_features=2000. Run ICA factorization with getICAGenes and NMF with getNMFGenes on the Seurat object; because individual NMF factors are often redundant, consolidate them with consolidateNMF before treating them as gene programs. For each resulting program, retrieve GO annotations with getAnnotationPathways and compute GO-term recovery as the maximum column-wise Jaccard similarity between the program gene set and GO terms, then test enrichment of STRING protein-protein interactions. Aggregate per-method recovery and PPI-enrichment statistics into a comparison table (ssn_benchmark_results.csv) so ICA/NMF can be judged against the scale-free SSN run (runSSN with scale_free=TRUE, data_type='pearson', pca_var_explained=0.9, target_purity=0.8), which the paper reports outperforms both factorization methods (Fig. 5c-e).

## Related tools

- **scMiko** (R package supplying getICAGenes, getNMFGenes, consolidateNMF, findNetworkFeatures, and getAnnotationPathways used to run the ICA/NMF factorizations and evaluate the resulting gene programs) — https://github.com/NMikolajewicz/scMiko
- **Seurat** (Framework on which scMiko is built; provides the Seurat object holding the scRNA-seq expression data fed to the factorization functions) — https://satijalab.org/seurat/
- **scPipeline** (Modular Rmarkdown workflow collection (gene program discovery module) that packages the SSN/ICA/NMF comparison into reproducible dashboard reports) — https://github.com/NMikolajewicz/scPipeline

## Examples

```
features_hvg <- findNetworkFeatures(object = so.query, method = "hvg", n_features = 2000); ica_progs <- getICAGenes(object = so.query, features = features_hvg); nmf_progs <- consolidateNMF(getNMFGenes(object = so.query, features = features_hvg))
```

## Evaluation signals

- GO-term recovery per program, computed as the maximum column-wise Jaccard similarity between program gene sets and GO terms from getAnnotationPathways, is aggregated per method
- STRING PPI enrichment tests are performed for every program from each method and aggregated into ssn_benchmark_results.csv
- ICA/NMF are run on the identical feature sets as SSN (n_features=2000; method='expr' with min_pct=0.5) so the method comparison is feature-matched
- NMF factors are consolidated with consolidateNMF before comparison — unconsolidated factors would inflate program counts and distort recovery statistics
- Expected outcome reproduced from the paper: SSN programs (not ICA/NMF) achieve significantly superior GO term recovery and STRING PPI enrichment (Fig. 5c, d)

## Limitations

- Within this benchmark, ICA and NMF are the weaker comparators — scale-free SSN yielded superior GO term recovery and STRING PPI enrichment, so factorization methods should not be assumed best for program discovery.
- NMF requires a consolidation step (consolidateNMF); raw factors are redundant and not directly comparable to SSN modules.
- Program quality assessment depends on GO annotation retrieval (getAnnotationPathways) and the STRING database; recovery metrics are limited to terms/interactions present in those resources.

## Evidence

- [discussion] gene programs identified by SSN yielded superior GO term recovery and enrichment of STRING PPIs compared to ICA and NMF methods: "gene programs identified by SSN yielded superior GO term recovery and enrichment of STRING PPIs compared to ICA and NMF methods"
- [methods] findNetworkFeatures expression-based feature selection call: "features_expr <- findNetworkFeatures(object = so.query, method = "expr", min_pct = 0.5)"
- [other] factorization comparator step from the benchmark task card: [evidence span withheld — anchor status: not_found]
- [readme] scMiko package role from repository README: "scMiko is an **R package** developed for single-cell RNA-seq analysis. It was designed using the [Seurat](https://satijalab.org/seurat/) framework"
