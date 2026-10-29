---
name: kmer-matrix-generation-from-reads
description: Use when raw single-cell RNA-seq reads (BAM/FASTA) are available and
  cell-type features must be derived without aligning to a reference genome, e.g.
  when the reference genome is incomplete or insufficient.
when_to_use_negative:
- A complete, high-quality reference genome exists and gene-expression features suffice
  — the paper offers scSimClassify as an alternative especially when the reference
  genome is incomplete or insufficient
- Input is already a cell x feature matrix (e.g. gene expression or CKG expression
  table); the k-mer matrix generation stage assumes raw reads
- The task requires transcript/gene-level quantification rather than k-mer abundance
  features
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: jellyfish
  role: k-mer counting from raw reads to produce jellyfish files from which the sparse
    k-mer matrix is generated
- name: Python 3.6
  role: runtime for the k-mer matrix generation module of scSimClassify
  repo: github.com/digi2002/scSimClassify
- name: pandas
  role: aggregating per-dataset classification results after features are built
- name: numpy
  role: array handling for the sparse k-mer matrix
- name: simhash
  role: downstream simhashing of k-mer vectors into fingerprints to build the CKG
    look-up table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:80d36455f2bb39f62e2375ff74743f9558eec9d117bd5cff2442d776371c75d0
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
  - Python 3.6
  - jellyfish
  - numpy
  - pandas
  - simhash
  - sklearn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# kmer-matrix-generation-from-reads

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Builds a sparse k-mer abundance matrix from scRNA-seq raw reads (BAM or FASTA) using jellyfish k-mer counting, indexing k-mers and converting k-mer sequences to k-mer ids, as the reference-free input stage of the scSimClassify pipeline.

## When to use

Use when raw single-cell RNA-seq reads (BAM/FASTA) are available and cell-type features must be derived without aligning to a reference genome, e.g. when the reference genome is incomplete or insufficient.

## When NOT to use

- A complete, high-quality reference genome exists and gene-expression features suffice — the paper offers scSimClassify as an alternative especially when the reference genome is incomplete or insufficient
- Input is already a cell x feature matrix (e.g. gene expression or CKG expression table); the k-mer matrix generation stage assumes raw reads
- The task requires transcript/gene-level quantification rather than k-mer abundance features

## Inputs

- Single-cell RNA-seq reads in BAM or FASTA format
- jellyfish k-mer count output files
- Chosen k-mer size (e.g. k=16 or k=21)
- Cell labels / cell metadata for downstream classification

## Outputs

- Sparse k-mer abundance matrix (cells x k-mers, k-mer ids as columns)
- k-mer index mapping k-mer sequences to k-mer ids

## How to apply

Input BAM or FASTA files and count k-mers with jellyfish at the chosen k size (e.g. k=16 or k=21 in the paper's comparisons). Index the k-mers and convert k-mer sequences to k-mer ids, then generate a sparse k-mer matrix directly from the jellyfish output files. This matrix is passed to the CKG generation module, where informative k-mers are selected and simhashed into fingerprints forming the CKG look-up table. Because CKGs are derived from raw reads without alignment to a reference genome, this step replaces gene-expression feature extraction entirely. Decision point: choose k per experiment; the paper evaluates both 16-mer and 21-mer variants while fixing fingerprint size, classifier, and read type.

## Related tools

- **jellyfish** (k-mer counting from raw reads to produce jellyfish files from which the sparse k-mer matrix is generated)
- **Python 3.6** (runtime for the k-mer matrix generation module of scSimClassify) — github.com/digi2002/scSimClassify
- **pandas** (aggregating per-dataset classification results after features are built)
- **numpy** (array handling for the sparse k-mer matrix)
- **simhash** (downstream simhashing of k-mer vectors into fingerprints to build the CKG look-up table)

## Evaluation signals

- Sparse k-mer matrix contains the same number of cell rows as the input BAM/FASTA samples
- Every k-mer column corresponds to a valid k-mer id from the jellyfish files after sequence-to-id conversion
- Downstream CKG look-up table is successfully generated by selecting informative k-mers and simhashing k-mer vectors
- In the 21-mer vs 16-mer comparison (fingerprint size, classifiers, and read types fixed), 21-mer CKG features matched or beat 16-mer CKG features in 10 of 16 dataset/classifier comparisons
- CKG features derived from the matrix yield better scRNA-seq classification accuracy than gene expression features in the majority of experimental cases

## Limitations

- Requires jellyfish k-mer count files; the module does not accept pre-aligned or quantified gene expression tables
- The choice of k-mer size matters: performance varies by k (16 vs 21) across datasets (Chuang, Karaayvaz, PBMC3k, Lee), so k must be tuned per dataset
- Usage requires modifying paths to use the modules, as stated in the README; no turnkey CLI is provided
- Classification performance depends on the fingerprint size fixed during simhashing; results are contingent on that parameter

## Evidence

- [results] 21-mer CKG feature variations achieved the same or better performance than 16-mer CKG feature variations in 10 out of 16 comparisons across the datasets.: [evidence span withheld — anchor status: not_found]
