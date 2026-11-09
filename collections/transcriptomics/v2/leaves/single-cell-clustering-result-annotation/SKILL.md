---
name: single-cell-clustering-result-annotation
description: Use when scQA (or similar) clustering outputs exist for a scRNA-seq dataset
  and you must confirm that each cluster's key genes match known markers/DEGs to biologically
  annotate clusters.
when_to_use_negative:
- Clustering labels have not yet been generated — scQA's label propagation must run
  first (labels.txt does not exist)
- No published marker/DEG reference gene sets exist for the dataset's tissue, so overlap
  validation cannot be grounded
- Goal is cluster identification itself (e.g., estimating number of cell types) rather
  than annotating/validating existing clusters — use scQA's LC1/LC2/CC pipeline directly
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scQA
  role: Produces the clusters and cluster-specific key genes (consensus landmarks)
    that this validation workflow annotates; labels.txt output feeds directly into
    the overlap analysis
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:f4213f683f443eafd73cf99a89e860ba43ede264b102bfcece6b5b6ed17c4681
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - scQA
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# single-cell-clustering-result-annotation

## Summary

Validates biological significance of cluster-defining key genes (consensus landmarks) identified by scQA by overlapping them against cell-type marker genes, differentially expressed genes (|logFC| > 2, adjusted p < 0.01), and hub genes from the dataset's original publication, plus GO-term enrichment checks.

## When to use

Use when scQA (or similar) clustering outputs exist for a scRNA-seq dataset and you must confirm that each cluster's key genes match known markers/DEGs to biologically annotate clusters.

## When NOT to use

- Clustering labels have not yet been generated — scQA's label propagation must run first (labels.txt does not exist)
- No published marker/DEG reference gene sets exist for the dataset's tissue, so overlap validation cannot be grounded
- Goal is cluster identification itself (e.g., estimating number of cell types) rather than annotating/validating existing clusters — use scQA's LC1/LC2/CC pipeline directly

## Inputs

- labels.txt clustering output from scQA (chained from a prior clustering task)
- Preprocessed expression matrix (cells * genes, no row/column names) as produced by pre.R
- Reference gene sets from the dataset's original publication: cell-type marker genes and DEGs (|logFC| > 2, adjusted p-value < 0.01)

## Outputs

- key_genes.tsv — per-cluster key-gene table with overlap counts and hypergeometric/Fisher-style enrichment statistics against marker and DEG reference sets

## How to apply

Load the scQA clustering output labels.txt together with the preprocessed cells-by-genes expression matrix (no row/column names) to recover the genes defining each cluster as consensus landmarks. Compile reference gene sets — cell-type markers and DEGs (|logFC| > 2 and adjusted p-value < 0.01) — from the dataset's original publication (e.g., Camp2 GSE81252 for liver, Camp1 GSE75140 for brain). For each cluster, compute overlap counts and hypergeometric/Fisher-style enrichment statistics between its key genes and the marker/DEG reference sets. Cross-check key-gene coherence against paper-reported examples, e.g., endothelial key genes (ADTRP, CDH5, CLEC14A, ACVRL1) enriched in blood vessel development GO terms. Write the per-cluster key-gene table with overlap and enrichment statistics to key_genes.tsv, accepting it only when each cluster's key genes show documented overlap with known markers/DEGs consistent with the paper's external and internal validation.

## Related tools

- **scQA** (Produces the clusters and cluster-specific key genes (consensus landmarks) that this validation workflow annotates; labels.txt output feeds directly into the overlap analysis) — https://github.com/LD-Lyndee/scQA

## Examples

```
./main xin_matrix.txt o  # produces labels.txt, then overlap cluster key genes against published marker/DEG sets and write key_genes.tsv
```

## Evaluation signals

- key_genes.tsv exists and contains one row-set per cluster with overlap counts and enrichment statistics against marker and DEG sets
- Key genes of each cluster show statistically documented overlap (hypergeometric/Fisher-style enrichment) with publication-derived marker/DEG reference sets
- Endothelial key genes (e.g., LM2 landmarks ADTRP, CDH5, CLEC14A, ACVRL1 in Camp2 GSE81252) are enriched in endothelial-related GO terms such as blood vessel development
- DEG reference sets were built with the paper's exact thresholds: |logFC| > 2 and adjusted p-value < 0.01
- Marker vs. key-gene mismatches are explained biologically (e.g., ADTRP is not a canonical marker but regulates endothelial proliferation and migration)

## Limitations

- Retaining only 2000 highly variable genes during preprocessing excluded some informative genes, which can limit the key genes recovered per cluster
- Genes may be misclassified in LC1 and LC2 due to high dimensionality, potentially contaminating cluster key genes
- Small clusters may lack marker support — e.g., Cluster8 in the Xenopus lung dataset contained only three cells and likely represents outliers
- Some clusters may incorrectly merge distinct cell types (e.g., endothelial and mesenchymal cells grouped into a single Cluster3), complicating marker-based annotation

## Evidence

- [results] the landmark genes scQA identified for the endothelial cell type are enriched in endothelial-related GO terms: [evidence span withheld — anchor status: not_found]
- [results] DEG selection criteria: "genes with the absolute value of logFC above 2 and adjusted p-value less than 0.01 were selected as DEGs"
- [intro] key genes share attributes with DEGs, hub genes, markers: "scQA captures key genes sharing attributes with differentially expressed genes, hub genes and marker genes, serving as reliable landmarks for clustering cells"
- [abstract] internal/external validation of key genes: "we confirm the effectiveness and potential biological significance of the identified key genes through both external and internal validation"
- [readme] input matrix format from README: "The input file must be in the form of a matrix of cells * genes, without names of rows and columns."
