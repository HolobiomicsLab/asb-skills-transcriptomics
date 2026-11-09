---
name: single-cell-alternative-splicing-event-identification
description: Use when input is single-cell SMART-seq BAM (or junction/SJ.out.tab)
  files and the goal is an unsupervised, annotation-free AS-based view of cell identity
  that gene-expression clustering misses.
when_to_use_negative:
- When the analysis goal is gene-expression-based cell clustering or quantification
  of expression levels rather than RNA splicing landscapes
- When the input is already an AS probability/feature matrix — SCASL expects BAM or
  junction files as initial input (intermediate files are only supported for step-by-step
  runs)
- When a differential splicing method with known cell labels is needed (e.g. BRIE,
  Expedition, rMATS, Psix); SCASL is for unsupervised clustering without predefined
  labels
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SCASL
  role: 'Main pipeline: identifies AS events de novo from junction reads, builds and
    filters the junction matrix, infers AS probabilities, and concatenates the AS
    probability matrix with an NA indicator matrix for clustering'
  repo: github.com/xryanglab/SCASL
- name: Leafcutter
  role: Annotation-free identification of AS events from spliced junction reads, called
    by SCASL during junction matrix construction; installable via conda or from source
  repo: github.com/davidaknowles/leafcutter
- name: scikit-learn
  role: Provides the spectral clustering (default), PCA, and alternative clustering
    algorithms (agglomerative, kmeans) applied to the imputed AS probability matrix
- name: umap-learn
  role: 'Dimension-reduction visualization of clusters (dimension_reduction_method:
    ''umap'') producing reduce.png and cluster.png'
- name: samtools
  role: Installed via bioconda to support BAM-file handling when BAM files are the
    initial input
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-46480-9
    title: Interrogations of single-cell RNA splicing landscapes with SCASL define
      new cell identities with physiological relevance
  source_content_hash: sha256:0b3db26597e8545bb941a26e0d2f4a877a9e8f6d87f89120cc8ed690c56da636
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Leafcutter
  - SCASL
  - samtools
  - scikit-learn
  - umap-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# single-cell-alternative-splicing-event-identification

## Summary

De novo identification and quantification of alternative splicing (AS) events from single-cell RNA-seq junction reads using SCASL, which integrates junctions into a site-by-cell matrix (Leafcutter-style), infers AS probabilities, and prepares the matrix for downstream imputation and clustering.

## When to use

Use when input is single-cell SMART-seq BAM (or junction/SJ.out.tab) files and the goal is an unsupervised, annotation-free AS-based view of cell identity that gene-expression clustering misses.

## When NOT to use

- When the analysis goal is gene-expression-based cell clustering or quantification of expression levels rather than RNA splicing landscapes
- When the input is already an AS probability/feature matrix — SCASL expects BAM or junction files as initial input (intermediate files are only supported for step-by-step runs)
- When a differential splicing method with known cell labels is needed (e.g. BRIE, Expedition, rMATS, Psix); SCASL is for unsupervised clustering without predefined labels

## Inputs

- Aligned single-cell SMART-seq BAM files (any mapping method), or pre-extracted junction files / STAR SJ.out.tab files
- configs/srr.yaml configuration file (bam, lc, junction, QC thresholds, imputation and clustering parameters)

## Outputs

- junc_matrix.csv / junc_mat.npy — junction reads matrix (rows = sites, columns = cells)
- filtered_matrix_start / filtered_matrix_end — QC-filtered junction matrices grouped by upstream/downstream AS events
- normalized_matrix — normalized AS probability matrix (rows = sites, columns = cells)
- site_hist.png / sample histograms of non-NA counts to guide QC parameter selection
- Final cluster label file in which preds gives the predicted AS-based cluster label, plus cluster.png and reduce.png visualizations

## How to apply

Set up the scasl conda environment (conda-forge for python 3.9/scikit-learn plus pandas, pyyaml, seaborn, tqdm, easydict, umap-learn; davidaknowles channel for r-leafcutter; bioconda for samtools). Edit configs/srr.yaml to point the bam (or junction) path at the aligned scRNA-seq input and configure Leafcutter's directory (lc) and output path. Run 'python main.py -y configs/srr.yaml' so SCASL performs de novo identification of AS events from junction reads in single-cell SMART-seq data without relying on predefined transcriptome annotation, integrating junction reads into a junction matrix (rows = sites, columns = cells). Group filtered sites into AS events by upstream/downstream grouping (filtered_matrix_start / filtered_matrix_end), then normalize to yield the AS probability matrix (normalized_matrix). Apply QC thresholds (sites_initial, runs_initial, sites_quality, runs_quality) via YAML; optionally set log_scale_histogram to T to inspect site/sample count distributions before choosing filters. Downstream, impute missing values (dropouts are pervasive in single-cell AS profiles) with the iterative weighted KNN procedure (num_iteration, knn) and cluster the imputed matrix (spectral by default after PCA at pca_dimension).

## Related tools

- **SCASL** (Main pipeline: identifies AS events de novo from junction reads, builds and filters the junction matrix, infers AS probabilities, and concatenates the AS probability matrix with an NA indicator matrix for clustering) — github.com/xryanglab/SCASL
- **Leafcutter** (Annotation-free identification of AS events from spliced junction reads, called by SCASL during junction matrix construction; installable via conda or from source) — github.com/davidaknowles/leafcutter
- **scikit-learn** (Provides the spectral clustering (default), PCA, and alternative clustering algorithms (agglomerative, kmeans) applied to the imputed AS probability matrix)
- **umap-learn** (Dimension-reduction visualization of clusters (dimension_reduction_method: 'umap') producing reduce.png and cluster.png)
- **samtools** (Installed via bioconda to support BAM-file handling when BAM files are the initial input)

## Examples

```
source activate scasl; python main.py -y configs/srr.yaml
```

## Evaluation signals

- The run completes and the final cluster label file exists with preds populated (predicted AS cluster labels)
- Normalized AS probability matrix (rows = sites, columns = cells) and grouped filtered_matrix_start/filtered_matrix_end files are generated after QC
- QC histograms (site_hist.png) show non-NA site/sample distributions consistent with the chosen sites_initial/runs_initial/sites_quality/runs_quality thresholds
- Reruns with identical parameters yield generally consistent clustering despite randomness in imputation, dimensionality reduction, and unsupervised clustering
- Demo intermediate files (TNBC-2) in data/junction complete in under ~3 minutes with results stored in process_result_demo

## Limitations

- Single-cell AS profiles suffer greatly from missing data points due to dropouts during scRNA-seq analysis and AS event identification, requiring iterative weighted KNN imputation
- Coverage of genes and AS events can be very unbalanced across different single cells, and many canonical clustering methods are unsuitable for processing the largely polarized AS profiles
- SMART-seq-based input makes splicing profiling more costly and difficult to multiplex for large cell numbers than droplet-based scRNA-seq
- Results may differ between runs due to randomness in imputation, dimensionality reduction, and unsupervised clustering; demo data are minimal and only illustrate operation and speed
- If Leafcutter cannot be installed via conda, the user must clone the source and manually add it to PATH

## Evidence

- [intro] SCASL employs a strategy similar to that of LeafCutter22 and FRASER23 to identify AS events from the junction reads in single-cell SMART-seq data: "SCASL employs a strategy similar to that of LeafCutter22 and FRASER23 to identify AS events from the junction reads in single-cell SMART-seq data"
- [methods] python main.py -y configs/srr.yaml: "python main.py -y configs/srr.yaml"
- [readme] sites_initial: quality control of the initial junction matrix, minimum number of expressing cells per site: "sites_initial: quality control of the initial junction matrix, minimum number of expressing cells per site"
- [readme] The final AS probability matrix, the rows correspond to sites, while the columns represent cells.: "**normalized_matrix**: The final AS probability matrix, the rows correspond to sites, while the columns represent cells."
- [discussion] SCASL employs an iterative, weighted KNN process for imputation of the missing values: "SCASL employs an iterative, weighted KNN process for imputation of the missing values"
