---
name: bimodal-gene-distribution-visualization
description: Use when synthetic scRNA-seq data generated from small pilot sub-samples
  (384–2304 cells) must be checked against the original data's per-gene distributions,
  especially to detect missed bimodal gene patterns.
when_to_use_negative:
- The goal is batch-effect assessment or correction — the paper explicitly omitted
  batch correction, so this skill says nothing about it.
- Evaluating cluster-level properties (ARI, DBI, cluster proportions) rather than
  per-gene univariate distributions — those are separate evaluation skills.
- The input data are not matched original/synthetic pairs from the same gene panel;
  per-gene MAD comparisons require identical gene sets.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3368
tools:
- name: scDBM.jl
  role: Provides the scDBM model whose synthetic samples (via Gibbs sampling) are
    compared against the original per-gene distributions; runs in Julia 1.5.0
  repo: https://github.com/MTreppner/scDBM.jl
- name: scVI
  role: Provides prior- and posterior-sampled synthetic datasets whose per-gene distributions
    are compared to the original data (installed via Conda as scvi=0.6 from bioconda)
  repo: https://github.com/scverse/scvi-tools
- name: BoltzmannMachines.jl
  role: Base implementation on which scDBM.jl builds for training the deep Boltzmann
    machine and sampling synthetic counts
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:1f117db6874261db79062c9dea4ecf8f2ef45142a377c623d7ed9c147660fb3a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MTreppner/scDBM.jl
  tools:
  - BoltzmannMachines.jl
  - Julia
  - scDBM.jl
  - scVI
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3368
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# bimodal-gene-distribution-visualization

## Summary

Reproduce the per-gene univariate distribution evaluation from the scDBM paper by computing per-gene descriptive statistics (median, IQR, CV, zero-count proportion) for original vs. synthetic scRNA-seq data and visualizing exemplary genes such as CD74 to expose the failure of scDBM/scVI to capture bimodal distributions.

## When to use

Use when synthetic scRNA-seq data generated from small pilot sub-samples (384–2304 cells) must be checked against the original data's per-gene distributions, especially to detect missed bimodal gene patterns.

## When NOT to use

- The goal is batch-effect assessment or correction — the paper explicitly omitted batch correction, so this skill says nothing about it.
- Evaluating cluster-level properties (ARI, DBI, cluster proportions) rather than per-gene univariate distributions — those are separate evaluation skills.
- The input data are not matched original/synthetic pairs from the same gene panel; per-gene MAD comparisons require identical gene sets.

## Inputs

- Original scRNA-seq count matrices (PBMC4k and Segerstolpe)
- Synthetic count matrices from scDBM, scVI prior, and scVI posterior for each pilot size (384, 768, 1152, 1536, 1920, 2304 cells) and 30 replicates
- Gene name files (e.g., segerstolpe_gene_names.csv)

## Outputs

- res_mad_descriptive_stats.csv (per-gene MAD of median, IQR, CV, zero-count proportion, aggregated over 30 replicates by model and pilot size)
- res_bimodality_cd74.png (CD74 marginal distribution comparison, original vs. synthetic)

## How to apply

Load the original count matrices (PBMC4k from 10x Genomics and Segerstolpe from Smart-seq2) together with the synthetic datasets from scDBM (Gibbs sampling), scVI prior, and scVI posterior, each generated for pilot sizes 384, 768, 1152, 1536, 1920, and 2304 cells with 30 sub-sampling replicates. For every gene in both original and synthetic data, compute four descriptive statistics: median, interquartile range, coefficient of variation, and proportion of zero counts. Aggregate the mean absolute deviation (MAD) per gene for each statistic between synthetic and original data across replicates, storing the results in res_mad_descriptive_stats.csv ordered by model, pilot size, and metric. Finally, plot the marginal univariate distributions of exemplary genes (e.g., CD74 in PBMC4k), overlaying the original and each model's synthetic distribution to make the bimodality failure visually explicit, saved as res_bimodality_cd74.png. Rationale: the paper's evaluation shows all models learn most genes' distributions but systematically fail on bimodal genes, so the figure is the diagnostic artifact.

## Related tools

- **scDBM.jl** (Provides the scDBM model whose synthetic samples (via Gibbs sampling) are compared against the original per-gene distributions; runs in Julia 1.5.0) — https://github.com/MTreppner/scDBM.jl
- **scVI** (Provides prior- and posterior-sampled synthetic datasets whose per-gene distributions are compared to the original data (installed via Conda as scvi=0.6 from bioconda)) — https://github.com/scverse/scvi-tools
- **BoltzmannMachines.jl** (Base implementation on which scDBM.jl builds for training the deep Boltzmann machine and sampling synthetic counts)

## Examples

```
julia> import Pluto; Pluto.run()
```

## Evaluation signals

- res_mad_descriptive_stats.csv contains all four metrics (median, IQR, CV, zero-count proportion) for all three models (scDBM, scVI prior, scVI posterior) across all six pilot sizes
- res_bimodality_cd74.png shows the original PBMC4k CD74 distribution is bimodal while every synthetic distribution collapses the bimodality, consistent with Supplementary Fig. 3
- MAD values are aggregated over exactly 30 sub-sampling replicates per pilot size
- Comparisons are computed on identical gene sets between original and synthetic matrices
- For most genes, MAD of descriptive statistics should be small relative to bimodal genes like CD74, reproducing the paper's finding that 'all approaches learn the univariate distribution of most genes'

## Limitations

- All models fail to reproduce bimodal gene distributions, so passing per-gene statistics does not guarantee faithful multimodal structure
- For sparse 10x Genomics data (higher sparsity, e.g., PBMC4k), inference from small to larger datasets is more challenging than for Smart-seq2 (Segerstolpe), so MAD may remain large for small pilot sizes
- Evaluation quality is ambiguous when based on multiple measures; per-gene MAD alone cannot fully certify synthetic data quality

## Evidence

- [results] MAD computation metrics: "we compare medians, interquartile ranges, coefficients of variation, and the proportion of zero counts"
- [abstract] Bimodality finding: "all approaches learn the univariate distribution of most genes, but problems occurred with bimodality"
- [intro] Pilot sizes and replicates: "The complete analysis is performed for different sizes of pilot datasets (384, 768, 1152, 1536, 1920, and 2304 cells) and repeated 30 times for each size."
- [readme] Pluto notebook execution: [evidence span withheld — anchor status: external_doc]
