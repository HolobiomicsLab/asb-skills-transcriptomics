---
name: spectral-clustering-of-as-matrices
description: Use when single-cell RNA-seq junction/AS matrices are QC-filtered and
  imputed and the goal is unsupervised discovery of splicing-defined cell subpopulations
  that gene-expression clustering misses.
when_to_use_negative:
- The input is raw junction reads or an unfiltered/unimputed junction matrix — QC
  and iterative weighted KNN imputation must precede spectral clustering
- The goal is differential splicing analysis with known cell labels (use BRIE, Expedition,
  rMATS, or MARVEL instead); SCASL is designed for unsupervised clustering without
  predefined labels
- Very few cells or extremely sparse junction coverage make the demo caveat relevant
  — run-to-run stochasticity may dominate the clustering result
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: SCASL
  role: Implements the spectral clustering on the imputed AS probability matrix plus
    NA indicator matrix via main.py with the YAML config
  repo: github.com/xryanglab/SCASL
- name: Leafcutter
  role: Upstream tool that extracts junction reads from single-cell BAM files and
    builds the junction reads matrix that feeds QC, AS grouping, imputation, and clustering
  repo: github.com/davidaknowles/leafcutter
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:feff82c57f0382584417053efd4f913074afbf4280f88a8fb07130d28bfbe4ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Leafcutter
  - SCASL
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# spectral-clustering-of-as-matrices

## Summary

SCASL's spectral clustering step groups single cells by their imputed alternative splicing (AS) probability matrix, concatenated with an NA indicator matrix, revealing subpopulations (e.g., C3 vs C4 tumor-infiltrating Tregs) invisible to gene-expression-based clustering.

## When to use

Use when single-cell RNA-seq junction/AS matrices are QC-filtered and imputed and the goal is unsupervised discovery of splicing-defined cell subpopulations that gene-expression clustering misses.

## When NOT to use

- The input is raw junction reads or an unfiltered/unimputed junction matrix — QC and iterative weighted KNN imputation must precede spectral clustering
- The goal is differential splicing analysis with known cell labels (use BRIE, Expedition, rMATS, or MARVEL instead); SCASL is designed for unsupervised clustering without predefined labels
- Very few cells or extremely sparse junction coverage make the demo caveat relevant — run-to-run stochasticity may dominate the clustering result

## Inputs

- Imputed single-cell AS probability matrix (normalized_matrix; rows = AS sites, columns = cells)
- NA position indicator matrix derived from the imputation step
- configs/srr.yaml with cluster_algorithm='spectral', pca_dimension, num_cluster, dimension_reduction_method settings

## Outputs

- Cluster label file whose 'preds' column assigns each cell an AS-based cluster label
- cluster.png UMAP/t-SNE visualization colored by SCASL clustering results
- Optional clustering scores from max_n_cluster to judge the optimal number of clusters

## How to apply

After quality-control filters (sites_initial, runs_initial, sites_quality, runs_quality) and iterative weighted KNN imputation yield the AS probability matrix (normalized_matrix), run SCASL via 'python main.py -y configs/srr.yaml' with cluster_algorithm='spectral' in configs/srr.yaml. The spectral clustering is performed on the full AS probability matrix concatenated with a NA indicator matrix, so cells with missing values remain clusterable; set pca_dimension for PCA and num_cluster for the expected number of clusters. Visualize the result with dimension_reduction_method='umap' (or 'tsne') producing cluster.png, and extract the predicted labels from the output file where 'preds' holds the AS cluster label per cell. Alternative algorithms ('agglomerative', 'kmeans') and max_n_cluster scoring in cluster.py can be used to sanity-check the chosen cluster number, but spectral is the article's default because canonical methods are unsuitable for the largely polarized AS profiles.

## Related tools

- **SCASL** (Implements the spectral clustering on the imputed AS probability matrix plus NA indicator matrix via main.py with the YAML config) — github.com/xryanglab/SCASL
- **Leafcutter** (Upstream tool that extracts junction reads from single-cell BAM files and builds the junction reads matrix that feeds QC, AS grouping, imputation, and clustering) — github.com/davidaknowles/leafcutter

## Examples

```
source activate scasl; python main.py -y configs/srr.yaml
```

## Evaluation signals

- The output 'preds' labels reproduce the reported fine AS-based clusters for the same parameters (e.g., C3 vs C4 Treg split; activated C0 vs exhausted C6/C9 CD8 clusters in HCC tumor-associated T cells); minor run-to-run differences are acceptable but results should be generally consistent
- cluster.png shows a UMAP embedding whose coloring matches the preds labels and resolves distinct subpopulations
- Clustering scores emitted via max_n_cluster in cluster.py support the chosen num_cluster value
- The normalized_matrix feeding clustering is a complete AS probability matrix (missing values filled by iterative weighted KNN imputation) after passing all four QC filters
- AS-based clusters differ from gene-expression-based clustering results, confirming the splicing layer adds resolution (most findings are not available via conventional clustering)

## Limitations

- The imputation, dimensionality reduction, and unsupervised clustering steps are stochastic, so run-to-run differences occur; results are only 'generally consistent' for identical parameters
- Many canonical clustering methods are unsuitable for processing the largely polarized AS profiles — spectral clustering with the NA indicator matrix is specifically required
- RNA splicing profiling is much more costly than gene expression quantification, and SMART-seq-style data is difficult to multiplex for large cell numbers
- Coverage of genes and AS events can be very unbalanced across single cells, creating dropout-driven NA values that must be imputed before clustering

## Evidence

- [discussion] SCASL performs cell clustering with the full AS probability matrix, concatenated with a NA indicator matrix: "SCASL performs cell clustering with the full AS probability matrix, concatenated with a NA indicator matrix"
- [intro] spectral clustering of the cells based on the imputed AS probability matrix: "followed by spectral clustering of the cells"
- [discussion] many of the canonical clustering methods are unsuitable for processing the largely polarized AS profiles: "many of the canonical clustering methods are unsuitable for processing the largely polarized AS profiles"
- [readme] preds represents the predicted AS cluster label: "The final **output** file is a cluster label file, **preds** represents the predicted **AS cluster** label."
- [readme] cluster.png colored by clustering results: "**cluster.png**: This figure represents the outcome of the final dimensionality reduction visualization, where the coloring is based on the clustering results of SCASL."
- [readme] cluster_algorithm default 'spectral': "**cluster_algorithm**: 'spectral' by default. ('spectral', 'agglomerative', 'kmeans'. )"
