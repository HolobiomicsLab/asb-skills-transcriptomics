---
name: simulated-gene-network-topology-generation
description: 'Use when benchmarking gene co-expression network methods (scLink, glasso,
  glasso-r, glasso-f) against ground truth: single-cell count data must be simulated
  over a power-law or hub-based topology to provide known true edges for AUPRC/AUROC
  evaluation.'
when_to_use_negative:
- Input is real experimental single-cell data without a known ground-truth adjacency
  — AUPRC/AUROC against true edges cannot be computed, so the simulation benchmark
  design does not apply
- Input is bulk RNA-seq or already-imputed expression data — the article notes imputed
  single-cell data can remain much sparser than bulk data and imputation itself can
  be time-consuming, changing the inference problem scLink's simulation framework
  targets
- The goal is imputation of dropouts (e.g. via scImpute) rather than network inference
  benchmarking — this skill generates labeled simulation data for evaluating sparse
  co-expression networks
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scLink
  role: Fits the data-adaptive penalized Gaussian graphical model on simulated counts
    via sclink_net, with BIC selection over lda = seq(0.5, 0.1, -0.05); also supplies
    the robust correlation matrix used to build the glasso-r ablation
  repo: https://github.com/Vivianstats/scLink
- name: R
  role: Implementation environment for the simulation, the scLink/glasso variant fits,
    and AUPRC/AUROC computation
- name: QUIC
  role: 'Solves the glasso-r ablation: a penalized graphical model run on scLink''s
    robust correlation matrix with constant unit penalty weights on every edge'
- name: parallel
  role: Supports reproducible single-core execution of sclink_net (ncores=1) during
    the ablation comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.11.006
    title: 'scLink: Inferring Sparse Gene Co-Expression Networks from Single-Cell
      Expression Data'
  source_content_hash: sha256:b6ee1618b2c256fab39dd290507db6408e03dcf8afa82f81592b341bc9a7719d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Vivianstats/scLink
  tools:
  - QUIC
  - R
  - parallel
  - scLink
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# simulated-gene-network-topology-generation

## Summary

Generates simulated single-cell expression count data over known gene network topologies (power-law and hub-based), following the scLink paper's simulation framework, so that co-expression network inference methods can be scored against a known true adjacency.

## When to use

Use when benchmarking gene co-expression network methods (scLink, glasso, glasso-r, glasso-f) against ground truth: single-cell count data must be simulated over a power-law or hub-based topology to provide known true edges for AUPRC/AUROC evaluation.

## When NOT to use

- Input is real experimental single-cell data without a known ground-truth adjacency — AUPRC/AUROC against true edges cannot be computed, so the simulation benchmark design does not apply
- Input is bulk RNA-seq or already-imputed expression data — the article notes imputed single-cell data can remain much sparser than bulk data and imputation itself can be time-consuming, changing the inference problem scLink's simulation framework targets
- The goal is imputation of dropouts (e.g. via scImpute) rather than network inference benchmarking — this skill generates labeled simulation data for evaluating sparse co-expression networks

## Inputs

- Simulation specification: network type (power-law or hub-based) and topology parameters following the paper's simulation framework
- Simulated single-cell read count matrix (genes x cells) with known true adjacency matrix
- scLink gene list (e.g. 500 genes as in the paper's downstream analysis)

## Outputs

- Simulated expression count matrix paired with its true simulated adjacency matrix
- Fitted networks: scLink net (BIC-selected), glasso-r, glasso-f, and plain glasso solutions
- Method-by-network-type comparison table of AUPRC and AUROC (figure_s2_auprc_auroc_comparison.csv)

## How to apply

Simulate single-cell expression data over power-law and hub-based topologies following the scLink paper's simulation framework, retaining the true simulated adjacency as evaluation ground truth. On each simulated dataset, fit scLink via sclink_net with ncores=1 and candidate regularization parameters lda = seq(0.5, 0.1, -0.05), selecting the final model by BIC from the returned summary element. Run the ablations: glasso-r fits a QUIC-based penalized graphical model on scLink's robust correlation matrix with constant unit penalty weights on every edge, and glasso-f removes cells with more than 70% zero counts before applying plain graphical lasso on the filtered data. Fit plain glasso on standard correlations of the unfiltered data as the baseline, then compute AUPRC and AUROC for each method-by-network-type combination against the true adjacency and write the comparison table (e.g. figure_s2_auprc_auroc_comparison.csv). The rationale is to isolate whether scLink's data-adaptive penalty weights, rather than its robust correlation or cell filtering, drive the accuracy gain.

## Related tools

- **scLink** (Fits the data-adaptive penalized Gaussian graphical model on simulated counts via sclink_net, with BIC selection over lda = seq(0.5, 0.1, -0.05); also supplies the robust correlation matrix used to build the glasso-r ablation) — https://github.com/Vivianstats/scLink
- **R** (Implementation environment for the simulation, the scLink/glasso variant fits, and AUPRC/AUROC computation)
- **QUIC** (Solves the glasso-r ablation: a penalized graphical model run on scLink's robust correlation matrix with constant unit penalty weights on every edge)
- **parallel** (Supports reproducible single-core execution of sclink_net (ncores=1) during the ablation comparison)

## Examples

```
net <- sclink_net(expr = expr.norm, genes = genes, lda = seq(0.5, 0.1, -0.05), ncores = 1); best <- net$summary[[which.min(sapply(net$summary, function(s) s$BIC))]]
```

## Evaluation signals

- glasso-f (cell filtering, dropping cells with >70% zero counts) shows no effective AUPRC/AUROC improvement over plain glasso on the simulated power-law and hub networks
- scLink attains higher AUPRC and slightly lower AUROC than glasso-r, confirming the adaptive penalty — not the robust correlation alone — drives the gain
- scLink model selection uses BIC from the summary element of sclink_net output across lda = seq(0.5, 0.1, -0.05)
- AUPRC and AUROC are computed for every method x network type cell against the exact simulated true adjacency, and the table is written to figure_s2_auprc_auroc_comparison.csv
- The 500-gene analysis scope is retained (normalize with library size of 10^6 and keep only the 500 genes in `genes` for downstream analysis)

## Limitations

- Edges identified by scLink are undirected statistical dependencies — the simulation cannot be used to distinguish regulatory, co-regulatory, and functionally coordinated edge types from expression alone
- Accuracy conclusions hold only within the paper's simulated power-law and hub-based topologies; no software version of the scLink R package is specified
- The comparison depends on the fixed candidate grid lda = seq(0.5, 0.1, -0.05) and ncores=1; different grids or parallelization may change runtime and selected models

## Evidence

- [abstract] We use both simulation and real data studies to demonstrate the advantages of scLink and its ability to improve single-cell gene network analysis.: "We use both simulation and real data studies to demonstrate the advantages of scLink and its ability to improve single-cell gene network analysis."
- [intro] scLink adapts the Gaussian graphical model [29] to distinguish direct associations between genes from indirect ones and leads to sparse networks: "scLink adapts the Gaussian graphical model [29] to distinguish direct associations between genes from indirect ones and leads to sparse networks"
- [intro] scLink uses a penalized likelihood approach to identify relatively sparse gene networks in a data-adaptive manner, adjusting the penalty strength on each edge based on the observed co-expression: "scLink uses a penalized likelihood approach to identify relatively sparse gene networks in a data-adaptive manner, adjusting the penalty strength on each edge"
- [readme] For detailed usage, please refer to the package manual: [evidence span withheld — anchor status: external_doc]
