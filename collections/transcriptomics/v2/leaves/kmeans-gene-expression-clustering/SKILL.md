---
name: kmeans-gene-expression-clustering
description: Use when building BIRD-style locus-level regression models where thousands
  of co-expressed genes must be reduced to a manageable set of compressed predictors
  for predicting DNase I hypersensitivity (DH) from RNA-seq.
when_to_use_negative:
- Input is not a gene expression matrix (e.g., you only have DNase-seq or ChIP-seq
  data with no matched expression training samples)
- You want to discover new DHS locations — BIRD only predicts DH at the fixed loci
  defined by the model
- You need to run prediction with a prebuilt model only — BIRD_predict with a released
  human_hg38_model.bin or Exonarray_model_file.bin already embeds the trained clusters,
  so re-clustering is unnecessary
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3517
tools:
- name: BIRD
  role: Provides the modeling framework and R-backed code in which genes are k-means
    clustered and the locus-level model BIR(X-bar, Y) is fitted
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD_predict
  role: Loads the serialized model and runs prediction; the -l flag invokes the locus-level
    model built from the clustered predictors
  repo: https://github.com/WeiqiangZhou/BIRD
- name: R
  role: Required runtime (Rscript must be executable) used to load training data,
    perform the clustering, and fit the per-locus regressions
  repo: https://www.r-project.org
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:b4e71744b4273234d84be3c367c72f03ae72ad61baab100142d4cd48905c461c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WeiqiangZhou/BIRD
  tools:
  - BIRD
  - BIRD_predict
  - R
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# kmeans-gene-expression-clustering

## Summary

Compresses the genome-wide gene expression predictor set by k-means clustering of co-expressed genes (K=1500 in BIRD) and computing per-cluster average expression (X-bar), which reduces predictor dimension, mitigates co-linearity, and makes predictors less sensitive to measurement noise before fitting locus-level DH regression models.

## When to use

Use when building BIRD-style locus-level regression models where thousands of co-expressed genes must be reduced to a manageable set of compressed predictors for predicting DNase I hypersensitivity (DH) from RNA-seq.

## When NOT to use

- Input is not a gene expression matrix (e.g., you only have DNase-seq or ChIP-seq data with no matched expression training samples)
- You want to discover new DHS locations — BIRD only predicts DH at the fixed loci defined by the model
- You need to run prediction with a prebuilt model only — BIRD_predict with a released human_hg38_model.bin or Exonarray_model_file.bin already embeds the trained clusters, so re-clustering is unnecessary

## Inputs

- ENCODE RNA-seq training expression matrix (FPKM, gene ensembl ids as first column) from BIRD-data (e.g., BIRD_data_ENCODE.zip)
- DNase-seq DH training data (200bp-window loci with location information in the first three columns), e.g., DNase_data_167_cells.rds

## Outputs

- K=1500 gene-cluster definitions mapping ensembl genes to clusters
- Per-cluster average expression matrix (X-bar) serving as the compressed predictor set
- Serialized binary model file (e.g., locus_level_model.bin) containing cluster definitions, selected predictor indices, and regression coefficients

## How to apply

Load the ENCODE RNA-seq training expression matrix and DH training data from the BIRD-data repository (e.g., DNase_data_167_cells.rds read via readRDS in R). Standardize the training expression matrix and apply BIRD's robust quantile normalization (the updated version handles large numbers of tied values, e.g., single-cell RNA-seq). Run k-means clustering of genes with K=1500 and compute each cluster's average expression to form the 1500 compressed predictors (X-bar). Rationale: gene expression predicts DH, and predictive information is contained in the whole transcriptome rather than only neighboring genes, so global clustering preserves transcriptome-wide signal while collapsing co-expressed genes into single predictors. Serialize the cluster definitions together with the per-locus selected predictor indices and regression coefficients into a binary model file (e.g., locus_level_model.bin) loadable by BIRD_predict with the -l flag.

## Related tools

- **BIRD** (Provides the modeling framework and R-backed code in which genes are k-means clustered and the locus-level model BIR(X-bar, Y) is fitted) — https://github.com/WeiqiangZhou/BIRD
- **BIRD_predict** (Loads the serialized model and runs prediction; the -l flag invokes the locus-level model built from the clustered predictors) — https://github.com/WeiqiangZhou/BIRD
- **R** (Required runtime (Rscript must be executable) used to load training data, perform the clustering, and fit the per-locus regressions) — https://www.r-project.org

## Examples

```
path_to_BIRD/BIRD_predict -b path_to_model/locus_level_model.bin -i FPKM_data_matrix.txt -o output_file.txt -l
```

## Evaluation signals

- locus_level_model.bin exists and is loadable by BIRD_predict with the -l locus-level flag
- Predictions produced by the locus-level model fall within the default bounded range of 0 to 14 on the log2(x+1) scale (adjustable via -u)
- Gene id matching succeeds: BIRD_predict outputs a matched matrix (input_file.match.txt) against the model's ensembl ids, ignoring ensembl version suffixes by default
- Clustering yields exactly K=1500 clusters whose per-cluster averages define 1500 predictors, with the top N=7 most correlated clusters retained per DHS locus

## Limitations

- BIRD will not discover new locations of DHSs when applied to analyze a new gene expression sample
- The quantile normalization must be robust to tied values; the older version performed poorly with many tied values (e.g., single-cell RNA-seq data)
- Genomic information in BIRD is based on human genome hg19 in some releases; hg38 support requires the newer model files

## Evidence

- [other] The locus-level model groups correlated predictors (i.e., co-expressed genes) into clusters and transforms each cluster into one predictor: [evidence span withheld — anchor status: not_found]
- [abstract] gene expression predicts DH, and information useful for prediction is contained in the whole transcriptome rather than limited to a regulatory element's neighboring genes: "gene expression predicts DH, and information useful for prediction is contained in the whole transcriptome rather than limited to a regulatory element's neighboring genes"
- [readme] The quantile normalization function is updated to a more robust version: "The quantile normalization function is updated to a more robust version"
- [readme] Users can also use -l option to perform prediction using the locus-level model rather than using the full model.: "Users can also use -l option to perform prediction using the locus-level model rather than using the full model."
- [discussion] one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
