---
name: snapccess-embedding-partitioning
description: Use when SnapCCESS variational-autoencoder embeddings of multimodal single-cell
  omics data (e.g. CITE-seq RNA/ADT) have already been generated and the goal is to
  partition cells into clusters and quantify clustering quality against known cell-type
  labels.
when_to_use_negative:
- Raw multimodal count matrices have not yet been converted into SnapCCESS ensemble
  embeddings — run the embedding-generation skill first
- Single-modality data with no cell-type reference labels available, since ARI/NMI
  evaluation against reference labels cannot be performed
- The task is choosing optimal hyperparameters for generating embeddings — the tutorial
  explicitly does not recommend best parameters and defers to the paper
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3173
tools:
- name: SnapCCESS
  role: Framework whose ensemble embeddings are partitioned by the clustering algorithms
  repo: https://github.com/PYangLab/SnapCCESS
- name: snapccess
  role: Python package used to load the generated embeddings for each dataset and
    ensemble size
  repo: https://github.com/yulijia/SnapCCESS/tree/main/snapccess-py
- name: k-means (scikit-learn)
  role: Clustering algorithm run on the embeddings for every dataset and ensemble-size
    setting
- name: CiteFuse spectral clustering
  role: Spectral clustering implementation run on the same embeddings
- name: SIMLR
  role: Clustering algorithm run on the same embeddings
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:07ffed059eeee273c618d1630680a986939dea2d813dc7e53bb56e8db00f8806
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - CiteFuse spectral clustering
  - Python
  - SIMLR
  - SnapCCESS
  - k-means (scikit-learn)
  - snapccess
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# snapccess-embedding-partitioning

## Summary

Couples SnapCCESS-generated multimodal single-cell embeddings with downstream clustering algorithms (k-means, CiteFuse spectral clustering, SIMLR) to generate consensus cell clusters, and evaluates them against reference cell-type labels using ARI and NMI across ensemble sizes.

## When to use

Use when SnapCCESS variational-autoencoder embeddings of multimodal single-cell omics data (e.g. CITE-seq RNA/ADT) have already been generated and the goal is to partition cells into clusters and quantify clustering quality against known cell-type labels.

## When NOT to use

- Raw multimodal count matrices have not yet been converted into SnapCCESS ensemble embeddings — run the embedding-generation skill first
- Single-modality data with no cell-type reference labels available, since ARI/NMI evaluation against reference labels cannot be performed
- The task is choosing optimal hyperparameters for generating embeddings — the tutorial explicitly does not recommend best parameters and defers to the paper

## Inputs

- SnapCCESS embeddings previously generated for each dataset at each ensemble size (snapccess Python package)
- Reference cell-type labels for each dataset
- CITE-seq-style multimodal single-cell data (RNA and ADT matrices, features by cells) underlying the embeddings

## Outputs

- Cell cluster assignments per algorithm (k-means, spectral clustering, SIMLR) per dataset and ensemble size
- ARI and NMI scores per algorithm, dataset, and ensemble size
- Aggregated ARI/NMI comparison table (e.g. algorithm_comparison_ari_nmi.csv) mirroring Figure 4b / Supplementary Fig. S5

## How to apply

Load the SnapCCESS embeddings previously generated for each eligible dataset at each ensemble size using the snapccess Python package. Run k-means (scikit-learn), spectral clustering as implemented in CiteFuse, and SIMLR on the same embeddings for every dataset and ensemble-size setting. Compute Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI) of each algorithm's clustering against the reference cell-type labels, then aggregate the ARI/NMI scores into a table mirroring Figure 4b / Supplementary Fig. S5. Rationale: the paper found clustering performance increases with ensemble size regardless of the clustering algorithm and concordance metric, with improvement peaking around an ensemble size of 50, so embeddings from multiple ensemble sizes should be compared rather than a single snapshot.

## Related tools

- **SnapCCESS** (Framework whose ensemble embeddings are partitioned by the clustering algorithms) — https://github.com/PYangLab/SnapCCESS
- **snapccess** (Python package used to load the generated embeddings for each dataset and ensemble size) — https://github.com/yulijia/SnapCCESS/tree/main/snapccess-py
- **k-means (scikit-learn)** (Clustering algorithm run on the embeddings for every dataset and ensemble-size setting)
- **CiteFuse spectral clustering** (Spectral clustering implementation run on the same embeddings)
- **SIMLR** (Clustering algorithm run on the same embeddings)

## Examples

```
pip install snapccess --index-url https://pypi.org/simple
```

## Evaluation signals

- ARI and NMI computed against reference cell-type labels for each dataset, algorithm, and ensemble size
- ARI/NMI scores should increase with ensemble size and peak around an ensemble size of 50, mirroring Figure 4b / Supplementary Fig. S5
- The trend holds across all three algorithms (k-means, spectral, SIMLR) and both concordance metrics
- Aggregated results reproduce a table comparable to algorithm_comparison_ari_nmi.csv with one row per dataset/ensemble-size/algorithm combination

## Limitations

- The tutorials only explain how to use the package; they do not recommend the best parameters for arbitrary datasets — parameters for the published datasets are listed in the paper
- No discussion-section guidance (changelog/body text) is available for troubleshooting
- SnapCCESS expects a list of modalities measured across the same cells (e.g. RNA and ADT from CITE-seq), arranged features-by-cells before preprocessing in the R wrapper

## Evidence

- [results] A clear increase in clustering performance was observed with increasing ensemble size, regardless of the clustering algorithm used (k-means, spectral, SIMLR) and the concordance metric, with improvement peaking around an ensemble size of 50.: [evidence span withheld — anchor status: not_found]
- [abstract] By creating snapshots of embeddings of multimodality using variational autoencoders, SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells: "By creating snapshots of embeddings of multimodality using variational autoencoders, SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells"
- [readme] SnapCCESS expects a list of modalities measured across the same cells, for example RNA and ADT matrices from CITE-seq. Each modality should be arranged as features by cells before preprocessing in the R wrapper.: [evidence span withheld — anchor status: external_doc]
- [methods] This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets.: "This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets."
