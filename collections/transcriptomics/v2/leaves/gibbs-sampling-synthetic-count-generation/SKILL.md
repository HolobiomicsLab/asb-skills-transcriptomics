---
name: gibbs-sampling-synthetic-count-generation
description: Use when a trained scDBM (BoltzmannMachines.jl-based model on scRNA-seq
  count data) must be turned into synthetic observations of a target cell count, and
  when comparing sampling strategies such as Gibbs sampling against scVI prior/posterior
  sampling.
when_to_use_negative:
- The goal is to reproduce a specific small pilot's cells rather than extrapolate
  to a larger dataset — scDBM sampling targets study-size generation, not pilot replication
- Batch effects must be modeled or corrected — scDBMs currently cannot learn batch
  effects and all models were examined without batch-correction capabilities
- The pilot sample is not representative of the target setting (experimental settings
  changed) — conclusions from small to larger datasets then violate the paper's stated
  assumption
edam_operation: http://edamontology.org/operation_3928
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scDBM.jl
  role: Julia implementation providing the scDBM training and Gibbs sampling used
    to generate synthetic counts
  repo: github.com/MTreppner/scDBM.jl
- name: BoltzmannMachines.jl
  role: Underlying Julia package on which scDBM.jl's DBM training and Gibbs sampling
    are based
  repo: github.com/MTreppner/scDBM.jl
- name: scDBM
  role: The deep Boltzmann machine model proposed by the paper; trained on pilot sub-samples
    then sampled via Gibbs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:24bc3ca42f2f122b4b400284d4231e8e2505f48226147f41b811f92326f8afb8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BoltzmannMachines.jl
  - Julia
  - scDBM
  - scDBM.jl
  edam_operation: http://edamontology.org/operation_3928
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# gibbs-sampling-synthetic-count-generation

## Summary

Generates synthetic scRNA-seq count matrices from a trained single-cell deep Boltzmann machine (scDBM) by Gibbs sampling until the original study's cell count is reached, enabling extrapolation from small pilot studies (e.g. 384–2304 cells from PBMC4k or Segerstolpe) to full study size.

## When to use

Use when a trained scDBM (BoltzmannMachines.jl-based model on scRNA-seq count data) must be turned into synthetic observations of a target cell count, and when comparing sampling strategies such as Gibbs sampling against scVI prior/posterior sampling.

## When NOT to use

- The goal is to reproduce a specific small pilot's cells rather than extrapolate to a larger dataset — scDBM sampling targets study-size generation, not pilot replication
- Batch effects must be modeled or corrected — scDBMs currently cannot learn batch effects and all models were examined without batch-correction capabilities
- The pilot sample is not representative of the target setting (experimental settings changed) — conclusions from small to larger datasets then violate the paper's stated assumption

## Inputs

- Preprocessed scRNA-seq count matrices (PBMC4k: 4182 cells x 1000 HVGs; Segerstolpe: 2090 cells x 2000 HVGs)
- Random pilot sub-samples of 384, 768, 1152, 1536, 1920, or 2304 cells with 0.7 training proportion split
- A trained scDBM model (scDBM.jl / BoltzmannMachines.jl)

## Outputs

- Synthetic count matrices with the original study's cell count (e.g. 4182 x 1000 for PBMC4k)
- 1080 total generated artifacts across 2 datasets x 3 models x 6 pilot sizes x 30 repetitions

## How to apply

Train an scDBM (Julia, scDBM.jl on BoltzmannMachines.jl) on the sub-sampled pilot count matrix using two hidden layers, a latent dimensionality between two and four, a small learning rate, and periodic refinement of the regularized Fisher-scoring estimate of the inverse dispersion parameter every five epochs. Once trained, run Gibbs sampling on the model to draw synthetic cell observations one at a time until the number of generated cells equals the original study's size (e.g. 4182 cells for PBMC4k, 2090 for Segerstolpe). The context-free prior of the DBM makes this sampling variant less dependent on the pilot's specific cells, which is why it exhibits much lower variability than scVI posterior sampling in the article's experiments. Validate that for each dataset x pilot-size x repetition combination a full-size synthetic count matrix was produced (1080 artifacts across 2 datasets x 6 pilot sizes x 30 repetitions x 3 models in the paper).

## Related tools

- **scDBM.jl** (Julia implementation providing the scDBM training and Gibbs sampling used to generate synthetic counts) — github.com/MTreppner/scDBM.jl
- **BoltzmannMachines.jl** (Underlying Julia package on which scDBM.jl's DBM training and Gibbs sampling are based) — github.com/MTreppner/scDBM.jl
- **scDBM** (The deep Boltzmann machine model proposed by the paper; trained on pilot sub-samples then sampled via Gibbs)

## Examples

```
julia> import Pluto; Pluto.run()  # run scDBM_notebook.jl from MTreppner/scDBM-paper after Conda.add("scvi=0.6", channel="bioconda")
```

## Evaluation signals

- For every dataset x model x pilot-size x repetition combination a synthetic count matrix with the original study's cell count was produced (1080 artifacts total)
- Clustering similarity (ARI, DBI, cluster proportions) of synthetic data vs. ground-truth clustering shows low variability compared to scVI posterior sampling
- Per-gene distribution checks: medians, interquartile ranges, coefficients of variation, and zero-count proportions show low mean absolute deviation from original data
- With increasing pilot size, cluster proportions of synthetic cells converge toward ground-truth proportions

## Limitations

- All approaches overrepresented highly abundant cell types and missed less abundant cell types
- Problems occurred with bimodal gene distributions even though univariate distributions of most genes were learned
- Inference from small to larger datasets is more challenging for sparser 10x Genomics datasets (PBMC4k) than Smart-seq2 (Segerstolpe)
- scDBMs currently cannot learn batch effects
- Evaluating mapping-based cluster labels relies on the unverified assumption that synthetic cells map to correct clusters

## Evidence

- [results] Gibbs sampling generates scDBM synthetic data to original size: "we sampled from the scDBM using Gibbs sampling and from scVI using the prior and posterior distribution"
- [abstract] Variability comparison favors prior-style sampling over posterior amplification: "we find that the scVI (posterior) variant resulted in high variability, most likely due to amplifying artifacts of small datasets."
- [intro] Pilot sizes and repetitions define the generation grid: "The complete analysis is performed for different sizes of pilot datasets (384, 768, 1152, 1536, 1920, and 2304 cells) and repeated 30 times for each size."
- [readme] README confirms Julia implementation and requirement: "The package is based on the implementation 'BoltzmannMachines.jl'"
