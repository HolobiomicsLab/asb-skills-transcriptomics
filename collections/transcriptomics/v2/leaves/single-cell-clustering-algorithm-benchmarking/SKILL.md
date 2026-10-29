---
name: single-cell-clustering-algorithm-benchmarking
description: Use when you have precomputed cell embeddings (e.g. SnapCCESS snapshots
  across ensemble sizes) for multimodal single-cell data and need to compare clustering
  algorithms against known cell-type labels using ARI and NMI.
when_to_use_negative:
- You only need a single clustering result on one embedding — a full algorithm/ensemble-size
  benchmark is overkill.
- No reference cell-type labels exist, so ARI/NMI against ground truth cannot be computed.
- The input is raw RNA/ADT count matrices rather than already-generated embeddings;
  generate SnapCCESS embeddings first.
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3577
tools:
- name: SnapCCESS
  role: Generates the multimodal embedding snapshots that are the input to each clustering
    algorithm at each ensemble size
  repo: https://github.com/PYangLab/SnapCCESS
- name: snapccess
  role: Python package used to load the previously generated embeddings for each dataset
    and ensemble-size setting
  repo: https://github.com/yulijia/SnapCCESS/tree/main/snapccess-py
- name: k-means (scikit-learn)
  role: First clustering algorithm benchmarked on the SnapCCESS embeddings
- name: CiteFuse spectral clustering
  role: Second clustering algorithm benchmarked, as implemented in CiteFuse
- name: SIMLR
  role: Third clustering algorithm benchmarked on the same embeddings
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad382
    title: Ensemble deep learning of embeddings for clustering multimodal single-cell
      omics data
  source_content_hash: sha256:d1e64a6b3c3d2b35fde079316079cf9273e9273476c231ef9c673d790e6e6ae9
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
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-clustering-algorithm-benchmarking

## Summary

Benchmarks how multiple clustering algorithms (k-means, CiteFuse spectral clustering, and SIMLR) perform when coupled with SnapCCESS embeddings of multimodal single-cell omics data, tracking whether performance improves with ensemble size and comparing ARI/NMI against reference cell-type labels.

## When to use

Use when you have precomputed cell embeddings (e.g. SnapCCESS snapshots across ensemble sizes) for multimodal single-cell data and need to compare clustering algorithms against known cell-type labels using ARI and NMI.

## When NOT to use

- You only need a single clustering result on one embedding — a full algorithm/ensemble-size benchmark is overkill.
- No reference cell-type labels exist, so ARI/NMI against ground truth cannot be computed.
- The input is raw RNA/ADT count matrices rather than already-generated embeddings; generate SnapCCESS embeddings first.

## Inputs

- SnapCCESS embeddings per dataset and ensemble size (from the snapccess Python package)
- Reference cell-type labels for each dataset (e.g. CITE-seq RNA/ADT cell type files)
- List of clustering algorithms to compare: k-means (scikit-learn), CiteFuse spectral clustering, SIMLR

## Outputs

- ARI and NMI scores per algorithm, dataset, and ensemble size
- Aggregated comparison table mirroring Figure 4b / Supplementary Fig. S5 (e.g. algorithm_comparison_ari_nmi.csv)

## How to apply

Load the SnapCCESS embeddings generated for each dataset at each ensemble size via the snapccess Python package. Run k-means clustering (scikit-learn), spectral clustering as implemented in CiteFuse, and SIMLR clustering on the same embeddings for every dataset and ensemble-size setting. Compute Adjusted Rand Index (ARI) and Normalized Mutual Information (NMI) of each algorithm's clustering against reference cell-type labels. Aggregate scores into a table mirroring the paper's Figure 4b / Supplementary Fig. S5 (e.g. algorithm_comparison_ari_nmi.csv) and inspect the trend across ensemble sizes; in the source study, improvement peaked around an ensemble size of 50 regardless of clustering algorithm or concordance metric.

## Related tools

- **SnapCCESS** (Generates the multimodal embedding snapshots that are the input to each clustering algorithm at each ensemble size) — https://github.com/PYangLab/SnapCCESS
- **snapccess** (Python package used to load the previously generated embeddings for each dataset and ensemble-size setting) — https://github.com/yulijia/SnapCCESS/tree/main/snapccess-py
- **k-means (scikit-learn)** (First clustering algorithm benchmarked on the SnapCCESS embeddings)
- **CiteFuse spectral clustering** (Second clustering algorithm benchmarked, as implemented in CiteFuse)
- **SIMLR** (Third clustering algorithm benchmarked on the same embeddings)

## Examples

```
pip install snapccess --index-url https://pypi.org/simple
```

## Evaluation signals

- ARI and NMI are computed against reference cell-type labels for every algorithm, dataset, and ensemble-size combination.
- Clustering performance should increase monotonically or near-monotonically with ensemble size before plateauing; the paper observed the peak around an ensemble size of 50.
- The trend should hold across all three algorithms (k-means, spectral, SIMLR) and both concordance metrics (ARI, NMI) — a result specific to one algorithm suggests a pipeline error.
- The aggregated output table should structurally mirror Figure 4b / Supplementary Fig. S5 (rows/dimensions covering algorithm x dataset x ensemble size).
- All three algorithms must run on identical embeddings per setting, ensuring differences reflect the clustering method rather than input variation.

## Limitations

- The package tutorial does not recommend best hyperparameters for new datasets; parameters used for the published datasets are only listed in the paper, so the user must consult the paper to guide parameter selection.
- Improvement saturates: the paper observed the increase in clustering performance peaking around an ensemble size of 50, so much larger ensembles add little.
- GPU acceleration is optional and depends on the local PyTorch/CUDA or PyTorch/ROCm installation, which can complicate reproducibility across environments.

## Evidence

- [results] A clear increase in clustering performance was observed with increasing ensemble size, regardless of the clustering algorithm used (k-means, spectral, SIMLR) and the concordance metric, with improvement peaking around an ensemble size of 50.: [evidence span withheld — anchor status: not_found]
- [methods] workflow-step grounding: "SnapCCESS can be coupled with various clustering algorithms for generating consensus clustering of cells."
- [readme] parameter caveat: "This tutorial only explains how to use this package; it doesn't recommend the best parameters for your datasets."
- [readme] installation: "pip install snapccess --index-url https://pypi.org/simple"
- [abstract] metric context: "SnapCCESS is effective and more efficient than conventional ensemble deep learning-based clustering methods and outperforms other state-of-the-art multimodal embedding generation methods"
