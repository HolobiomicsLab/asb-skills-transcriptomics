---
name: baseline-confounder-removal-benchmarking
description: Use when evaluating whether a confounder-removal method for single-cell
  RNA-seq outperforms PCA, LMM, RUV, and scLVM baselines on simulated data with known
  latent confounders and cell labels.
when_to_use_negative:
- Input is real data with unknown ground-truth cell labels — ARI against true labels
  cannot be computed, so this simulation benchmark design does not apply
- Evaluating scPLS on downstream analyses other than clustering (e.g. differential
  expression, lineage reconstruction), which the authors state is beyond the scope
  of the present study
- Modeling raw count data — the authors note no count-based model is provided and
  extending the framework to counts is left as future work
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scPLS
  role: Method under test; infers and removes confounding factors via partial least
    squares regression on control and target genes
  repo: http://chenmengjie.github.io/Citrus/
- name: Citrus
  role: Toolkit hosting scPLS and the simulation utilities used to generate benchmark
    datasets
  repo: http://chenmengjie.github.io/Citrus/
- name: Rcpp
  role: Required dependency; Citrus is written in C++ with R wrappers via Rcpp
- name: RcppArmadillo
  role: Required dependency for Citrus installation and the block-wise EM implementation
- name: genlasso
  role: R package dependency of Citrus used in the scPLS pipeline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:e906c413473197fbcaca57e049a3ee2bb2ee15e0ce3a60dc5a70c8a8b2fc58c9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Citrus
  - Rcpp
  - RcppArmadillo
  - genlasso
  - scPLS
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# baseline-confounder-removal-benchmarking

## Summary

Benchmark scPLS confounder removal against baseline methods (PCA, LMM, RUV, scLVM) on simulated scRNAseq data by sweeping proportion of differential expression (PDE) 20-100% across two scenarios with/without dropout, scoring corrected data by k-means Adjusted Rand Index and PVE of estimated latent factors.

## When to use

Use when evaluating whether a confounder-removal method for single-cell RNA-seq outperforms PCA, LMM, RUV, and scLVM baselines on simulated data with known latent confounders and cell labels.

## When NOT to use

- Input is real data with unknown ground-truth cell labels — ARI against true labels cannot be computed, so this simulation benchmark design does not apply
- Evaluating scPLS on downstream analyses other than clustering (e.g. differential expression, lineage reconstruction), which the authors state is beyond the scope of the present study
- Modeling raw count data — the authors note no count-based model is provided and extending the framework to counts is left as future work

## Inputs

- Simulated scRNA-seq count/expression matrices with known latent confounders (scenario I and II, PDE 20-100%, with and without dropout)
- Control-gene set from CellCycleData (Citrus resource)
- True simulated cell labels for ARI computation
- Citrus simulation utilities

## Outputs

- ARI table comparing k-means clustering accuracy of scPLS vs PCA, LMM, RUV, scLVM across PDE levels, scenarios, and dropout conditions
- PVE-accuracy panel quantifying accuracy of estimated confounding factors per method
- simulation_evaluation_results.csv combining ARI and PVE results

## How to apply

Simulate single-cell RNA-seq expression with known latent confounders under scenario I and scenario II, sweeping the proportion of differential expression (PDE) from 20% to 100% in both standard and dropout settings, using the Citrus package's simulation utilities together with the CellCycleData control-gene resource. For each dataset, infer and remove confounding factors with scPLS (scPLS(Target, Control, ...), Rcpp/RcppArmadillo block-wise EM) and with the four baselines: PCA, linear mixed model (LMM), RUV, and scLVM. Apply k-means clustering to each corrected expression matrix and compute the Adjusted Rand Index against the true simulated cell labels per scenario, dropout condition, and PDE level. Also compute the proportion of variance explained (PVE) of the estimated confounding factors to quantify latent-factor estimation accuracy. Assemble the ARI comparison and PVE-accuracy panel into a single results table (e.g. simulation_evaluation_results.csv). Validation: the benchmark is considered reproduced if scPLS attains comparable or higher ARI and accurate PVE across the PDE sweep, matching the paper's finding that scPLS-corrected data yield the highest ARI.

## Related tools

- **scPLS** (Method under test; infers and removes confounding factors via partial least squares regression on control and target genes) — http://chenmengjie.github.io/Citrus/
- **Citrus** (Toolkit hosting scPLS and the simulation utilities used to generate benchmark datasets) — http://chenmengjie.github.io/Citrus/
- **Rcpp** (Required dependency; Citrus is written in C++ with R wrappers via Rcpp)
- **RcppArmadillo** (Required dependency for Citrus installation and the block-wise EM implementation)
- **genlasso** (R package dependency of Citrus used in the scPLS pipeline)

## Examples

```
install.packages(c("Rcpp","RcppArmadillo","genlasso","gplots")); devtools::install_github("ChenMengjie/Citrus"); corrected <- scPLS(Target, Control)
```

## Evaluation signals

- scPLS-corrected data yield the highest Adjusted Rand Index in k-means clustering versus PCA, LMM, RUV, and scLVM across the PDE 20-100% sweep in both scenarios
- The pattern holds both with and without dropout events
- PVE of estimated confounding factors is comparable or higher for scPLS, quantifying latent-factor estimation accuracy
- All methods' ARI and PVE values are assembled into simulation_evaluation_results.csv matching the paper's simulation finding

## Limitations

- scPLS can only infer biological effects (Lambda_u u_i) but not the biological factors (u_i) because of model identifiability
- The 'best' confounding effects removing method may vary depending on the downstream analysis; performance of scPLS in other analysis settings has not been evaluated
- No count-based model is provided; extending the framework to modeling count data is left as future work
- No software version numbers are stated for the tools used

## Evidence

- [abstract] Finding: scPLS-corrected data yield highest ARI: "With extensive simulations and comparisons with other methods, we demonstrate the effectiveness of scPLS."
- [intro] scPLS bridges PCA/LMM and RUV/scLVM subcategories: "scPLS is closely related to and bridges between two existing subcategories of methods for transcriptome analysis"
- [methods] scPLS call signature: "`scPLS(Target, Control, k1, k2, iter, alpha, kappa, diagH, g, h1, h2, c, d, limit, method, penalty, tol, givenpenalty, kfold, Chunk, chunk.size, center)`"
- [readme] Citrus installation dependencies: "**Citrus** relies on the following R packages: **Rcpp**, **RcppArmadillo**, **genlasso**, **gplots**. All packagess are hosted on CRAN."
- [discussion] Caveat on best method depending on downstream analysis: "the “best” confounding effects removing method may vary depending on the question of interest."
