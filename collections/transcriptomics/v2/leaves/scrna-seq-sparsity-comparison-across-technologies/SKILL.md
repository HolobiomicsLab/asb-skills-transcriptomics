---
name: scrna-seq-sparsity-comparison-across-technologies
description: Use when comparing synthetic-data quality from small pilot sub-samples
  between scRNA-seq technologies (10x Genomics vs Smart-seq2), specifically to quantify
  whether sparsity degrades small-to-large inference via DBI, ARI, and cluster-proportion
  metrics.
when_to_use_negative:
- The goal is batch correction or batch-effect modeling — the paper explicitly omits
  batch correction and examined all models without batch correction capabilities
- You need per-gene distribution quality assessment (medians, IQRs, CVs, zero-count
  proportions) rather than technology-level sparsity comparison via DBI/ARI/cluster
  proportions
- The pilot samples come from different experimental settings than the target data,
  violating the representativeness assumption required for small-to-large inference
edam_operation: http://edamontology.org/operation_3436
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: scVI
  role: Generative model trained on pilot sub-samples; posterior variant (standard
    approach) shows high variability on sparse 10x data, prior variant sampled for
    comparison
  repo: https://github.com/scverse/scvi-tools
- name: scDBM
  role: Single-cell deep Boltzmann machine trained on pilot sub-samples and sampled
    via Gibbs sampling; compared against scVI variants across technologies
  repo: https://github.com/MTreppner/scDBM.jl
- name: UMAP
  role: Dimensionality reduction used to map synthetic observations to original data
    by Euclidean distance to acquire cluster labels
- name: Seurat
  role: Clustering of original data providing the reference DBI (orange dotted line
    in Figure 2) and ground-truth cluster labels
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-021-88875-4
    title: Synthetic single cell RNA sequencing data from small pilot studies using
      deep generative models
  source_content_hash: sha256:7b84579c0b8436e989addd0ba73b35da8ab393656c9f6102f39498c3d000e2e7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/scverse/scvi-tools
  tools:
  - Seurat
  - UMAP
  - scDBM
  - scVI
  edam_operation: http://edamontology.org/operation_3436
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# scrna-seq-sparsity-comparison-across-technologies

## Summary

A stratified tabulation skill that contrasts DBI/ARI variability and cluster-proportion deviations across 10x Genomics (PBMC4k) and Smart-seq2 (Segerstolpe) scRNA-seq data to show that higher sparsity makes inference from small pilot studies to larger datasets more challenging.

## When to use

Use when comparing synthetic-data quality from small pilot sub-samples between scRNA-seq technologies (10x Genomics vs Smart-seq2), specifically to quantify whether sparsity degrades small-to-large inference via DBI, ARI, and cluster-proportion metrics.

## When NOT to use

- The goal is batch correction or batch-effect modeling — the paper explicitly omits batch correction and examined all models without batch correction capabilities
- You need per-gene distribution quality assessment (medians, IQRs, CVs, zero-count proportions) rather than technology-level sparsity comparison via DBI/ARI/cluster proportions
- The pilot samples come from different experimental settings than the target data, violating the representativeness assumption required for small-to-large inference

## Inputs

- Reported DBI and ARI values from Figure 2 panels for PBMC4k and Segerstolpe across six pilot sizes with 30 sub-samples each
- Table 1 mean absolute differences in cell numbers per cluster between original and synthetic datasets
- Prepared count matrices and cluster labels from data_preparation_segerstolpe.Rmd and data_preparation_PBMC.Rmd

## Outputs

- technology_comparison.csv — tidy stratified table with columns technology, dataset, generative model, metric, and value per pilot size

## How to apply

Extract the reported DBI and ARI values for PBMC4k (10x Genomics) and Segerstolpe (Smart-seq2) across the six pilot sizes (384, 768, 1152, 1536, 1920, 2304 cells; 30 sub-samples each) from Figure 2 panels, plus the mean absolute differences in cell numbers per cluster from Table 1 (noting Segerstolpe covers only 1-5 plates). Assemble a tidy table with columns for technology, dataset, generative model (scDBM, scVI prior, scVI posterior, negative binomial noise baseline), metric, and value per pilot size. Contrast the high variability of the scVI posterior variant on PBMC4k against the tighter DBI distributions and smaller Table 1 differences on Segerstolpe, which supports the finding that sparse 10x data are harder to generalize from small pilots. Write the result to technology_comparison.csv and validate against the paper's reported numbers and qualitative Figure 2 trends.

## Related tools

- **scVI** (Generative model trained on pilot sub-samples; posterior variant (standard approach) shows high variability on sparse 10x data, prior variant sampled for comparison) — https://github.com/scverse/scvi-tools
- **scDBM** (Single-cell deep Boltzmann machine trained on pilot sub-samples and sampled via Gibbs sampling; compared against scVI variants across technologies) — https://github.com/MTreppner/scDBM.jl
- **UMAP** (Dimensionality reduction used to map synthetic observations to original data by Euclidean distance to acquire cluster labels)
- **Seurat** (Clustering of original data providing the reference DBI (orange dotted line in Figure 2) and ground-truth cluster labels)

## Examples

```
julia> import Pluto; Pluto.run()  # then open scDBM_notebook.jl and table_1.Rmd from MTreppner/scDBM-paper to reproduce Table 1 and Figure 2 comparisons
```

## Evaluation signals

- Tabulated values match the paper's reported Table 1 numbers exactly
- Qualitative DBI/ARI trends of Figure 2 are reproduced: higher variability for scVI posterior on PBMC4k, tighter DBI distributions closer to the reference on Segerstolpe
- Segerstolpe shows smaller mean absolute cluster-proportion differences than PBMC4k in the technology_comparison.csv
- All six pilot sizes (384–2304 cells) and 30 sub-samples per size are represented for both datasets and all four generative models (scDBM, scVI prior, scVI posterior, noise baseline)
- Output CSV has the required schema: technology, dataset, generative model, metric (DBI, ARI, cluster-proportion difference), value per pilot size

## Limitations

- Inference from small to larger datasets requires the pilot sample to be representative, which depends on keeping experimental settings unchanged
- Segerstolpe covers only 1-5 plates, limiting the range of pilot sizes comparable to PBMC4k
- The approach relies on the unverified assumption that synthetic cells mapped to the original data are mapped to the correct clusters
- All models were examined without potentially available batch correction capabilities, and scDBMs currently cannot learn batch effects

## Evidence

- [readme] Across all analyses, in comparing 10x Genomics and Smart-seq2 technologies, we could show that for 10x datasets, which have higher sparsity, it is more challenging to make an inference from small to larger datasets.: "for 10x datasets, which have higher sparsity, it is more challenging to make an inference from small to larger datasets."
- [results] Looking at the Segerstolpe data, which were generated with the Smart-seq2 protocol: "Looking at the Segerstolpe data, which were generated with the Smart-seq2 protocol"
- [results] We calculated the mean of the absolute differences between the number of cells in the original dataset and the respective synthetic datasets across all clusters: "We calculated the mean of the absolute differences between the number of cells in the original dataset and the respective synthetic datasets across all clusters"
- [intro] The complete analysis is performed for different sizes of pilot datasets (384, 768, 1152, 1536, 1920, and 2304 cells) and repeated 30 times for each size.: "The complete analysis is performed for different sizes of pilot datasets (384, 768, 1152, 1536, 1920, and 2304 cells) and repeated 30 times for each size."
- [discussion] If conclusions are drawn from a small sample to a larger dataset, it is important that the pilot sample is representative.: "If conclusions are drawn from a small sample to a larger dataset, it is important that the pilot sample is representative."
