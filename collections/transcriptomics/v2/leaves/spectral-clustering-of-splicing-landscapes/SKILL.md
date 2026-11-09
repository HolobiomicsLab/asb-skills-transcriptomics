---
name: spectral-clustering-of-splicing-landscapes
description: Use when the goal is to define unsupervised cell subpopulations from
  single-cell RNA-seq junction/AS data (e.g., SMART-seq BAMs), especially where gene-expression
  clustering fails to separate transitional states such as precancerous epithelium.
when_to_use_negative:
- The analysis goal is differential splicing quantification with known cell labels
  (tools like BRIE, Expedition, rMATS, Psix already cover that); SCASL is designed
  for unsupervised clustering without predefined labels.
- Input is already a gene-expression feature table (e.g. counts matrix) with no junction/alignable
  read information — SCASL requires junction reads or BAMs from SMART-seq-like data.
- 'Data are droplet-based scRNA-seq with sparse coverage unsuited to AS-event detection:
  the article notes SMART-seq-like data are used, and RNA splicing profiling coverage
  can be very unbalanced and costly across cells.'
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
tools:
- name: SCASL
  role: 'Main pipeline: integrates junction reads, infers AS probabilities, imputes
    NAs, and performs the spectral clustering of the AS landscape'
  repo: github.com/xryanglab/SCASL
- name: Leafcutter
  role: De novo identification of AS events from junction reads feeding the junction
    matrix used for clustering
  repo: github.com/davidaknowles/leafcutter
- name: scikit-learn
  role: Provides the spectral (default), agglomerative, and kmeans clustering algorithms
    and PCA used on the imputed AS probability matrix
- name: umap-learn
  role: 'Provides UMAP (alternative: tsne) for dimensionality-reduction visualization
    of the clustering output'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:03b275ea4c0b5572e3bf4ecafc65316c392c587d07f5f99b229884dae2c6016d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Leafcutter
  - SCASL
  - scikit-learn
  - umap-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# spectral-clustering-of-splicing-landscapes

## Summary

Unsupervised spectral clustering of single cells based on their alternative splicing (AS) probability landscape, as implemented in SCASL. It assigns cells to AS-defined clusters after Leafcutter-based AS event identification, quality filtering, and iterative weighted KNN imputation, revealing cell states that gene-expression clustering misses.

## When to use

Use when the goal is to define unsupervised cell subpopulations from single-cell RNA-seq junction/AS data (e.g., SMART-seq BAMs), especially where gene-expression clustering fails to separate transitional states such as precancerous epithelium.

## When NOT to use

- The analysis goal is differential splicing quantification with known cell labels (tools like BRIE, Expedition, rMATS, Psix already cover that); SCASL is designed for unsupervised clustering without predefined labels.
- Input is already a gene-expression feature table (e.g. counts matrix) with no junction/alignable read information — SCASL requires junction reads or BAMs from SMART-seq-like data.
- Data are droplet-based scRNA-seq with sparse coverage unsuited to AS-event detection: the article notes SMART-seq-like data are used, and RNA splicing profiling coverage can be very unbalanced and costly across cells.

## Inputs

- Single-cell RNA-seq BAM file (any mapping method, e.g. SMART-seq) or junction file (e.g. STAR SJ.out.tab)
- configs/srr.yaml configuration file (bam, lc, junction, QC thresholds, num_iteration, knn, pca_dimension, cluster_algorithm, num_cluster)
- Optional demo intermediate junction files (from TNBC-2) in data/junction
- Optional known label file (use_label, label_file, truth_column, run_column)

## Outputs

- Cluster label file in which preds represents the predicted AS cluster label per cell
- cluster.png and reduce.png dimensionality-reduction visualizations (umap or tsne)
- junc_matrix.csv / junc_mat.npy junction reads matrix (rows = sites, columns = cells)
- filtered_matrix_start / filtered_matrix_end AS-grouped filtered matrices
- normalized_matrix: final AS probability matrix (rows = sites, columns = cells)
- site/sample histograms (e.g. site_hist.png) to assist filter-parameter selection

## How to apply

Run SCASL end-to-end via 'python main.py -y configs/srr.yaml' with the scasl conda environment (python 3.9, scikit-learn via conda-forge, Leafcutter via the davidaknowles channel, samtools via bioconda). SCASL integrates junction reads from BAM (or SJ.out.tab) input into a junction matrix (rows = sites, columns = cells), applies Leafcutter-based de novo AS event identification, groups sites into AS events (filtered_matrix_start/filtered_matrix_end by upstream/downstream grouping), and normalizes to an AS probability matrix. Apply the configured QC filters (sites_initial, runs_initial, sites_quality, runs_quality) on the junction and AS matrices, optionally enabling log_scale_histogram (T/F) to inspect site/sample non-NA distributions and choose thresholds. Impute missing values (dropout-driven NAs) with the iterative weighted KNN procedure controlled by num_iteration and knn, then cluster the imputed matrix, concatenated with an NA indicator matrix, after PCA at pca_dimension using cluster_algorithm: 'spectral' (alternatives: agglomerative, kmeans) with num_cluster set. The rationale for spectral clustering is that many canonical clustering methods are unsuitable for the largely polarized AS profiles; set num_cluster thoughtfully or set max_n_cluster in cluster.py to obtain clustering scores that help judge the most appropriate number of clusters. Visualize with dimension_reduction_method ('umap' or 'tsne') to generate cluster.png (colored by SCASL clusters) and reduce.png.

## Related tools

- **SCASL** (Main pipeline: integrates junction reads, infers AS probabilities, imputes NAs, and performs the spectral clustering of the AS landscape) — github.com/xryanglab/SCASL
- **Leafcutter** (De novo identification of AS events from junction reads feeding the junction matrix used for clustering) — github.com/davidaknowles/leafcutter
- **scikit-learn** (Provides the spectral (default), agglomerative, and kmeans clustering algorithms and PCA used on the imputed AS probability matrix)
- **umap-learn** (Provides UMAP (alternative: tsne) for dimensionality-reduction visualization of the clustering output)

## Examples

```
source activate scasl; python main.py -y configs/srr.yaml
```

## Evaluation signals

- The run finishes in the expected time (README: demo test < 3 minutes; environment setup < 15 minutes) and the final cluster label file with preds exists.
- Reruns with identical parameters yield generally consistent clustering; only moderate differences arise from the randomness of imputation, dimensionality reduction, and unsupervised clustering.
- cluster.png coloring reflects SCASL AS clusters and reduce.png reflects user ground-truth labels, allowing visual comparison of AS-defined vs. expression-defined groupings.
- QC filter distributions (sites_initial/runs_initial/sites_quality/runs_quality) are sanity-checked via the site/sample non-NA histograms before locking thresholds.
- Biologically, AS-defined clusters should recover states missed by gene-expression clustering, e.g. the precancerous transitional C3 epithelial cluster in the 443-cell TNBC dataset.

## Limitations

- Single-cell AS profiles suffer greatly from missing data points due to dropouts during scRNA-seq and AS event identification, requiring KNN imputation before clustering.
- Coverage of genes and AS events can be very unbalanced across different single cells, and many canonical clustering methods are unsuitable for the largely polarized AS profiles.
- Due to randomness in imputation, dimensionality reduction, and unsupervised clustering, results may differ between runs even with identical parameters.
- The GitHub demo files are very few and only show operation and speed; the method was developed/tested on SMART-seq-like data, which is costlier and harder to multiplex than droplet-based scRNA-seq.

## Evidence

- [intro] Spectral clustering of the cells based on the imputed AS probability matrix: "followed by spectral clustering of the cells"
- [discussion] AS probability matrix construction from junction reads: "Taking aligned RNA-seq files as input, SCASL first performs de novo identification of AS events for each single cell"
- [readme] Run command: "python main.py -y configs/srr.yaml"
- [readme] Default clustering algorithm and options: "cluster_algorithm: 'spectral' by default. ('spectral', 'agglomerative', 'kmeans'. )"
- [readme] Output preds meaning: "The final output file is a cluster label file, preds represents the predicted AS cluster label."
- [discussion] Missingness motivating imputation: "single-cell AS profiles suffer greatly from missing data points due to dropouts during single-cell RNA sequencing analysis and the identification of AS events"
