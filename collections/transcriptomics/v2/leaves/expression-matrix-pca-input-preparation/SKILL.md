---
name: expression-matrix-pca-input-preparation
description: Use when a single-cell dataset must be formatted as a cell-by-feature
  CSV (genes, transcripts, or principal components) before running Sphetcher at a
  given sketch size.
when_to_use_negative:
- Input is raw single-cell sequencing data (e.g. FASTQ or 10x count matrices) that
  has not yet been reduced to a cell-by-feature/PC numeric table — Sphetcher consumes
  the precomputed feature/PC matrix, not raw reads
- The downstream task requires gene-level annotation, normalization, or clustering
  itself — this skill only covers formatting the matrix for the spherical sketcher,
  not performing those analyses
- Prior labels are non-categorical or cannot be encoded as integers between 1 and
  K, so they cannot be supplied via the class_labels.csv / l_min fairness interface
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Sphetcher
  role: Consumes the prepared cell-by-feature CSV (plus optional class labels and
    l_min) to produce the sketch indicator vector; run after compilation with a C++11
    compiler
  repo: https://github.com/canzarlab/Sphetcher
- name: git
  role: Clones the Sphetcher repository, whose /data directory (e.g. zeisel_pca.csv)
    provides the reference input format the prepared matrix must match
  repo: https://github.com/canzarlab/Sphetcher
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.101126
    title: 'Sphetcher: Spherical Thresholding Improves Sketching of Single-Cell Transcriptomic
      Heterogeneity'
  source_content_hash: sha256:f28459e18f9c7dd7b11a1cfbdfea413614b611c188928822f17f316877da849f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/canzarlab/Sphetcher
  tools:
  - Sphetcher
  - git
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# expression-matrix-pca-input-preparation

## Summary

Prepare a cell-by-feature CSV expression/PCA matrix (and optional class-label file) so it can be consumed by Sphetcher's spherical thresholding sketcher, following the input format defined in the Sphetcher paper and README.

## When to use

Use when a single-cell dataset must be formatted as a cell-by-feature CSV (genes, transcripts, or principal components) before running Sphetcher at a given sketch size.

## When NOT to use

- Input is raw single-cell sequencing data (e.g. FASTQ or 10x count matrices) that has not yet been reduced to a cell-by-feature/PC numeric table — Sphetcher consumes the precomputed feature/PC matrix, not raw reads
- The downstream task requires gene-level annotation, normalization, or clustering itself — this skill only covers formatting the matrix for the spherical sketcher, not performing those analyses
- Prior labels are non-categorical or cannot be encoded as integers between 1 and K, so they cannot be supplied via the class_labels.csv / l_min fairness interface

## Inputs

- expression_matrix.csv: cell-by-feature numeric CSV (rows are cells, columns are genes, transcripts, or principal components PCs), e.g. zeisel_pca.csv
- sketch_size: number of samples to obtain from the data set (e.g. 1000, or 1%/4%/10% of the dataset)
- class_labels.csv (optional): column vector of prior class labels, each class an integer between 1 and K
- l_min (optional): minimum number of representatives to sample from each class

## Outputs

- sketch_indicator_output.csv: an indicator vector of n samples where 1 indicates the sample is in the sketch, 0 otherwise, containing exactly sketch_size 1's

## How to apply

Format the single-cell data as a matrix where rows are samples (cells) and columns are features (genes, transcripts, or principal components PCs), saved in comma-separated values (CSV) format, mirroring the example input zeisel_pca.csv provided in the repository's /data directory. If prior biological or experimental knowledge (e.g., cell type, collection time point) must be preserved, also prepare a class_labels.csv column vector where each class is an integer between 1 and K, and choose an l_min minimum number of representatives per class. Select the sketch_size as an absolute cell count, either an absolute number such as 1000 or a percentage of the dataset as in the paper's experiments (1%, 4%, and 10% of the Saunders 665,858-cell, Zeisel 465,281-cell, and Cao 2,026,641-cell datasets). Note that in the paper's experiments Sphetcher used Pearson correlation as the distance metric computed on these feature columns, so the PCA/feature transformation of the matrix directly affects sketch quality. Validate the preparation by running the compiled sphetcher binary on the matrix and checking that the resulting sketch_indicator_output.csv is an indicator vector of n samples with exactly sketch_size entries equal to 1.

## Related tools

- **Sphetcher** (Consumes the prepared cell-by-feature CSV (plus optional class labels and l_min) to produce the sketch indicator vector; run after compilation with a C++11 compiler) — https://github.com/canzarlab/Sphetcher
- **git** (Clones the Sphetcher repository, whose /data directory (e.g. zeisel_pca.csv) provides the reference input format the prepared matrix must match) — https://github.com/canzarlab/Sphetcher

## Examples

```
sphetcher zeisel_pca.csv 1000 sketch_indicator_output.csv
```

## Evaluation signals

- CSV parses such that rows correspond to cells and columns to numeric features/PCs, matching the structure of the /data example zeisel_pca.csv
- Running sphetcher on the prepared matrix yields sketch_indicator_output.csv containing exactly sketch_size entries equal to 1 and n-sketch_size entries equal to 0
- Percentage-based sketch sizes match the paper's expected counts, e.g. 1% of the 665,858-cell Saunders dataset corresponds to the 6646-cell sketch and 10% to the 66649-cell sketch
- The feature columns are the same space over which the Pearson correlation distance metric is computed, so PCA/feature choice is consistent across all datasets and trials being compared
- If fairness inputs are used, the class_labels.csv vector contains integers between 1 and K and l_min representatives per class appear in the sketch

## Limitations

- The article does not specify software version numbers for Sphetcher or how the PCA/feature transformation feeding the matrix should be performed
- The interplay between distance-metric-based sketching (Pearson correlation on the prepared features) and internal preprocessing routines of downstream scRNA-seq algorithms was not systematically addressed
- For very large datasets (e.g. the 2,026,641-cell MOCA), Sphetcher was combined with a prior grid sampling strategy, so plain matrix preparation alone may not cover the full scalability workflow

## Evidence

- [methods] First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs).: "First you need to provide a matrix where rows are samples (cells) and columns are features (genes, transcripts, principal components PCs)."
- [readme] prior information stored in a column vector, each class is presented by an integer between 1 and K: "prior information stored in a column vector, each class is presented by an integer between 1 and K, where K is the number of classes."
- [methods] An example of inputs is provided in the directory /data: "An example of inputs is provided in the directory /data"
- [readme] an indicator vector of n samples where 1 indicates the sample is in the sketch: "an indicator vector of `n` samples where 1 indicates the sample is in the sketch, 0 otherwise (there are `sketch_size` 1's in the vector)"
- [discussion] In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric: "In most of the experiments in this study, Sphetcher used Pearson correlation as distance metric and was combined with a specific algorithm for downstream analysis."
