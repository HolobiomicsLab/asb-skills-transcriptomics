---
name: variance-explained-parameterization
description: Use when constructing simulated scRNAseq count matrices whose confounding
  and biological latent factors must each explain a specified proportion of variance
  (PVE) so removal methods can be benchmarked under controlled effect sizes.
when_to_use_negative:
- The goal is to model count data with a dedicated count-based (e.g. negative binomial)
  likelihood — the article leaves extending the framework to modeling count data as
  future work
- Real scRNAseq data are being analyzed rather than benchmark simulations — use scPLS
  inference on the data instead of re-simulating it
- The downstream comparison concerns clustering methods (RaceID, SCUBA, SNN-Cliq,
  ZIFA, t-SNE, SC3) rather than confounding-effect removal benchmarks parameterized
  by PVE
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Citrus
  role: R/C++ toolkit providing the CellCycleData dataset used as the source of gene
    expression profiles and hosting scPLS
  repo: http://chenmengjie.github.io/Citrus/
- name: scPLS
  role: The confounding-effect removal method whose benchmark simulations are generated
    with this PVE-parameterized design
  repo: http://chenmengjie.github.io/Citrus/
- name: Rcpp
  role: R package dependency used to vectorize Poisson count generation in the simulator
- name: RcppArmadillo
  role: R package dependency used with Rcpp for fast vectorized simulation of count
    matrices
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:b1f36e97857afb67bb2f1750a4f97ef384ec25d3cabbafd041dadecd359f3603
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Citrus
  - Rcpp
  - RcppArmadillo
  - scPLS
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Variance-explained parameterization

## Summary

A simulation design skill for parameterizing latent-factor generators by the proportion of variance explained (PVE) each hidden factor contributes, as used to build the scPLS benchmark scRNAseq count matrices from the CellCycleData gene profiles.

## When to use

Use when constructing simulated scRNAseq count matrices whose confounding and biological latent factors must each explain a specified proportion of variance (PVE) so removal methods can be benchmarked under controlled effect sizes.

## When NOT to use

- The goal is to model count data with a dedicated count-based (e.g. negative binomial) likelihood — the article leaves extending the framework to modeling count data as future work
- Real scRNAseq data are being analyzed rather than benchmark simulations — use scPLS inference on the data instead of re-simulating it
- The downstream comparison concerns clustering methods (RaceID, SCUBA, SNN-Cliq, ZIFA, t-SNE, SC3) rather than confounding-effect removal benchmarks parameterized by PVE

## Inputs

- CellCycleData dataset from the Citrus R package (baseline gene expression profiles)
- Simulation specification: 200 cells, 2 equal-sized groups, 50 control genes, 1,000 target genes, 200 DE target genes, kz=2 confounding factors, ku=5 biological factors
- Per-factor PVE settings and scenario definitions (scenario I, scenario II)
- Random seed

## Outputs

- simulated_counts.csv (integer UMI-like count matrix, 200 cells x 1050 genes)
- latent_factors.csv (true latent factor values per cell, for both scenarios)

## How to apply

Load the CellCycleData dataset from the Citrus R package as the source gene expression profiles, then set up 200 cells in two equal-sized groups with kz=2 confounding factors and ku=5 biological factors. Draw factor loadings for the 50 control genes and the 1,000 target genes at the paper's specified PVE settings, so that among the target genes only 200 are differentially expressed and form the group signature. Combine baseline expression with the confounding and biological loading effects to build the mean expression matrix (separately parameterized for scenario I and scenario II), convert to integer UMI-like counts via Poisson generation (rpois, vectorized with Rcpp/RcppArmadillo), and optionally apply the dropout procedure for zero inflation. The rationale is that fixing the empirical PVE of each factor makes the difficulty of confounding-effect removal explicit and reproducible across method comparisons.

## Related tools

- **Citrus** (R/C++ toolkit providing the CellCycleData dataset used as the source of gene expression profiles and hosting scPLS) — http://chenmengjie.github.io/Citrus/
- **scPLS** (The confounding-effect removal method whose benchmark simulations are generated with this PVE-parameterized design) — http://chenmengjie.github.io/Citrus/
- **Rcpp** (R package dependency used to vectorize Poisson count generation in the simulator)
- **RcppArmadillo** (R package dependency used with Rcpp for fast vectorized simulation of count matrices)

## Examples

```
install_github("ChenMengjie/Citrus"); data(CellCycleData, package="Citrus"); counts <- apply(M, 2, function(x) rpois(200, exp(x)))
```

## Evaluation signals

- Each simulated count matrix has exactly 200 rows (cells) by 1050 columns (50 control + 1000 target genes)
- Re-running the generator with the same seed reproduces byte-identical simulated_counts.csv and latent_factors.csv files
- The empirical PVE of each simulated latent factor matches the specified simulation parameter
- Exactly 200 of the 1,000 target genes are differentially expressed between the two equal-sized cell groups and form the group signature
- CellCycleData from the Citrus package loads successfully and supplies the baseline expression profiles used in the mean expression matrix

## Limitations

- The simulation framework generates expression via Poisson counts plus optional dropout, so it does not implement a full count-based model; the article notes extending the framework to modeling count data is future work
- The design is parameterized for the specific scPLS benchmark geometry (50 control genes, 1,000 target genes, 200 cells, kz=2, ku=5); other geometries require re-deriving the PVE settings
- Two scenarios (I and II) are provided; generalization to additional confounding structures is not covered by the source card

## Evidence

- [methods] The simulation generates gene expression levels for 50 control genes and 1,000 target genes across 200 cells, which come from two equal-sized groups: [evidence span withheld — anchor status: not_found]
- [methods] Poisson count generation from workflow card: [evidence span withheld — anchor status: not_found]
- [intro] Citrus availability from article: "Our method is implemented as a part of the Citrus project and is freely available at: http://chenmengjie.github.io/Citrus/."
- [readme] Citrus README installation dependencies: "To use Citrus, the installation of **Rcpp** and **RcppArmadillo** is required."
