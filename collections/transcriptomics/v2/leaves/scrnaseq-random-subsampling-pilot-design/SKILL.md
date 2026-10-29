---
name: scrnaseq-random-subsampling-pilot-design
description: 'Use when planning scRNA-seq experiments from small pilot data: you need
  to test whether generative models trained on sub-sampled pilot datasets can produce
  synthetic data whose clustering, cell-type proportions, and per-gene distributions
  match a larger ground-truth dataset.'
when_to_use_negative:
- 'Datasets with batch effects you intend to correct: the study examined all models
  without batch-correction capabilities, and scDBMs currently cannot learn batch effects'
- Pilot samples that are not representative of the target population (changed experimental
  settings), since extrapolation from small to larger datasets assumes representativeness
- 'Goals requiring detection of rare cell types from very small pilots: all approaches
  overrepresented highly abundant cell types and missed less abundant ones'
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scDBM
  role: Deep Boltzmann machine trained on pilot sub-samples; synthetic cells generated
    by Gibbs sampling to the original study size
  repo: github.com/MTreppner/scDBM.jl
- name: scVI
  role: 'Variational autoencoder trained on pilots in two variants: posterior sampling
    p(z|x) (latent dim 10) and prior sampling Normal(0,1) with library size Normal(l_mu,1)'
  repo: https://github.com/scverse/scvi-tools
- name: BoltzmannMachines.jl
  role: Julia package on which scDBM.jl is built for training the deep Boltzmann machines
  repo: https://github.com/MTreppner/scDBM.jl
- name: UMAP
  role: Dimensionality reduction used to map synthetic observations to original data
    by Euclidean distance for cluster-label assignment
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:d170264db8d4559b81fffe06ab2d28cbb0fa0008086ae03329acadc4aa92fc25
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BoltzmannMachines.jl
  - Julia
  - UMAP
  - scDBM
  - scVI
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# scrnaseq-random-subsampling-pilot-design

## Summary

A pilot-study design skill for scRNA-seq: random sub-samples (384–2304 cells, 30 repetitions per size) are drawn from a larger ground-truth dataset (PBMC4k, Segerstolpe) to train deep generative models (scDBM, scVI posterior/prior) that extrapolate synthetic data back to the original study size.

## When to use

Use when planning scRNA-seq experiments from small pilot data: you need to test whether generative models trained on sub-sampled pilot datasets can produce synthetic data whose clustering, cell-type proportions, and per-gene distributions match a larger ground-truth dataset.

## When NOT to use

- Datasets with batch effects you intend to correct: the study examined all models without batch-correction capabilities, and scDBMs currently cannot learn batch effects
- Pilot samples that are not representative of the target population (changed experimental settings), since extrapolation from small to larger datasets assumes representativeness
- Goals requiring detection of rare cell types from very small pilots: all approaches overrepresented highly abundant cell types and missed less abundant ones

## Inputs

- Preprocessed scRNA-seq count matrix with HVG filtering, e.g. PBMC4k (4182 cells, 1000 HVGs) or Segerstolpe (2090 cells, 2000 HVGs)
- Cluster labels and gene name files (e.g. segerstolpe_hvg_clustering.csv, segerstolpe_gene_names.csv)
- Pilot-size grid specification (384–2304 cells) and number of repetitions (30) with train/test split proportion 0.7

## Outputs

- 30 random pilot sub-samples per size per dataset with 0.7/0.3 train/test splits
- Synthetic count matrices at original study size for each of 1080 (dataset x model x pilot size x repetition) combinations
- Evaluation metrics: Davies–Bouldin index, adjusted Rand index, cluster-proportion mean absolute differences, and per-gene MAD of medians, IQRs, coefficients of variation, and zero-count proportions

## How to apply

Load the preprocessed count matrices (PBMC4k: 4182 cells x 1000 highly variable genes; Segerstolpe: 2090 cells x 2000 highly variable genes). For each pilot size (384, 768, 1152, 1536, 1920, 2304 cells), draw 30 random sub-samples with each cell sampled at equal probability, and split each sub-sample into train/test subsets with a 0.7 training proportion. Train scDBM (Julia, scDBM.jl on BoltzmannMachines.jl; two hidden layers, latent dimensionality 2–4, small learning rate, regularized Fisher-scoring update of the inverse dispersion parameter every five epochs) and the two scVI variants (scvi=0.6; posterior sampling p(z|x) with latent dimension 10 by replicating the pilot data seven times; prior sampling from Normal(0,1) with library sizes Normal(l_mu,1)) on each pilot. Generate synthetic count matrices via Gibbs sampling (scDBM) or prior/posterior sampling (scVI) until the original study's cell count is reached, giving 2 datasets x 3 models x 6 sizes x 30 repetitions = 1080 synthetic artifacts. Evaluate authenticity with the Davies–Bouldin index, adjusted Rand index, cluster proportions (mean absolute difference across clusters), and per-gene statistics (medians, interquartile ranges, coefficients of variation, zero-count proportions), using UMAP mapping of synthetic cells to nearest original cells for cluster labels.

## Related tools

- **scDBM** (Deep Boltzmann machine trained on pilot sub-samples; synthetic cells generated by Gibbs sampling to the original study size) — github.com/MTreppner/scDBM.jl
- **scVI** (Variational autoencoder trained on pilots in two variants: posterior sampling p(z|x) (latent dim 10) and prior sampling Normal(0,1) with library size Normal(l_mu,1)) — https://github.com/scverse/scvi-tools
- **BoltzmannMachines.jl** (Julia package on which scDBM.jl is built for training the deep Boltzmann machines) — https://github.com/MTreppner/scDBM.jl
- **UMAP** (Dimensionality reduction used to map synthetic observations to original data by Euclidean distance for cluster-label assignment)

## Examples

```
julia> import Pluto; julia> Pluto.run()  # then open scDBM_notebook.jl after Conda.add("scvi=0.6", channel="bioconda")
```

## Evaluation signals

- Completeness check: 2 datasets x 3 models x 6 pilot sizes x 30 repetitions = 1080 synthetic count matrices, each with the original study's cell count
- Each sub-sample split into train/test with 0.7 training proportion before model fitting
- ARI/DBI of clustering on synthetic data (Seurat clustering after UMAP mapping) compared against the reference DBI of Seurat clustering on the original data
- Cluster-proportion mean absolute difference between original and synthetic datasets should decrease with increasing pilot size
- Per-gene MAD of medians, IQRs, coefficients of variation, and zero-count proportions computed against ground truth; expect failures at bimodal genes

## Limitations

- Posterior sampling in scVI amplifies artifacts of small pilot datasets, giving high variability in clustering similarity; prior sampling and scDBM show lower variability
- 10x Genomics datasets (higher sparsity, e.g. PBMC4k) are more challenging for inference from small to larger datasets than Smart-seq2 (Segerstolpe)
- Cluster-label mapping assumes synthetic cells mapped to the original data land in the correct clusters; quality evaluation across multiple measures can be ambiguous
- All approaches overrepresent abundant cell types and miss rare ones; problems occurred with bimodal gene distributions

## Evidence

- [intro] Pilot sizes and repetitions: "The complete analysis is performed for different sizes of pilot datasets (384, 768, 1152, 1536, 1920, and 2304 cells) and repeated 30 times for each size."
- [intro] Random equal-probability sub-sampling: "First, we extract small pilot datasets from the larger original data by random sub-sampling, where each cell is drawn with equal probability"
- [results] Sampling mechanisms: [evidence span withheld — anchor status: not_found]
- [results] Cluster-label mapping: "acquired the cluster labels by mapping the synthetic observations to the original data based on the Euclidean distance in UMAP space"
- [readme] scVI installation in Pluto notebooks: [evidence span withheld — anchor status: external_doc]
- [abstract] Posterior-variant variability finding: "we find that the scVI (posterior) variant resulted in high variability, most likely due to amplifying artifacts of small datasets."
