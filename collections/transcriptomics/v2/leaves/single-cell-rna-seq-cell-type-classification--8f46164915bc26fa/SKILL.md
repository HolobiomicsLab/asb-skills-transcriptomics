---
name: single-cell-rna-seq-cell-type-classification
description: Use when classifying cell types in scRNA-seq data without aligning reads
  to a reference genome, or when benchmarking compressed k-mer group (CKG) features
  against gene expression features with standard classifiers.
when_to_use_negative:
- The task requires reference-based cell type annotation with a complete, high-quality
  reference genome — scSimClassify is positioned as an alternative specifically when
  the reference genome is incomplete or insufficient to represent subject genomes.
- Input is already a curated CKG or gene expression feature matrix and the goal is
  only model fitting — skip k-mer matrix and CKG generation and go directly to the
  classification module.
- Reads cannot be decomposed into k-mers (e.g. inputs lacking sequence data), since
  CKG features are derived from raw reads without alignment to a reference genome.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: jellyfish
  role: counts k-mers from BAM/FASTA reads; the k-mer matrix is generated from jellyfish
    files
  repo: github.com/digi2002/scSimClassify
- name: simhash
  role: hashes k-mer abundance vectors into fingerprints to build the CKG look-up
    table
  repo: github.com/digi2002/scSimClassify
- name: keras
  role: implements the MLP neural network classifier used in the classification module
    and cross-dataset scSimClassify model
  repo: github.com/digi2002/scSimClassify
- name: sklearn
  role: provides random forest, gradient boosting, and SVM classifiers plus label
    handling and accuracy/weighted-F1 metrics
  repo: github.com/digi2002/scSimClassify
- name: pandas
  role: loads gene expression matrices and labels and writes the aggregated comparison
    CSV tables
  repo: github.com/digi2002/scSimClassify
- name: numpy
  role: array operations for feature matrices and score aggregation
  repo: github.com/digi2002/scSimClassify
- name: Bio
  role: Biopython listed among prerequisite packages for sequence handling
  repo: github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_004
  - task_006
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:4fa61990d9db8327d728b38a37353aa331bc20ab13006c9d1b3b5aa820b04588
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/digi2002/scSimClassify
  tools:
  - Bio
  - Python
  - jellyfish
  - keras
  - numpy
  - pandas
  - simhash
  - sklearn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# single-cell-rna-seq-cell-type-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reference-free scRNA-seq cell type classification (scSimClassify) that replaces gene expression features with compressed k-mer groups (CKGs) identified by simhash, then trains MLP, random forest, gradient boosting, and SVM classifiers to compare CKG vs gene expression accuracy.

## When to use

Use when classifying cell types in scRNA-seq data without aligning reads to a reference genome, or when benchmarking compressed k-mer group (CKG) features against gene expression features with standard classifiers.

## When NOT to use

- The task requires reference-based cell type annotation with a complete, high-quality reference genome — scSimClassify is positioned as an alternative specifically when the reference genome is incomplete or insufficient to represent subject genomes.
- Input is already a curated CKG or gene expression feature matrix and the goal is only model fitting — skip k-mer matrix and CKG generation and go directly to the classification module.
- Reads cannot be decomposed into k-mers (e.g. inputs lacking sequence data), since CKG features are derived from raw reads without alignment to a reference genome.

## Inputs

- BAM or FASTA raw read files per scRNA-seq dataset (processed by jellyfish into k-mer files)
- Gene expression feature matrices and cell type labels for Chuang, Karaayvaz, PBMC3k, and Lee datasets
- k-mer abundance profiles for k=21 and k=16 variants

## Outputs

- Sparse k-mer matrix (k-mer ids x cells) from jellyfish files
- CKG look-up table mapping informative k-mers to simhash fingerprints
- Cell-by-CKG expression feature matrix
- classification_results.csv with accuracy and weighted F1 per classifier/feature set
- cross_dataset_classification_results.csv with accuracy and weighted F1 for each train-on-one/predict-on-other pair
- kmer_size_comparison.csv aggregating the 16 k=21 vs k=16 comparisons

## How to apply

Run the four-module scSimClassify pipeline: (1) build a sparse k-mer matrix by indexing k-mers, converting k-mer sequences to k-mer ids, and generating the matrix from jellyfish files (BAM or FASTA inputs); (2) generate a CKG look-up table by selecting informative k-mers and simhashing k-mer vectors into fingerprints; (3) use the cell matrix generation module to transform cell features into CKG expression from k-mer abundance; (4) train the four classification methods (keras MLP, sklearn random forest, gradient boosting, SVM) on the CKG features. Split each dataset into train/test partitions, predict labels on held-out cells, and compute accuracy and weighted F1 with sklearn.metrics. Aggregate scores for intra-dataset comparisons (Chuang, Karaayvaz, PBMC3k, Lee) and cross-dataset transfers (e.g. train on Chuang, predict Karaayvaz; train on PBMC3k, predict Lee), then verify that CKG features beat gene expression features in the majority of experimental cases and that 21-mer variants match or beat 16-mer variants in 10 of 16 comparisons.

## Related tools

- **jellyfish** (counts k-mers from BAM/FASTA reads; the k-mer matrix is generated from jellyfish files) — github.com/digi2002/scSimClassify
- **simhash** (hashes k-mer abundance vectors into fingerprints to build the CKG look-up table) — github.com/digi2002/scSimClassify
- **keras** (implements the MLP neural network classifier used in the classification module and cross-dataset scSimClassify model) — github.com/digi2002/scSimClassify
- **sklearn** (provides random forest, gradient boosting, and SVM classifiers plus label handling and accuracy/weighted-F1 metrics) — github.com/digi2002/scSimClassify
- **pandas** (loads gene expression matrices and labels and writes the aggregated comparison CSV tables) — github.com/digi2002/scSimClassify
- **numpy** (array operations for feature matrices and score aggregation) — github.com/digi2002/scSimClassify
- **Bio** (Biopython listed among prerequisite packages for sequence handling) — github.com/digi2002/scSimClassify

## Examples

```
python generate_kmer_matrix.py --input reads.bam --kmer 21; python ckg_generation.py --fingerprint_size <fixed>; python classification.py --features ckg --classifier mlp --labels cell_labels.csv
```

## Evaluation signals

- Intra-dataset table (classification_results.csv) shows CKG features beating gene expression features in the majority of experimental cases across MLP, RF, GBM, and SVM on the four datasets.
- 21-mer vs 16-mer comparison table (kmer_size_comparison.csv) shows 21-mer CKG variants performing the same or better in 10 of 16 dataset/classifier comparisons.
- Cross-dataset results (cross_dataset_classification_results.csv) show CKG-feature accuracy and weighted F1 meeting or exceeding the gene-expression baseline for Chuang->Karaayvaz and PBMC3k/Lee transfers.
- CKG look-up table is reproducibly derived: informative k-mers selected then simhashed into fixed-size fingerprints for both k=21 and k=16.
- Weighted F1 scores are computed with sklearn.metrics on held-out test splits (not training data) for every classifier/feature-set combination.

## Limitations

- Requires Python 3.6 with simhash, keras, sklearn, pandas, numpy, and Bio installed; usage requires manually modifying paths in the repository modules.
- Pipeline depends on jellyfish k-mer counting files; raw reads must first be processed into jellyfish output before the k-mer matrix can be built.
- Cross-dataset classification is limited to shared cell types between dataset pairs; findings were demonstrated on only four datasets (Chuang, Karaayvaz, PBMC3k, Lee).
- CKG advantage is a majority-case result, not universal — gene expression features can outperform CKG features in individual classifier/dataset cases.

## Evidence
