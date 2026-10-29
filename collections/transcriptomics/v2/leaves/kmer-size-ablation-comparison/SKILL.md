---
name: kmer-size-ablation-comparison
description: Use when you need to decide which k-mer size (e.g., k=21 vs k=16) for
  CKG feature generation gives equal or better cell type classification accuracy across
  datasets like Chuang, Karaayvaz, PBMC3k, and Lee.
when_to_use_negative:
- Input is already a gene expression feature table — this skill compares k-mer-size-derived
  CKG features, not gene-expression-based baselines
- You need to evaluate fingerprint size or classifier choice rather than k-mer size
  — those must be held fixed for this ablation to be valid
- Reference-genome-aligned feature extraction is required — scSimClassify CKG features
  are derived from raw reads without alignment to a reference genome
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: jellyfish
  role: Produces the k-mer count files from which the sparse k-mer matrix is generated
  repo: github.com/digi2002/scSimClassify
- name: simhash
  role: Hashes k-mer vectors into fingerprints to build the CKG look-up table for
    each k value
  repo: github.com/digi2002/scSimClassify
- name: keras
  role: Implements part of the fixed classification module used for both k-mer sizes
  repo: github.com/digi2002/scSimClassify
- name: sklearn
  role: Implements classification methods applied identically to 21-mer and 16-mer
    CKG features
  repo: github.com/digi2002/scSimClassify
- name: pandas
  role: Records and aggregates per-dataset, per-k accuracy into the comparison table
  repo: github.com/digi2002/scSimClassify
- name: numpy
  role: Supports the sparse k-mer matrix and CKG expression computations
  repo: github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:9a08a1a534a67ce71546d36b22b5f183ef60ce680909a0f1289224b40ff4770d
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# kmer-size-ablation-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the scSimClassify ablation comparing 21-mer vs 16-mer compressed k-mer group (CKG) features for cell type classification, holding fingerprint size, classifier, and read type fixed. It tests whether longer k-mers preserve or improve classification accuracy in reference-free scRNA-seq analysis.

## When to use

Use when you need to decide which k-mer size (e.g., k=21 vs k=16) for CKG feature generation gives equal or better cell type classification accuracy across datasets like Chuang, Karaayvaz, PBMC3k, and Lee.

## When NOT to use

- Input is already a gene expression feature table — this skill compares k-mer-size-derived CKG features, not gene-expression-based baselines
- You need to evaluate fingerprint size or classifier choice rather than k-mer size — those must be held fixed for this ablation to be valid
- Reference-genome-aligned feature extraction is required — scSimClassify CKG features are derived from raw reads without alignment to a reference genome

## Inputs

- jellyfish k-mer count files derived from BAM or FASTA scRNA-seq reads (Chuang, Karaayvaz, PBMC3k, Lee datasets)
- Cell labels for each dataset
- Fixed fingerprint size and one fixed classifier from the four classification methods (keras/sklearn)
- Read-type specification held constant across k-mer sizes

## Outputs

- kmer_size_comparison.csv — aggregated 16-comparison table of classification accuracy for k=21 vs k=16
- Per-dataset, per-classifier accuracy records for both k-mer sizes

## How to apply

Build sparse k-mer matrices separately for k=21 and k=16 using the k-mer matrix generation module, which indexes k-mers and converts k-mer sequences to k-mer ids from jellyfish files. For each k, generate a CKG look-up table by selecting informative k-mers and simhashing k-mer vectors into fingerprints of a fixed fingerprint size, then transform cell features to CKG expression from k-mer abundance via the cell matrix generation module. Train the same fixed classifier — one of the four methods in the classification module implemented with keras/sklearn — on identical read-type data for each dataset and each k value. Record classification accuracy per dataset and k-mer size with pandas, aggregating all 16 dataset/classifier comparisons into a single table. The ablation is validated if 21-mer CKG features match or beat 16-mer CKG features in 10 of the 16 comparisons.

## Related tools

- **jellyfish** (Produces the k-mer count files from which the sparse k-mer matrix is generated) — github.com/digi2002/scSimClassify
- **simhash** (Hashes k-mer vectors into fingerprints to build the CKG look-up table for each k value) — github.com/digi2002/scSimClassify
- **keras** (Implements part of the fixed classification module used for both k-mer sizes) — github.com/digi2002/scSimClassify
- **sklearn** (Implements classification methods applied identically to 21-mer and 16-mer CKG features) — github.com/digi2002/scSimClassify
- **pandas** (Records and aggregates per-dataset, per-k accuracy into the comparison table) — github.com/digi2002/scSimClassify
- **numpy** (Supports the sparse k-mer matrix and CKG expression computations) — github.com/digi2002/scSimClassify

## Examples

```
import pandas as pd; comparisons = pd.DataFrame(rows); comparisons.to_csv('kmer_size_comparison.csv')
```

## Evaluation signals

- The reproduced comparison table shows 21-mer CKG features matching or beating 16-mer CKG features in 10 of the 16 dataset/classifier comparisons
- Sparse k-mer matrices for k=21 and k=16 are built from the same jellyfish files with identical read-type data
- Fingerprint size and classifier are identical across the two k values — only k differs
- Aggregated table contains exactly 16 rows (4 datasets x 4 classifier comparisons) with accuracy values per k-mer size
- Output file kmer_size_comparison.csv is produced via pandas aggregation

## Limitations

- The conclusion is conditional on fingerprint size, classifier, and read type being fixed; varying them changes the comparison
- CKG features are derived from raw reads without alignment to a reference genome, so results may not transfer to annotation-dependent pipelines
- README provides usage only as 'modify the path to use the modules', so exact command-line parameters must be inferred from the module code

## Evidence

- [other] Finding on 21-mer vs 16-mer performance: [evidence span withheld — anchor status: not_found]
