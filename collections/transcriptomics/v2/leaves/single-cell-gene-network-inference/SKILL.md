---
name: single-cell-gene-network-inference
description: Use when the input is a single-cell read count matrix (e.g., Smart-seq2
  or GEO deposits like GSE114727, GSE75748, Tabula Muris) and the goal is to infer
  an undirected, sparse gene co-expression network robust to excess zero counts/dropout.
when_to_use_negative:
- 'Data are bulk RNA-seq or already dense low-sparsity: bulk-network methods may be
  appropriate and imputed single-cell data can still be much sparser than bulk, degrading
  bulk-data network methods'
- The goal is directed regulatory edges or distinguishing regulatory, co-regulatory,
  and functionally coordinated edge types — scLink's edges are undirected statistical
  dependencies and expression data alone cannot separate these types
- The gene set is very large without prior exploration — imputation can be time-consuming,
  and scLink's authors suggest dividing genes into major modules and applying scLink
  per module instead
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scLink
  role: 'Core R package: sclink_norm for 10^6 library-size normalization and gene
    filtering, sclink_cor for robust correlation, sclink_net for sparse network construction
    with lda = seq(0.5, 0.1, -0.05) and BIC model selection'
  repo: https://github.com/Vivianstats/scLink
- name: glasso
  role: Graphical-lasso baseline run on standard Pearson correlations of unfiltered
    data (and as glasso-r/glasso-f ablation variants using QUIC or 70%-zero cell filtering)
- name: PIDC
  role: Partial information decomposition baseline for network comparison against
    scLink
- name: GENIE3
  role: Tree-ensemble regression baseline that decomposes network inference into p
    regression problems for comparison against scLink
- name: ldstatsHD
  role: Generates block-diagonal ground-truth connectivity matrices with power-law
    and hub-based topologies for simulated Gaussian expression data
- name: QUIC
  role: 'Fits the glasso-r variant: penalized graphical model on scLink''s robust
    correlation matrix with constant unit penalty weights'
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:d6afc17a402f6b43e757271c1a891c286b830f7da59b72bba9bb9ae2d8b3c082
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - GENIE3
  - PIDC
  - QUIC
  - R
  - glasso
  - ldstatsHD
  - parallel
  - scLink
  - sclink_net
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# single-cell-gene-network-inference

## Summary

Infer sparse gene co-expression networks from sparse single-cell RNA-seq count data using scLink, which combines a robust correlation estimator with a penalized Gaussian graphical model with data-adaptive edge penalty weights, and benchmark against glasso, PIDC, GENIE3, and thresholded Pearson/Spearman correlation.

## When to use

Use when the input is a single-cell read count matrix (e.g., Smart-seq2 or GEO deposits like GSE114727, GSE75748, Tabula Muris) and the goal is to infer an undirected, sparse gene co-expression network robust to excess zero counts/dropout.

## When NOT to use

- Data are bulk RNA-seq or already dense low-sparsity: bulk-network methods may be appropriate and imputed single-cell data can still be much sparser than bulk, degrading bulk-data network methods
- The goal is directed regulatory edges or distinguishing regulatory, co-regulatory, and functionally coordinated edge types — scLink's edges are undirected statistical dependencies and expression data alone cannot separate these types
- The gene set is very large without prior exploration — imputation can be time-consuming, and scLink's authors suggest dividing genes into major modules and applying scLink per module instead

## Inputs

- Raw single-cell read/gene count matrix (cells x genes), e.g., from GEO GSE114727, GSE75748, or Tabula Muris Smart-seq2 cell-type matrices
- List of target genes (optionally top 500 highly expressed genes selected by sclink_norm filter.genes = TRUE)
- For simulation: per-gene means and standard deviations estimated from a count matrix, plus ldstatsHD block-diagonal connectivity matrices with power-law or hub-based topologies
- For validation: pooled TF–gene ChIP-seq reference set from ENCODE, ChEA, and Harmonizome

## Outputs

- Sparse undirected gene co-expression network (adjacency matrix) from sclink_net with edges at most 5% of all possible gene pairs
- Robust co-expression/correlation matrix from sclink_cor
- Evaluation table of AUPRC and AUROC per method per condition (e.g., simulation_auprc_auroc.csv, figure_s2_auprc_auroc_comparison.csv)
- TF–gene edge overlap table per cell type (e.g., tf_edge_overlap_tabula_muris.csv)

## How to apply

Preprocess raw counts with sclink_norm using scale.factor = 1e6 (library size 10^6 normalization) and, optionally, filter.genes = TRUE with n = 500 to retain the 500 genes with largest average expression values; for large gene lists, first explore correlation structure with sclink_cor before network inference. Run sclink_net over a candidate regularization grid lda = seq(0.5, 0.1, -0.05) (ncores can be 1), select the model by minimum BIC from the summary element, and restrict the final network to at most 5% of all possible edges. Compare inferred adjacency against ground truth (simulated power-law or hub-based topologies via ldstatsHD) using AUPRC and AUROC, where scLink has been shown to outperform glasso, PIDC, GENIE3, and thresholded Pearson/Spearman correlation. For real-data validation, compute the proportion of inferred TF–gene edges present in a pooled ChIP-seq reference (ENCODE, ChEA, Harmonizome) built from ~310 transcription factors.

## Related tools

- **scLink** (Core R package: sclink_norm for 10^6 library-size normalization and gene filtering, sclink_cor for robust correlation, sclink_net for sparse network construction with lda = seq(0.5, 0.1, -0.05) and BIC model selection) — https://github.com/Vivianstats/scLink
- **glasso** (Graphical-lasso baseline run on standard Pearson correlations of unfiltered data (and as glasso-r/glasso-f ablation variants using QUIC or 70%-zero cell filtering))
- **PIDC** (Partial information decomposition baseline for network comparison against scLink)
- **GENIE3** (Tree-ensemble regression baseline that decomposes network inference into p regression problems for comparison against scLink)
- **ldstatsHD** (Generates block-diagonal ground-truth connectivity matrices with power-law and hub-based topologies for simulated Gaussian expression data)
- **QUIC** (Fits the glasso-r variant: penalized graphical model on scLink's robust correlation matrix with constant unit penalty weights)

## Examples

```
dat.norm <- sclink_norm(counts, scale.factor = 1e6, filter.genes = TRUE, n = 500); net <- sclink_net(dat.norm, ncores = 1, lda = seq(0.5, 0.1, -0.05))
```

## Evaluation signals

- On simulated power-law and hub-based networks (50 repeats, dropout rho in {0.07, 0.10, 0.13, 0.16} plus no-extra-sparsity control), scLink should attain the best AUPRC per condition
- In the ablation table, glasso-f (filtering cells with >70% zero counts) should not effectively improve over plain glasso, and scLink should show higher AUPRC but slightly lower AUROC than glasso-r
- On Tabula Muris, the 59-row TF–gene overlap table should show scLink's median ChIP-seq edge overlap of approximately 59.3% (range 15.6%–89.5%), with scLink and PIDC above Pearson, Spearman, glasso-f, and glasso-r
- Network sparsity check: sclink_net output restricted to at most 5% of all possible edges, with model chosen by minimum BIC from the summary element
- Schema check: preprocessing uses scale.factor = 1e6 and retains exactly n = 500 top-expressed genes when filter.genes = TRUE

## Limitations

- Edges identified by scLink are based on statistical dependencies and do not have directions; regulatory, co-regulatory, and functionally coordinated edge types cannot be distinguished using only expression data
- Imputed data may still be much sparser than bulk data, and imputation can be time-consuming depending on cell number
- No software version specified for the scLink R package, and no validation code/data availability beyond the GitHub repository for reproducing real-data analyses

## Evidence

- [intro] scLink uses a penalized likelihood approach to identify relatively sparse gene networks in a data-adaptive manner, adjusting the penalty strength on each edge based on the observed co-expression: "scLink uses a penalized likelihood approach to identify relatively sparse gene networks in a data-adaptive manner, adjusting the penalty strength on each edge"
- [methods] normalize the read count matrix with a library size of 10^6 and only keep the 500 genes in `genes` for downstream analysis: "normalize the read count matrix with a library size of $10^6$ and only keep the 500 genes in `genes` for downstream analysis"
- [methods] use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network: "use the function `sclink_net` to calculate the robust correlation matrix and identifed sparse co-expression network of scLink"
- [discussion] the edges identified by scLink are based on statistical dependencies and do not have directions: "the edges identified by scLink are based on statistical dependencies and do not have directions"
- [readme] The scLink R package is available at https://github.com/Vivianstats/scLink.: [evidence span withheld — anchor status: not_found]
