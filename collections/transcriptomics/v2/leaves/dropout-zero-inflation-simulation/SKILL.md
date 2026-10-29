---
name: dropout-zero-inflation-simulation
description: 'Use when building a benchmark simulation for confounder-removal or normalization
  methods in scRNA-seq: you need count matrices with known latent factors, a group
  signature of 200 DE genes, and controlled zero inflation via dropout.'
when_to_use_negative:
- 'Applying the simulated-data rationale directly to real scRNAseq studies: scPLS''s
  framework itself does not model count data, and extending it to modeling count data
  is explicitly left as future work'
- Using the simulation to recover biological factors u_i themselves — the associated
  method can only infer biological effects (Lambda_u u_i), not the factors, because
  of model identifiability
- Evaluating downstream analyses other than clustering (e.g. differential expression,
  lineage reconstruction), which the source study declares beyond its scope
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Citrus
  role: R/C++ toolkit hosting scPLS and the CellCycleData dataset used as the expression
    source for simulation
  repo: http://chenmengjie.github.io/Citrus/
- name: Rcpp
  role: R package dependency enabling fast C++-backed Poisson count generation for
    the simulation
- name: RcppArmadillo
  role: R package dependency for vectorized linear-algebra-heavy simulation of factor
    loadings and count generation
- name: scPLS
  role: Target method benchmarked on the simulated counts; jointly models control
    and target genes by partial least squares regression to infer confounding effects
  repo: http://chenmengjie.github.io/Citrus/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-017-13665-w
    title: Controlling for Confounding Effects in Single Cell RNA Sequencing Studies
      Using both Control and Target Genes
  source_content_hash: sha256:457d8ddc64f6e6a51028956b0ab92040d3a05604cdbf822d11e1d96fd0b0614f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Citrus
  - Rcpp
  - RcppArmadillo
  - scPLS
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# dropout-zero-inflation-simulation

## Summary

Generates synthetic scRNA-seq count matrices with latent confounding (kz=2) and biological (ku=5) factors over 200 cells and 1,050 genes (50 control + 1,000 target), converting continuous expression to UMI-like counts via Poisson sampling and optionally applying a dropout procedure to mimic zero inflation, for benchmarking scPLS against other confounding-effect removal methods.

## When to use

Use when building a benchmark simulation for confounder-removal or normalization methods in scRNA-seq: you need count matrices with known latent factors, a group signature of 200 DE genes, and controlled zero inflation via dropout.

## When NOT to use

- Applying the simulated-data rationale directly to real scRNAseq studies: scPLS's framework itself does not model count data, and extending it to modeling count data is explicitly left as future work
- Using the simulation to recover biological factors u_i themselves — the associated method can only infer biological effects (Lambda_u u_i), not the factors, because of model identifiability
- Evaluating downstream analyses other than clustering (e.g. differential expression, lineage reconstruction), which the source study declares beyond its scope

## Inputs

- CellCycleData dataset from the Citrus R package (source of gene expression profiles)
- Simulation parameters: 200 cells, 50 control genes, 1,000 target genes, 200 DE genes, kz=2 confounding factors, ku=5 biological factors, per-factor PVE settings, random seed, dropout on/off flag

## Outputs

- simulated_counts.csv — integer UMI-like count matrix (200 cells x 1050 genes) per scenario, optionally dropout-inflated
- latent_factors.csv — true latent confounding (kz=2) and biological (ku=5) factors for the 200 cells

## How to apply

Load the CellCycleData dataset from the Citrus R package as the source of gene expression profiles. Generate the latent structure for 200 cells with kz=2 confounding factors and ku=5 biological factors, drawing loadings for the 50 control and 1,000 target genes at the specified proportions of variance explained (PVE) for each scenario (scenario I vs scenario II), with only 200 of the target genes differentially expressed to form the group signature. Combine baseline expression with confounding and biological loading effects into a mean expression matrix, then convert to integer UMI-like counts via Poisson sampling (rpois in R, vectorized with Rcpp/RcppArmadillo). Apply the optional dropout procedure to the count matrix to mimic zero inflation in scRNAseq data, and write seeded outputs (simulated_counts.csv, latent_factors.csv) per scenario so the ground-truth latent factors are available for evaluating method performance.

## Related tools

- **Citrus** (R/C++ toolkit hosting scPLS and the CellCycleData dataset used as the expression source for simulation) — http://chenmengjie.github.io/Citrus/
- **Rcpp** (R package dependency enabling fast C++-backed Poisson count generation for the simulation)
- **RcppArmadillo** (R package dependency for vectorized linear-algebra-heavy simulation of factor loadings and count generation)
- **scPLS** (Target method benchmarked on the simulated counts; jointly models control and target genes by partial least squares regression to infer confounding effects) — http://chenmengjie.github.io/Citrus/

## Examples

```
counts <- matrix(rpois(200*1050, lambda = expr_mean), nrow = 200); write.csv(counts, "simulated_counts.csv")
```

## Evaluation signals

- Each simulated count matrix has exactly 200 rows (cells) by 1050 columns (50 control + 1000 target genes)
- Re-running the generator with the same seed reproduces identical simulated_counts.csv and latent_factors.csv files
- Empirical PVE of each simulated factor matches the specified PVE simulation parameter
- Dropout-inflated runs show increased zero fraction relative to pre-dropout counts while row/column dimensions are preserved
- Exactly 200 target genes are differentially expressed between the two equal-sized cell groups, forming the group signature

## Limitations

- Direct inference of the biological factors u_i is not possible with the associated scPLS method because of model identifiability; only the biological effects are inferable
- The simulation framework supports benchmarking, but the 'best' confounding-effect removal method may vary depending on the downstream question of interest, and scPLS performance in other analysis settings has not been evaluated
- No count-based model is provided by the underlying method; extending the framework to modeling count data is left as future work

## Evidence

- [full_text] The simulation generates gene expression levels for 50 control genes and 1,000 target genes across 200 cells, which come from two equal-sized groups: [evidence span withheld — anchor status: not_found]
- [full_text] Poisson count generation: [evidence span withheld — anchor status: not_found]
- [full_text] Dropout application: [evidence span withheld — anchor status: not_found]
- [readme] Citrus toolkit description: "Citrus is a toolkit for single cell sequencing analysis under development. Citrus is written in c++ with easy-to-use R wrappers."
