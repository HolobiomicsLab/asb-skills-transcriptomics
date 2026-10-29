---
name: single-cell-splicing-landscape-clustering
description: Use when single-cell SMART-seq (bam or junction-file) data must be partitioned
  into cell subpopulations where gene-expression-based clustering fails, e.g. TNBC
  tumor cells, embryonic hepatoblasts, or heterogeneous tumor-associated CD4/CD8 T
  cells.
when_to_use_negative:
- Input is already a gene-expression count/feature table with no splice-junction or
  alignment-level information — SCASL requires junction reads from bam or junction
  files.
- Differential splicing between predefined cell groups is the goal — prior tools like
  BRIE, Expedition, rMATS, MARVEL require known cell labels, whereas SCASL is for
  unsupervised discovery.
- Droplet-based 10x data with very large cell numbers where SMART-seq-style splicing
  profiling is impractical — the article notes SMART-seq is more costly and difficult
  to multiplex for large cell numbers.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3320
- http://edamontology.org/topic_0203
tools:
- name: SCASL
  role: 'Main pipeline: junction-matrix construction, QC, AS probability inference,
    weighted KNN imputation, and spectral clustering of cells'
  repo: https://github.com/xryanglab/SCASL
- name: Leafcutter
  role: Extracts junction reads from single-cell alignments and supports de novo,
    annotation-free identification and grouping of AS events
  repo: https://github.com/davidaknowles/leafcutter
- name: samtools
  role: Conda-installed dependency used for alignment/junction processing in the scasl
    environment
provenance:
  source_task_ids:
  - task_005
  - task_008
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:3fc9cd2a37be84710d70f9c8f829ab2e3ef81ccb8fad71dbb4344a5bd9f72cf6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xryanglab/SCASL
  tools:
  - Leafcutter
  - SCASL
  - samtools
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3320
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# single-cell-splicing-landscape-clustering

## Summary

Unsupervised clustering of single cells based on alternative-splicing (AS) landscapes rather than gene expression, implemented in SCASL: de novo AS event identification from junction reads via Leafcutter-style grouping, AS probability inference, iterative weighted KNN imputation, and spectral clustering.

## When to use

Use when single-cell SMART-seq (bam or junction-file) data must be partitioned into cell subpopulations where gene-expression-based clustering fails, e.g. TNBC tumor cells, embryonic hepatoblasts, or heterogeneous tumor-associated CD4/CD8 T cells.

## When NOT to use

- Input is already a gene-expression count/feature table with no splice-junction or alignment-level information — SCASL requires junction reads from bam or junction files.
- Differential splicing between predefined cell groups is the goal — prior tools like BRIE, Expedition, rMATS, MARVEL require known cell labels, whereas SCASL is for unsupervised discovery.
- Droplet-based 10x data with very large cell numbers where SMART-seq-style splicing profiling is impractical — the article notes SMART-seq is more costly and difficult to multiplex for large cell numbers.

## Inputs

- Single-cell SMART-seq BAM alignment files (any mapper) or junction files (e.g. STAR SJ.out.tab)
- configs/srr.yaml parameter file (bam, lc, junction, QC, imputation, clustering settings)
- Optional demo intermediate junction files from TNBC-2 in data/junction
- Optional known-label file (use_label, label_file, truth_column, run_column)

## Outputs

- preds: final cluster label file (predicted AS cluster per cell)
- junc_matrix.csv / junc_mat.npy: integrated junction reads matrix (rows=sites, columns=cells)
- filtered_matrix_start / filtered_matrix_end: QC-filtered, AS-grouped junction matrices
- normalized_matrix: imputed AS probability matrix
- site_hist.png quality-control histograms
- reduce.png (colored by ground truth) and cluster.png (colored by SCASL clusters)

## How to apply

Configure configs/srr.yaml with the bam (or STAR SJ.out.tab junction) input paths, Leafcutter directory, and QC thresholds sites_initial/runs_initial (junction-matrix filters: minimum expressing cells per site, minimum expressing sites per cell) plus sites_quality/runs_quality (AS-matrix filters), imputation settings num_iteration and knn, pca_dimension, dimension_reduction_method ('umap' or 'tsne'), cluster_algorithm='spectral', and num_cluster. Run 'python main.py -y configs/srr.yaml' so SCASL builds the integrated junction reads matrix (junc_matrix.csv), groups junctions sharing 5'/3' splice sites into AS events (filtered_matrix_start/end), normalizes per-site AS probabilities (normalized_matrix), imputes missing values by iterative weighted KNN, and clusters cells using the full imputed AS probability matrix concatenated with a NA indicator matrix. Consult site_hist.png histograms to choose QC cutoffs; use the demo intermediate files (TNBC-2) in data/junction for a fast (<3 min) sanity test. Because imputation, dimensionality reduction, and clustering are stochastic, expect minor run-to-run differences but generally consistent clusters for identical parameters.

## Related tools

- **SCASL** (Main pipeline: junction-matrix construction, QC, AS probability inference, weighted KNN imputation, and spectral clustering of cells) — https://github.com/xryanglab/SCASL
- **Leafcutter** (Extracts junction reads from single-cell alignments and supports de novo, annotation-free identification and grouping of AS events) — https://github.com/davidaknowles/leafcutter
- **samtools** (Conda-installed dependency used for alignment/junction processing in the scasl environment)

## Examples

```
source activate scasl; python main.py -y configs/srr.yaml
```

## Evaluation signals

- preds file exists and contains a cluster label for every input cell, partitioning cells into the configured num_cluster groups
- Intermediate artifacts are produced in order: junc_matrix.csv, filtered_matrix_start/end, normalized_matrix, site_hist.png, reduce.png, cluster.png
- Clustering recapitulates paper-reported biology, e.g. a micrometastatic-enriched cluster among 3 TNBC clusters, hepatocyte ordering C2-C10-C3-C1-C8-C5, or the C3-vs-C4 Treg split in HCC tumor-associated T cells
- For robustness tests, SCASL normalized entropy under 50% forced dropout or 100/75/50% cell sub-sampling is lower (more stable) than Seurat gene-expression clustering
- Stochastic steps (imputation, dimensionality reduction, spectral clustering) may shift labels run-to-run, but results should be generally consistent for identical parameters

## Limitations

- Run-to-run variability: 'Due to the randomness of the interpolation, dimensionality reduction and unsupervised clustering processes, there may be some differences between the running results.'
- Single-cell AS profiles suffer from sparse, biased coverage, which is why iterative weighted KNN imputation and a NA indicator matrix are required.
- RNA splicing profiling is more costly than gene-expression quantification, and SMART-seq-style inputs are hard to multiplex for large cell numbers.
- The GitHub demo data are minimal ('very few') and only demonstrate operation and speed, not full biological reproduction.

## Evidence

- [intro] SCASL employs a strategy similar to that of LeafCutter22 and FRASER23 to identify AS events from the junction reads in single-cell SMART-seq data: "SCASL employs a strategy similar to that of LeafCutter22 and FRASER23 to identify AS events from the junction reads in single-cell SMART-seq data"
- [discussion] SCASL performs cell clustering with the full AS probability matrix, concatenated with a NA indicator matrix: "SCASL performs cell clustering with the full AS probability matrix, concatenated with a NA indicator matrix"
- [discussion] SCASL employs an iterative, weighted KNN process for imputation of the missing values: "SCASL employs an iterative, weighted KNN process for imputation of the missing values"
- [methods] sites_initial: quality control of the initial junction matrix, minimum number of expressing cells per site: "sites_initial: quality control of the initial junction matrix, minimum number of expressing cells per site"
- [methods] python main.py -y configs/srr.yaml: "python main.py -y configs/srr.yaml"
- [readme] The final output file is a cluster label file, preds represents the predicted AS cluster label.: "The final **output** file is a cluster label file, **preds** represents the predicted **AS cluster** label."
- [readme] Due to the randomness of the interpolation, dimensionality reduction and unsupervised clustering processes, there may be some differences between the running results: "Due to the randomness of the interpolation, dimensionality reduction and unsupervised clustering processes, there may be some differences between the running results"
