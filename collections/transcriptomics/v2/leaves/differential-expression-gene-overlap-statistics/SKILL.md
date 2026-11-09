---
name: differential-expression-gene-overlap-statistics
description: Use when you have cluster-derived key genes from scQA (e.g. labels.txt
  chained from a prior clustering task) and need to confirm they agree with published
  marker genes, hub genes, or DEGs for the same cell type.
when_to_use_negative:
- The clustering output does not yet exist or labels.txt has not been produced by
  scQA for the dataset in question.
- No publication-derived reference gene sets (markers or DEGs) exist for the dataset,
  so no external overlap validation is possible.
- The goal is de novo marker discovery rather than validation of scQA landmarks against
  known markers.
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scQA
  role: Source of the key genes (landmarks) being validated; its clustering output
    labels.txt plus the preprocessed expression matrix define per-cluster key genes.
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:7f3b5bf6cde896f5dd487c13f0ee026d9867d67568076ba141271fc7b3117c8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - scQA
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# differential-expression-gene-overlap-statistics

## Summary

Validates that key genes (landmarks) identified by scQA for each cell-type cluster are biologically significant by computing overlap counts and hypergeometric/Fisher-style enrichment against reference sets of cell-type marker genes and DEGs (|logFC| > 2, adjusted p-value < 0.01) from the dataset's original publication.

## When to use

Use when you have cluster-derived key genes from scQA (e.g. labels.txt chained from a prior clustering task) and need to confirm they agree with published marker genes, hub genes, or DEGs for the same cell type.

## When NOT to use

- The clustering output does not yet exist or labels.txt has not been produced by scQA for the dataset in question.
- No publication-derived reference gene sets (markers or DEGs) exist for the dataset, so no external overlap validation is possible.
- The goal is de novo marker discovery rather than validation of scQA landmarks against known markers.

## Inputs

- labels.txt — scQA clustering output mapping cells to clusters (chained from the clustering task)
- preprocessed expression matrix (cells * genes, no row/column names) used for scQA clustering
- reference gene sets from the dataset's original publication: cell-type marker genes and DEGs (|logFC| > 2, adjusted p-value < 0.01)

## Outputs

- key_genes.tsv — per-cluster key-gene table with overlap counts and hypergeometric/Fisher-style enrichment statistics against marker/DEG reference sets

## How to apply

Load the scQA clustering output labels.txt together with the preprocessed expression matrix to recover the genes defining each cluster as key genes (consensus landmarks). Compile reference gene sets from the dataset's original publication: cell-type marker genes and DEGs selected with |logFC| > 2 and adjusted p-value < 0.01. For each cluster, compute overlap statistics between its key genes and the marker/DEG reference sets (overlap counts plus hypergeometric/Fisher-style enrichment). Cross-check key-gene coherence against paper-reported examples — e.g. endothelial key genes ADTRP, CDH5, CLEC14A, and ACVRL1 associated with blood vessel development in the Camp2 (GSE81252) dataset — then write the per-cluster key-gene table with overlap and enrichment statistics to key_genes.tsv.

## Related tools

- **scQA** (Source of the key genes (landmarks) being validated; its clustering output labels.txt plus the preprocessed expression matrix define per-cluster key genes.) — https://github.com/LD-Lyndee/scQA

## Examples

```
git clone https://github.com/LD-Lyndee/scQA.git; ./main xin_matrix.txt o  # then compare key genes per cluster in labels.txt against publication DEGs (|logFC|>2, padj<0.01) to write key_genes.tsv
```

## Evaluation signals

- key_genes.tsv exists and contains a row per cluster with overlap counts and enrichment statistics for both marker and DEG reference sets.
- Each cluster's key genes show documented overlap with known markers/DEGs consistent with the paper's external/internal key-gene validation.
- Endothelial key genes from Camp2 are enriched in blood vessel development GO terms, matching the reported ADTRP, CDH5, CLEC14A, ACVRL1 examples.
- Reference DEG sets were compiled using the paper's thresholds: |logFC| > 2 and adjusted p-value < 0.01.
- Enrichment p-values are computed with hypergeometric/Fisher-style tests and are reported alongside raw overlap counts, not percentages alone.

## Limitations

- Cluster composition can be imperfect: in the Xenopus lung dataset, Cluster8 contained only three cells (likely outliers) and lacked marker genes, and endothelial and mesenchymal cells were incorrectly grouped into a single cluster (Cluster3).
- Retaining only 2000 highly variable genes during preprocessing excluded some informative genes, which may affect which landmarks are recovered as key genes.
- Genes may be misclassified in LC1 and LC2 due to the influence of high dimensionality, potentially distorting key-gene membership.

## Evidence

- [results] genes with the absolute value of logFC above 2 and adjusted p-value less than 0.01 were selected as DEGs: "genes with the absolute value of logFC above 2 and adjusted p-value less than 0.01 were selected as DEGs"
- [intro] scQA captures key genes sharing attributes with differentially expressed genes, hub genes and marker genes: "scQA captures key genes sharing attributes with differentially expressed genes, hub genes and marker genes, serving as reliable landmarks for clustering cells"
- [abstract] The source codes are publicly available at https://github.com/LD-Lyndee/scQA.: "The source codes are publicly available at https://github.com/LD-Lyndee/scQA."
- [readme] labels.txt — clustering result output: "- **labels.txt** Clustering result."
- [abstract] we confirm the effectiveness and potential biological significance of the identified key genes through both external and internal validation: "we confirm the effectiveness and potential biological significance of the identified key genes through both external and internal validation"
