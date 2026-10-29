---
name: siamese-network-dimensionality-reduction
description: Use when visualizing or embedding high-dimensional single-cell expression
  data (CyTOF/scRNA-seq matrices) while needing to preserve global inter-cluster distances,
  or when new cells must be mapped into an existing embedding.
when_to_use_negative:
- When the analysis goal is only local neighbourhood preservation or cluster separation
  and global inter-cluster distances are irrelevant — t-SNE-style methods may be the
  paper's choice here
- When the data contain heavy random noise and spurious clean-boundary clusters would
  be problematic without the robustness ivis demonstrated only on the synthetic Smiley-type
  data
- When features are unscaled/untransformed raw cytometry intensities — arcsinh (X/5)
  and MinMaxScaler to [0,1] must be applied first or the triplet-loss training is
  invalid
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: ivis
  role: 'Primary implementation: siamese neural network trained with triplet loss
    to produce the structure-preserving low-dimensional embedding'
  repo: https://github.com/beringresearch/ivis
- name: t-SNE
  role: Baseline embedding method against which global-structure preservation (centroid-distance
    PCC) is compared
- name: Python
  role: Execution environment for ivis, preprocessing (arcsinh, MinMaxScaler), subsampling,
    Mantel test, and phenograph clustering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:9206fd5736d0c6b1abecfd7c87190495e1abf6619f6c10b7adc3480488022032
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  - t-SNE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# siamese-network-dimensionality-reduction

## Summary

A dimensionality-reduction skill in which a siamese neural network trained with a triplet loss maps high-dimensional single-cell expression data to a low-dimensional embedding that preserves both local and global structure. Grounded in the ivis algorithm (Szubert et al. 2019), which outperformed t-SNE on CyTOF benchmarks (median PCC 0.76 vs 0.53 on BMMC; 0.73 vs 0.14 on Samusik).

## When to use

Use when visualizing or embedding high-dimensional single-cell expression data (CyTOF/scRNA-seq matrices) while needing to preserve global inter-cluster distances, or when new cells must be mapped into an existing embedding.

## When NOT to use

- When the analysis goal is only local neighbourhood preservation or cluster separation and global inter-cluster distances are irrelevant — t-SNE-style methods may be the paper's choice here
- When the data contain heavy random noise and spurious clean-boundary clusters would be problematic without the robustness ivis demonstrated only on the synthetic Smiley-type data
- When features are unscaled/untransformed raw cytometry intensities — arcsinh (X/5) and MinMaxScaler to [0,1] must be applied first or the triplet-loss training is invalid

## Inputs

- High-dimensional single-cell expression matrix (e.g. CyTOF BMMC or Samusik marker expression, scRNA-seq count matrix, MNIST feature matrix)
- Optional manually annotated cell population (cluster) labels for benchmarking centroid-distance preservation
- Sparse matrices, numpy arrays, or hdf5 files as supported by ivis

## Outputs

- 2D (or user-defined dimensional) embedding coordinates per cell
- Per-subsample and median PCC and ARI summary (e.g. pcc_ari_benchmark_results.csv)
- Trained parametric mapping reusable via transform() for embedding new data points

## How to apply

Preprocess the expression matrix with arcsinh transformation (X = np.arcsinh(X/5)) and then scale to the [0, 1] range with MinMaxScaler, since ivis requires scaled inputs. Fit a siamese neural network via Ivis(embedding_dims=2, k=15) using default triplet-loss settings (model='maaten'), sampling positives from the KNN vector and negatives from outside it. For benchmarking, draw random subsamples (e.g. 100 subsamples of 10,000 cells), build Euclidean distance matrices between annotated cluster centroids in the original and embedding spaces, and assess concordance with a Mantel test using Pearson correlation coefficient (PCC). The trained parametric mapping also supports adding new data points to the embedding via the transform method without refitting. Optionally cluster the embedding with phenograph and quantify agreement against manual labels with Adjusted Rand Index (reported ARI: 0.97 BMMC, 0.45 Samusik).

## Related tools

- **ivis** (Primary implementation: siamese neural network trained with triplet loss to produce the structure-preserving low-dimensional embedding) — https://github.com/beringresearch/ivis
- **t-SNE** (Baseline embedding method against which global-structure preservation (centroid-distance PCC) is compared)
- **Python** (Execution environment for ivis, preprocessing (arcsinh, MinMaxScaler), subsampling, Mantel test, and phenograph clustering)

## Examples

```
from ivis import Ivis; from sklearn.preprocessing import MinMaxScaler; X_scaled = MinMaxScaler().fit_transform(np.arcsinh(X/5)); embeddings = Ivis(embedding_dims=2, k=15).fit_transform(X_scaled)
```

## Evaluation signals

- Median PCC from the Mantel test between original and embedding centroid-distance matrices reaches the reported ranges (ivis BMMC 0.76, ivis Samusik 0.73), exceeding t-SNE baselines (0.53 and 0.14)
- Phenograph clustering of the embedding against manual labels yields ARI near reported medians (0.97 BMMC, 0.45 Samusik)
- All embedding input matrices are bounded in [0, 1] after MinMaxScaler, confirming preprocessing invariants
- model.transform() successfully embeds held-out/new cells without refitting, demonstrating the parametric mapping generalises
- Embedding dimensionality matches embedding_dims=2 and each row corresponds to an input cell

## Limitations

- Reported hyperparameter robustness is referenced via Supplementary Figures rather than explicit thresholds, so practitioners must verify settings for their own data
- Compared software (t-SNE, SIMLR, scvis, DeepCyTOF) versions are not stated in the discussion, limiting reproducibility of the head-to-head comparison
- In the presence of uniform random noise, t-SNE yields spurious clusters with clean boundaries — a failure mode ivis was only demonstrated to avoid on synthetic data

## Evidence

- [abstract] ivis utilizes a siamese neural network architecture that is trained using a novel triplet loss function.: "ivis utilizes a siamese neural network architecture that is trained using a novel triplet loss function."
- [methods] Scaling to [0,1] is required preprocessing: "Prior to preprocessing, values in both datasets were scaled to [0, 1] range."
- [methods] Mantel test with Pearson correlation assesses distance-matrix concordance: "The level of correlation between the original distance matrix and the distance matrices in the embedding spaces was then assessed using the `Mantel test"
- [discussion] Triplet sampling scheme from KNN and non-KNN points: "ivis samples positive and negative data-points for each triplet, from the KNN vector and outside the KNN vector respectively"
- [readme] Concrete ivis invocation from the repository README: "model = Ivis(embedding_dims=2, k=15)"
- [readme] ivis outperforms t-SNE at preserving global structure: [evidence span withheld — anchor status: external_doc]
