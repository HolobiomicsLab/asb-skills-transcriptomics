---
name: ckg-feature-construction
description: Use when raw scRNA-seq reads (BAM/FASTA) must be converted into alignment-free
  features for cell type classification, especially when a reference genome is incomplete
  or insufficient; also for comparing k-mer sizes (e.g., 21-mer vs 16-mer) under fixed
  fingerprints, classifiers, and read types.
when_to_use_negative:
- A high-quality, complete reference genome is available and alignment-based gene
  expression features are preferred, since CKG features are derived from raw reads
  without alignment to a reference genome.
- Input is already a gene expression feature table or processed feature matrix rather
  than raw reads, so k-mer matrix generation from jellyfish files is not applicable.
- The task requires interpretability of features as genes, since compressed k-mer
  groups (CKGs) are simhashed fingerprints rather than annotated gene features.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3934
- http://edamontology.org/topic_3308
tools:
- name: jellyfish
  role: Counts k-mers from reads; the sparse k-mer matrix is generated from jellyfish
    files after indexing k-mer and converting k-mer sequence to k-mer id.
- name: simhash
  role: Simhashes selected informative k-mer vectors into fingerprints to define the
    compressed k-mer groups (CKGs) and the CKG look-up table.
- name: keras
  role: Implements one of the four classification methods applied to the CKG expression
    features.
- name: sklearn
  role: Implements the fixed classifiers used to evaluate CKG features across datasets
    (Chuang, Karaayvaz, PBMC3k, Lee).
- name: pandas
  role: Records and aggregates per-dataset, per-k classification accuracy into a comparison
    table such as kmer_size_comparison.csv.
- name: numpy
  role: Supports sparse k-mer matrix construction and vectorized feature transformation.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:2eb0b78954570c18d8d1db0244091d57391e9c180ba5c9474effda1632ffebdc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/digi2002/scSimClassify
  tools:
  - Python
  - jellyfish
  - keras
  - numpy
  - pandas
  - simhash
  - sklearn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3934
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# ckg-feature-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct compressed k-mer group (CKG) features from raw scRNA-seq reads with scSimClassify: build a sparse k-mer matrix from jellyfish files, select informative k-mers, simhash k-mer vectors into fingerprints, and transform cell k-mer abundance into CKG expression for reference-free cell type classification.

## When to use

Use when raw scRNA-seq reads (BAM/FASTA) must be converted into alignment-free features for cell type classification, especially when a reference genome is incomplete or insufficient; also for comparing k-mer sizes (e.g., 21-mer vs 16-mer) under fixed fingerprints, classifiers, and read types.

## When NOT to use

- A high-quality, complete reference genome is available and alignment-based gene expression features are preferred, since CKG features are derived from raw reads without alignment to a reference genome.
- Input is already a gene expression feature table or processed feature matrix rather than raw reads, so k-mer matrix generation from jellyfish files is not applicable.
- The task requires interpretability of features as genes, since compressed k-mer groups (CKGs) are simhashed fingerprints rather than annotated gene features.

## Inputs

- Raw scRNA-seq reads as BAM or FASTA files
- jellyfish k-mer count files (k=16 or k=21)
- Cell labels and gene expression feature tables provided with the repository datasets

## Outputs

- Sparse k-mer matrix (k-mers indexed as k-mer ids)
- CKG look-up table mapping simhash fingerprints to informative k-mer groups
- Cell-by-CKG feature matrix (CKG expression from k-mer abundance)
- Classification accuracy per dataset/classifier (e.g., kmer_size_comparison.csv aggregating 16 comparisons)

## How to apply

Run the scSimClassify pipeline modules in sequence with Python 3.6 and the simhash, keras, sklearn, pandas, numpy, and Bio packages. First, in the k-mer matrix generation module, build a sparse k-mer matrix by indexing k-mer, converting k-mer sequence to k-mer id, and generating the k-mer matrix from jellyfish files (inputs may be BAM or FASTA). Second, in the CKG generation module, select informative k-mers and simhash the k-mer vectors into fingerprints to produce a CKG look-up table, fixing the fingerprint size (as in the 21-mer vs 16-mer comparison where size, classifiers, and read types are held fixed). Third, in the cell matrix generation module, use the CKG look-up table to transform cell features into CKG expression from k-mer abundance. Finally, train one of the four classification methods in the classification module (keras/sklearn implementations) and evaluate accuracy — e.g., 21-mer CKG features matched or beat 16-mer variants in 10 of 16 dataset/classifier comparisons.

## Related tools

- **jellyfish** (Counts k-mers from reads; the sparse k-mer matrix is generated from jellyfish files after indexing k-mer and converting k-mer sequence to k-mer id.)
- **simhash** (Simhashes selected informative k-mer vectors into fingerprints to define the compressed k-mer groups (CKGs) and the CKG look-up table.)
- **keras** (Implements one of the four classification methods applied to the CKG expression features.)
- **sklearn** (Implements the fixed classifiers used to evaluate CKG features across datasets (Chuang, Karaayvaz, PBMC3k, Lee).)
- **pandas** (Records and aggregates per-dataset, per-k classification accuracy into a comparison table such as kmer_size_comparison.csv.)
- **numpy** (Supports sparse k-mer matrix construction and vectorized feature transformation.)

## Evaluation signals

- The CKG look-up table contains simhash fingerprints built only from selected informative k-mers, with fingerprint size held fixed across compared variants.
- Cell-by-CKG matrices reproduce for both k=21 and k=16, and identical classifiers/read types are used per dataset so the k-mer size comparison is controlled.
- Aggregated results (16 comparisons) show 21-mer CKG features matching or beating 16-mer CKG features in 10 of 16 dataset/classifier comparisons.
- CKG features yield better classification accuracy than gene expression features in the majority of experimental cases.
- Output table (e.g., kmer_size_comparison.csv) has one row per dataset/classifier pairing with accuracy recorded for each k-mer size.

## Limitations

- Pipeline requires python 3.6 and specific packages (simhash, keras, sklearn, pandas, numpy, Bio); the README notes 'Please modify the path to use the modules,' so paths must be adjusted manually.
- CKGs are derived from raw reads without alignment to a reference genome, so features are not directly interpretable as genes.
- The 21-mer advantage is not universal: 21-mer CKG variations matched or exceeded 16-mer performance in 10 of 16 comparisons, not all.

## Evidence

- [abstract] Compressed k-mer groups (CKGs) are identified by the simhash method and serve as cells' features: "The compressed k -mer groups (CKGs), identified by the simhash method, contain k -mers with similar abundance profiles and serve as the cells' features."
- [methods] CKG generation from informative k-mers and simhashed fingerprints: "By selecting informative k-mers and simhashing k-mer vectores into fingerprints, a CKG look-up table is generated."
- [methods] Sparse k-mer matrix generation from jellyfish files: "Sparse k-mer matrix is build by several steps, such as indexing k-mer, convert k-mer sequence to k-mer id, and generate k-mer matrix from jellyfish files."
- [readme] README prerequisite packages: "python 3.6

python packages: simhash, keras, sklearn, pandas, numpy, Bio"
- [readme] README input formats: "Both BAM and FASTA file can be the inputs."
