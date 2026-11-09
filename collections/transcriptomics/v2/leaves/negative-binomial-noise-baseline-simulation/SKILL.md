---
name: negative-binomial-noise-baseline-simulation
description: 'Use when a synthetic scRNA-seq benchmark needs a noise-floor control:
  pilot sub-samples (384–2304 cells) exist and Davies–Bouldin indices of generative
  models (scDBM, scVI) must be judged against NB-noise-augmented pilot data.'
when_to_use_negative:
- Do not use as evidence of biological realism — the baseline carries no cell-type
  proportions or gene-gene structure, so cluster proportions, ARI, and per-gene distribution
  checks (medians, IQRs, CVs, zero proportions) require the generative models.
- 'Skip it when batch effects matter: the study explicitly omitted batch correction
  and scDBM cannot learn batch effects, so the noise baseline offers no batch-aware
  comparison.'
- Not suitable for already-normalized continuous expression matrices — the gamma/Poisson-rate
  construction assumes raw count-like values added to pilot counts.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scDBM
  role: Deep Boltzmann machine (built on BoltzmannMachines.jl) trained on the same
    pilot sub-samples; its synthetic-data DBI is benchmarked against the NB-noise
    baseline
  repo: github.com/MTreppner/scDBM.jl
- name: scVI
  role: Generates synthetic data from the prior and posterior distributions whose
    DBIs are compared with the noise baseline's DBI
  repo: scverse/scvi-tools
- name: UMAP
  role: Embedding space in which noisy-pilot cells are assigned nearest-original-neighbor
    cluster labels before the baseline DBI is computed
- name: Seurat
  role: Provides the reference DBI (orange dotted line) of the Seurat clustering on
    the original data used to frame the baseline comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:6136a018c2cda48e43d916afd0f74e0cd9fb6866c0557f63edfee305726d4d6b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Julia
  - Seurat
  - UMAP
  - scDBM
  - scVI
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# negative-binomial-noise-baseline-simulation

## Summary

Builds the paper's trivial benchmark for synthetic scRNA-seq: gamma scale and shape parameters drawn from Uniform(0, 5) and Uniform(0, 10) act as Poisson rate parameters, and the resulting negative-binomial noise is added to pilot sub-samples so its Davies–Bouldin index can be compared against synthetic data from scDBM and scVI (prior/posterior).

## When to use

Use when a synthetic scRNA-seq benchmark needs a noise-floor control: pilot sub-samples (384–2304 cells) exist and Davies–Bouldin indices of generative models (scDBM, scVI) must be judged against NB-noise-augmented pilot data.

## When NOT to use

- Do not use as evidence of biological realism — the baseline carries no cell-type proportions or gene-gene structure, so cluster proportions, ARI, and per-gene distribution checks (medians, IQRs, CVs, zero proportions) require the generative models.
- Skip it when batch effects matter: the study explicitly omitted batch correction and scDBM cannot learn batch effects, so the noise baseline offers no batch-aware comparison.
- Not suitable for already-normalized continuous expression matrices — the gamma/Poisson-rate construction assumes raw count-like values added to pilot counts.

## Inputs

- Preprocessed original scRNA-seq count matrices (PBMC4k 10x and Segerstolpe Smart-seq2)
- Random pilot sub-samples of 384, 768, 1152, 1536, 1920, and 2304 cells (30 replicates per size)
- UMAP embedding and Seurat cluster labels of the original data (for nearest-neighbor label assignment and reference DBI)

## Outputs

- Negative-binomial-noise-augmented pilot dataset merged to the original dataset size
- Baseline Davies–Bouldin index values per pilot size and replicate, written alongside model DBIs into dbi_ari_results.csv

## How to apply

Take the same random pilot sub-samples (384, 768, 1152, 1536, 1920, and 2304 cells; 30 replicates per size) that were used to train scDBM and scVI on PBMC4k or Segerstolpe. For each sub-sample, draw gamma scale parameters from Uniform(0, 5) and shape parameters from Uniform(0, 10) and use them as Poisson rate parameters, so the resulting mixture constitutes negative-binomial noise. Add this noise to the sub-sampled pilot data, then merge replicated noisy pilot sets to the size of the original study so the control matches the synthetic datasets' observation count. Assign cluster labels to the noisy observations by nearest original observation (Euclidean distance in UMAP space) and compute the Davies–Bouldin index, aggregating values across pilot sizes and replicates into dbi_ari_results.csv. The rationale is that the noise baseline's DBI defines the floor the generative models must beat — the paper reports the scVI (posterior) variant sometimes performed even worse than this simple noisy-pilot baseline at small pilot sizes.

## Related tools

- **scDBM** (Deep Boltzmann machine (built on BoltzmannMachines.jl) trained on the same pilot sub-samples; its synthetic-data DBI is benchmarked against the NB-noise baseline) — github.com/MTreppner/scDBM.jl
- **scVI** (Generates synthetic data from the prior and posterior distributions whose DBIs are compared with the noise baseline's DBI) — scverse/scvi-tools
- **UMAP** (Embedding space in which noisy-pilot cells are assigned nearest-original-neighbor cluster labels before the baseline DBI is computed)
- **Seurat** (Provides the reference DBI (orange dotted line) of the Seurat clustering on the original data used to frame the baseline comparison)

## Examples

```
julia> import Pluto; Pluto.run()  # then open scDBM_notebook.jl, which includes the NB-noise baseline analysis
```

## Evaluation signals

- A complete grid of baseline DBI values exists for all 6 pilot sizes × 30 replicates, matching the scDBM/scVI entries in dbi_ari_results.csv.
- Replicated noisy pilot sets are merged to exactly the original dataset size before DBI computation, so observation counts match the synthetic datasets.
- Gamma draws respect Uniform(0, 5) for scale and Uniform(0, 10) for shape, and the Poisson-rate construction yields non-negative integer counts.
- At small pilot sizes the scVI-posterior DBI distribution shows higher variability than the noise baseline, consistent with the reported artifact-amplification finding.
- The baseline DBI is reported together with the orange dotted reference DBI of the original-data Seurat clustering for scale sanity.

## Limitations

- The baseline is a pure noise floor: proximity to it only signals that a model amplified artifacts of small datasets, not that synthetic data is realistic.
- DBI-based benchmarking relies on the unverified assumption that cells mapped to nearest original observations land in the correct clusters.
- Quality assessment via DBI alone can be ambiguous — the paper notes synthetic-data quality is difficult to evaluate from multiple measures.
- No batch effects are modeled: batch correction was outside the manuscript's scope and scDBM cannot learn batch effects.

## Evidence

- [results] NB-noise baseline construction: gamma draws from Uniform(0,5)/Uniform(0,10): "We generated negative binomial noise by randomly drawing the scale and shape parameters of a gamma distribution from Uniform(0, 5) and Uniform(0, 10)"
- [intro] Design grid: six pilot sizes, 30 replicates each: "The complete analysis is performed for different sizes of pilot datasets (384, 768, 1152, 1536, 1920, and 2304 cells) and repeated 30 times for each size."
- [readme] Why the noise floor matters: scVI-posterior variability at small pilots: "the scVI (posterior) variant resulted in high variability, most likely due to amplifying artifacts of small datasets"
- [results] Reference DBI of original Seurat clustering anchors the benchmark: "The orange dotted line indicates the reference DBI for the Seurat clustering on the original data."
