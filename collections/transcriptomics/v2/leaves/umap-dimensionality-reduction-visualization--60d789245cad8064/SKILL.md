---
name: umap-dimensionality-reduction-visualization
description: Use when running SCASL on single-cell SMART-seq junction/AS matrices
  and you need a 2D visualization of spectral-clustering results on the imputed AS
  probability matrix (dimension_reduction_method='umap' in configs/srr.yaml).
when_to_use_negative:
- You only have a gene-expression matrix without junction reads — SCASL's AS-based
  UMAP requires junction-derived AS probability data, and gene-expression clustering
  will not resolve the same fine clusters.
- You need a deterministic, reproducible embedding — imputation, dimensionality reduction,
  and unsupervised clustering are stochastic, so run-to-run results differ.
- You need cell identities from splicing without any visualization — UMAP here is
  purely a 2D visualization layer; the clustering itself is spectral and does not
  depend on the UMAP output.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: SCASL
  role: Pipeline that performs QC, AS-event grouping, weighted KNN imputation, spectral
    clustering, and UMAP visualization of single-cell AS clusters
  repo: https://github.com/xryanglab/SCASL
- name: Leafcutter
  role: Extracts junction reads from single-cell alignments to build the junction
    reads matrix that underlies the AS probability matrix being visualized
  repo: https://github.com/davidaknowles/leafcutter
- name: umap-learn
  role: conda-installed Python library implementing the UMAP embedding selected via
    dimension_reduction_method='umap' in configs/srr.yaml
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:6144eaf71897eaa758fc5eb83e78b6f5b002ff3f573c088251317ffd06f4546f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xryanglab/SCASL
  tools:
  - Leafcutter
  - PyTorch
  - SCASL
  - TACTiCS
  - scPlant
  - umap-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# umap-dimensionality-reduction-visualization

## Summary

Use UMAP (via the umap-learn package) inside the SCASL pipeline to embed and visualize single-cell alternative splicing clusters computed on the imputed AS probability matrix, producing cluster-colored 2D plots (cluster.png) that expose fine subpopulations such as the C3 vs C4 tumor-infiltrating Treg split.

## When to use

Use when running SCASL on single-cell SMART-seq junction/AS matrices and you need a 2D visualization of spectral-clustering results on the imputed AS probability matrix (dimension_reduction_method='umap' in configs/srr.yaml).

## When NOT to use

- You only have a gene-expression matrix without junction reads — SCASL's AS-based UMAP requires junction-derived AS probability data, and gene-expression clustering will not resolve the same fine clusters.
- You need a deterministic, reproducible embedding — imputation, dimensionality reduction, and unsupervised clustering are stochastic, so run-to-run results differ.
- You need cell identities from splicing without any visualization — UMAP here is purely a 2D visualization layer; the clustering itself is spectral and does not depend on the UMAP output.

## Inputs

- normalized_matrix — imputed AS probability matrix (rows = sites, columns = cells)
- NA indicator matrix concatenated with the AS probability matrix for clustering
- configs/srr.yaml with pca_dimension, dimension_reduction_method='umap', cluster_algorithm='spectral', num_cluster
- optional label_file with run_column/truth_column for ground-truth-colored reduce.png

## Outputs

- cluster.png — UMAP embedding colored by SCASL AS-cluster labels
- reduce.png — UMAP embedding colored by user-supplied ground truth (if use_label=T)
- preds — predicted AS cluster label file

## How to apply

After Leafcutter extracts junction reads and builds junc_matrix.csv/junc_mat.npy, apply the QC filters (sites_initial, runs_initial, sites_quality, runs_quality), group junctions into AS events (filtered_matrix_start/end), and run the iterative weighted KNN imputation (num_iteration, knn) to obtain normalized_matrix. Spectral clustering (cluster_algorithm='spectral', num_cluster) is then performed on the imputed AS probability matrix concatenated with the NA indicator matrix; first reduce to pca_dimension principal components, then embed with umap-learn by setting dimension_reduction_method='umap' in configs/srr.yaml ('tsne' is the alternative). Run 'python main.py -y configs/srr.yaml'; the colored 2D embedding is written to cluster.png (colored by SCASL clustering) and reduce.png (colored by user-provided ground-truth labels if use_label=T). Because imputation and dimensionality reduction are stochastic, embeddings vary run-to-run, so judge correctness by general cluster consistency rather than exact cell positions.

## Related tools

- **SCASL** (Pipeline that performs QC, AS-event grouping, weighted KNN imputation, spectral clustering, and UMAP visualization of single-cell AS clusters) — https://github.com/xryanglab/SCASL
- **Leafcutter** (Extracts junction reads from single-cell alignments to build the junction reads matrix that underlies the AS probability matrix being visualized) — https://github.com/davidaknowles/leafcutter
- **umap-learn** (conda-installed Python library implementing the UMAP embedding selected via dimension_reduction_method='umap' in configs/srr.yaml)

## Examples

```
source activate scasl && python main.py -y configs/srr.yaml  # with dimension_reduction_method: 'umap' and cluster_algorithm: 'spectral' in configs/srr.yaml
```

## Evaluation signals

- cluster.png shows a UMAP embedding whose colors reproduce the expected fine AS-based clusters (e.g., C3 vs C4 Treg split; activated C0 vs exhausted C6/C9 CD8 clusters) for identical parameters
- preds cluster-label file is generated and its cluster count matches num_cluster set in configs/srr.yaml
- reduce.png (if use_label=T) aligns ground-truth labels with the embedding geometry, indicating the embedding respects known structure
- Upstream intermediates exist and are non-empty: junc_matrix.csv/junc_mat.npy, filtered_matrix_start, filtered_matrix_end, and normalized_matrix
- Re-running with identical parameters yields generally consistent cluster structure despite stochastic imputation/clustering

## Limitations

- Due to randomness of the imputation, dimensionality reduction and unsupervised clustering processes, run-to-run differences occur; results are only generally consistent.
- Single-cell AS profiles suffer from missing data points (dropouts), so the NA indicator matrix must be concatenated before clustering to handle missing values.
- Many canonical clustering methods are unsuitable for the largely polarized AS profiles, so the default spectral algorithm should be retained unless carefully re-tuned.
- Coverage of genes and AS events can be very unbalanced across different single cells, affecting embedding quality.

## Evidence

- [methods] UMAP visualization of spectral clustering in SCASL config: [evidence span withheld — anchor status: not_found]
- [readme] cluster.png is the UMAP output colored by clustering: "**cluster.png**: This figure represents the outcome of the final dimensionality reduction visualization, where the coloring is based on the clustering results of SCASL."
- [discussion] Clustering uses the imputed AS matrix plus NA indicator, followed by UMAP: "SCASL performs cell clustering with the full AS probability matrix, concatenated with a NA indicator matrix"
- [readme] Stochasticity of embedding/clustering steps: "Due to the randomness of the interpolation, dimensionality reduction and unsupervised clustering processes, there may be some differences between the running results"
- [readme] umap-learn installation dependency: "conda install -c conda-forge pandas pyyaml seaborn tqdm easydict umap-learn"
