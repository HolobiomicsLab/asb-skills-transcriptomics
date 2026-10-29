---
name: clustering-accuracy-metric-computation
description: Use when cluster assignments produced from multimodal single-cell embeddings
  must be scored against reference cell-type labels (e.g. CITE-seq RNA/ADT data) to
  compare clustering algorithms or settings such as ensemble size.
when_to_use_negative:
- No reference cell-type labels are available — ARI and NMI are concordance metrics
  requiring ground-truth labels and cannot be computed
- Input is raw RNA/ADT count matrices rather than SnapCCESS embeddings — clustering
  accuracy scoring presupposes an embedding step has already been run
- The goal is choosing best hyperparameters for new datasets — the SnapCCESS tutorial
  explicitly does not recommend parameters and defers to the publication's dataset-specific
  settings
edam_operation: http://edamontology.org/operation_0342
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: snapccess
  role: Python package supplying the ensemble embeddings on which clustering and ARI/NMI
    evaluation are run
  repo: https://github.com/PYangLab/SnapCCESS
- name: scikit-learn
  role: Provides k-means clustering and the adjusted_rand_score/normalized_mutual_info_score
    metrics
- name: CiteFuse
  role: Source of the spectral clustering implementation applied to the SnapCCESS
    embeddings
- name: SIMLR
  role: Third clustering algorithm coupled to the SnapCCESS embeddings for algorithm
    comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:499ad5795541c592a3bc8c9d43bad70c26566adf5b631680411db50c588b94bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PYangLab/SnapCCESS
  tools:
  - CiteFuse
  - CiteFuse spectral clustering
  - Python
  - SIMLR
  - scikit-learn
  - snapccess
  edam_operation: http://edamontology.org/operation_0342
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# clustering-accuracy-metric-computation

## Summary

Quantifies clustering quality of cell embeddings (e.g. SnapCCESS ensemble snapshots) by computing Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI) against reference cell-type labels, enabling comparison of downstream clustering algorithms such as k-means, CiteFuse spectral clustering, and SIMLR.

## When to use

Use when cluster assignments produced from multimodal single-cell embeddings must be scored against reference cell-type labels (e.g. CITE-seq RNA/ADT data) to compare clustering algorithms or settings such as ensemble size.

## When NOT to use

- No reference cell-type labels are available — ARI and NMI are concordance metrics requiring ground-truth labels and cannot be computed
- Input is raw RNA/ADT count matrices rather than SnapCCESS embeddings — clustering accuracy scoring presupposes an embedding step has already been run
- The goal is choosing best hyperparameters for new datasets — the SnapCCESS tutorial explicitly does not recommend parameters and defers to the publication's dataset-specific settings

## Inputs

- SnapCCESS embeddings generated for each dataset at each ensemble-size setting (snapccess Python package output)
- Reference cell-type label file per dataset (e.g. CITE-seq cell type files as provided in tutorials/in/)
- Choice of clustering algorithm settings: k-means (scikit-learn), spectral clustering (CiteFuse), SIMLR

## Outputs

- Per-dataset, per-ensemble-size ARI and NMI values for each clustering algorithm
- Aggregated comparison table (e.g. algorithm_comparison_ari_nmi.csv) mirroring Figure 4b / Supplementary Fig. S5

## How to apply

Load the SnapCCESS embeddings previously generated for each dataset at each ensemble-size setting (via the snapccess Python package). For each dataset and ensemble size, run k-means clustering with scikit-learn, spectral clustering as implemented in CiteFuse, and SIMLR clustering on the same embeddings. Compute Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI) of each algorithm's clustering against the reference cell-type labels. Aggregate the ARI/NMI scores into a table mirroring Figure 4b / Supplementary Fig. S5 (e.g. algorithm_comparison_ari_nmi.csv). Expect a clear increase in clustering performance with increasing ensemble size regardless of the clustering algorithm used, with improvement peaking around an ensemble size of 50 — a deviation from this trend indicates a pipeline error.

## Related tools

- **snapccess** (Python package supplying the ensemble embeddings on which clustering and ARI/NMI evaluation are run) — https://github.com/PYangLab/SnapCCESS
- **scikit-learn** (Provides k-means clustering and the adjusted_rand_score/normalized_mutual_info_score metrics)
- **CiteFuse** (Source of the spectral clustering implementation applied to the SnapCCESS embeddings)
- **SIMLR** (Third clustering algorithm coupled to the SnapCCESS embeddings for algorithm comparison)

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import adjusted_rand_score, normalized_mutual_info_score; pred = KMeans(n_clusters=n_types).fit_predict(embeddings); ari = adjusted_rand_score(ref_labels, pred)
```

## Evaluation signals

- ARI and NMI values fall in their expected concordance ranges (NMI in [0,1]; ARI near 0 for random labeling, up toward 1 for perfect recovery of reference cell types)
- Clustering performance (ARI/NMI) increases monotonically with ensemble size up to ~50, then plateaus — consistent with the paper's finding
- The trend holds for all three algorithms (k-means, spectral, SIMLR) and both concordance metrics, so cross-algorithm inconsistency flags a bug
- Aggregated CSV has one row per dataset × ensemble size × algorithm with both ARI and NMI columns, mirroring Figure 4b / Supplementary Fig. S5
- Reference label counts match the number of clusters requested for each dataset (cluster count derived from reference cell types)

## Limitations

- The SnapCCESS tutorial does not recommend best parameters for new datasets; parameters must be taken from the publication or tuned per dataset
- ARI/NMI require reference cell-type labels, so this skill cannot validate clustering on fully unlabeled data
- The article only recommends parameters for the datasets used in the published paper; behavior on other datasets/ensemble sizes is not covered
- Python package depends on PyTorch; GPU acceleration is optional and depends on the local PyTorch/CUDA or ROCm installation

## Evidence

- [intro] SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells.: "SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells."
- [readme] README description of snapshot embeddings used with downstream clustering: "which can then be used with downstream clustering
methods to generate consensus cell clusters."
- [methods] Python installation of snapccess: "pip install snapccess  --index-url https://pypi.org/simple"
- [readme] Tutorial parameter caveat: "This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets."
