---
name: key-gene-identification-from-clusters
description: Use when scQA clustering labels are available and you must identify and
  validate cluster-defining key genes against known cell-type markers, DEGs, and hub
  genes.
when_to_use_negative:
- Clustering labels have not yet been produced by scQA — key genes can only be recovered
  from the clustering output (labels.txt).
- No published reference gene sets (markers, hub genes, DEGs) exist for the dataset,
  so external validation of key-gene significance is impossible.
- The goal is cell clustering itself rather than validating key genes for already-produced
  clusters.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scQA
  role: Produces the cluster assignments (labels.txt) from which landmark/key genes
    per cluster are recovered; scQA concurrently identifies cell types and cell type-specific
    key genes from qualitative and quantitative perspectives.
  repo: https://github.com/LD-Lyndee/scQA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.12.021
    title: 'scQA: A dual-perspective cell type identification model for single cell
      transcriptome data'
  source_content_hash: sha256:908888ffdf8e2fbec1e65808045425a88409008c8f2767a37a5d7d7c89ba9609
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LD-Lyndee/scQA
  tools:
  - scQA
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# key-gene-identification-from-clusters

## Summary

Recovers the key genes (landmark genes) defining each scQA cluster and validates their biological significance against published marker genes, hub genes, and differentially expressed genes (|logFC| > 2, adjusted p-value < 0.01) from the dataset's original publication.

## When to use

Use when scQA clustering labels are available and you must identify and validate cluster-defining key genes against known cell-type markers, DEGs, and hub genes.

## When NOT to use

- Clustering labels have not yet been produced by scQA — key genes can only be recovered from the clustering output (labels.txt).
- No published reference gene sets (markers, hub genes, DEGs) exist for the dataset, so external validation of key-gene significance is impossible.
- The goal is cell clustering itself rather than validating key genes for already-produced clusters.

## Inputs

- labels.txt clustering output from scQA (chained from cluster assignment task)
- preprocessed expression matrix (cells × genes, no row/column names, from pre.R)
- Reference marker gene sets and DEG lists from the dataset's original publication (DEGs with |logFC| > 2, adjusted p-value < 0.01)

## Outputs

- key_genes.tsv: per-cluster key-gene table with overlap and enrichment statistics against marker/DEG reference sets

## How to apply

Load the scQA clustering output labels.txt (chained from the clustering step) together with the preprocessed expression matrix (cells × genes, no row/column names, produced by pre.R) to recover the genes defining each cluster as consensus landmark genes. Compile reference gene sets from the dataset's original publication: cell-type marker genes and DEGs selected with |logFC| > 2 and adjusted p-value < 0.01. For each cluster, compute overlap counts and hypergeometric/Fisher-style enrichment between its key genes and the marker/DEG reference sets. Check coherence against paper-reported examples, e.g. endothelial key genes ADTRP, CDH5, CLEC14A, and ACVRL1 associated with blood vessel development GO terms in the Camp2 dataset. Write the per-cluster key-gene table with overlap and enrichment statistics to key_genes.tsv.

## Related tools

- **scQA** (Produces the cluster assignments (labels.txt) from which landmark/key genes per cluster are recovered; scQA concurrently identifies cell types and cell type-specific key genes from qualitative and quantitative perspectives.) — https://github.com/LD-Lyndee/scQA

## Examples

```
./main xin_matrix.txt o  # then post-process labels.txt with the preprocessed matrix to write key_genes.tsv
```

## Evaluation signals

- Each cluster's key genes show documented overlap with known markers/DEGs consistent with the paper's external and internal key-gene validation.
- Endothelial key genes (e.g., LM2 in the Camp2 dataset) are enriched in endothelial-related GO terms such as blood vessel development.
- Reported marker examples (ADTRP, CDH5, CLEC14A, ACVRL1) appear among key genes with expected annotation roles (ADTRP regulates endothelial proliferation and migration).
- key_genes.tsv contains, for every cluster, overlap counts plus hypergeometric/Fisher-style enrichment statistics against the marker and DEG reference sets.
- The number of key genes per cluster is small, consistent with scQA's iterative minimization of landmarks to approximately a dozen.

## Limitations

- Retaining only 2000 highly variable genes for clustering analysis led to the exclusion of some informative genes, so some meaningful key genes may be absent.
- Genes may be misclassified in the LC1 and LC2 landmark-construction modules due to the influence of high dimensionality.
- Some clusters may lack marker genes entirely — e.g., Cluster8 in the Xenopus lung dataset contained only three cells, likely outliers during clustering.
- scQA may merge biologically distinct cell types into one cluster (endothelial and mesenchymal cells grouped into Cluster3), confounding key-gene specificity.

## Evidence

- [abstract] Effectiveness and potential biological significance of identified key genes: "we confirm the effectiveness and potential biological significance of the identified key genes through both external and internal validation"
- [intro] scQA captures key genes sharing attributes with DEGs, hub genes, and markers: "scQA captures key genes sharing attributes with differentially expressed genes, hub genes and marker genes, serving as reliable landmarks for clustering cells"
- [results] DEG selection criteria: "genes with the absolute value of logFC above 2 and adjusted p-value less than 0.01 were selected as DEGs"
- [abstract] Landmark minimization: "Through an iterative process, our approach aims to minimize the number of landmarks to approximately a dozen while maximizing the inclusion of quasi-trend-preserved genes with dropouts both"
- [readme] README output format: "labels.txt Clustering result."
