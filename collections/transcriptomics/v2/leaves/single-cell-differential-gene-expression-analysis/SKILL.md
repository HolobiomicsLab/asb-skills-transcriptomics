---
name: single-cell-differential-gene-expression-analysis
description: Use when you have a log2-normalized scRNA-seq expression matrix with
  cluster labels and need to interpret clusters biologically via DE genes, GO terms,
  and KEGG pathways.
when_to_use_negative:
- Input is raw counts without cluster labels — no cell grouping exists for pairwise
  differential testing yet
- The goal is to benchmark clustering accuracy (ARI/NMI) rather than interpret cluster
  biology
- Expression data has not been log2-normalized and low-expression genes (detected
  in <2% of cells) have not been filtered
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: scEFSC
  role: Produces the cluster labels (via ensemble feature selection and weighted-based
    meta-clustering) that are interpreted by differential expression and enrichment
    analysis
  repo: https://github.com/Conan-Bian/scEFSC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.04.023
    title: 'scEFSC: Accurate single-cell RNA-seq data analysis via ensemble consensus
      clustering based on multiple feature selections'
  source_content_hash: sha256:b1bd654bb2115304de509a201375f999081d986ed98ea59def9a2853ae69b22b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Conan-Bian/scEFSC
  repo_url: https://github.com/Conan-Bian/scEFSC
  tools:
  - scEFSC
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# single-cell-differential-gene-expression-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Establishes biological interpretability of scRNA-seq cluster labels (e.g., from the scEFSC wMetaC ensemble) by running pairwise differential gene expression testing followed by GO and KEGG enrichment analysis on the significant gene lists.

## When to use

Use when you have a log2-normalized scRNA-seq expression matrix with cluster labels and need to interpret clusters biologically via DE genes, GO terms, and KEGG pathways.

## When NOT to use

- Input is raw counts without cluster labels — no cell grouping exists for pairwise differential testing yet
- The goal is to benchmark clustering accuracy (ARI/NMI) rather than interpret cluster biology
- Expression data has not been log2-normalized and low-expression genes (detected in <2% of cells) have not been filtered

## Inputs

- log2-normalized scRNA-seq expression matrix (rows=genes, columns=cells; e.g., GSE83139 human pancreas)
- final cluster labels from scEFSC weighted-based meta-clustering (wMetaC)
- gene lists of top differentially expressed genes per cluster (e.g., top twenty per cluster)

## Outputs

- dge_results.csv with per-cluster-comparison DE genes and adjusted p-values
- go_kegg_enrichment.csv with GO terms and KEGG pathways including term, gene count, p-value, adjusted p-value

## How to apply

Load the log2-normalized expression matrix (e.g., GSE83139 human pancreas) together with the final cluster labels produced by the scEFSC weighting-based meta-clustering step. For each pair of clusters, rank genes with a statistical test such as the Wilcoxon rank-sum test between the two cell groups, then apply multiple-testing correction to per-gene p-values and retain significant DE genes per comparison (dge_results.csv). Submit the significant gene lists to GO enrichment across biological process, molecular function, and cellular component categories, and run KEGG pathway enrichment on the same lists to find over-represented signaling and metabolic pathways. Write enrichment statistics (term, gene count, p-value, adjusted p-value) to go_kegg_enrichment.csv. In the paper this yielded 533 enriched GOs (465 BP, 31 CC, 37 MF) and 9 KEGG pathways with 70 annotated genes from the top twenty DE genes of nine clusters.

## Related tools

- **scEFSC** (Produces the cluster labels (via ensemble feature selection and weighted-based meta-clustering) that are interpreted by differential expression and enrichment analysis) — https://github.com/Conan-Bian/scEFSC

## Examples

```
dge <- apply(combn(levels(clusters), 2), 2, function(p) wilcox.test(expr[, clusters==p[1]], expr[, clusters==p[2]])); enrichGO(gene=sig_genes, OrgDb=org.Hs.eg.db, ont="BP"); enrichKEGG(gene=sig_genes, organism="hsa")
```

## Evaluation signals

- dge_results.csv exists and contains per-cluster-comparison DE genes with adjusted p-values
- go_kegg_enrichment.csv exists and contains GO terms and KEGG pathways with enrichment statistics
- GO results span all three categories (BP, CC, MF), comparable to the paper's 533 enriched GOs (465 BP, 31 CC, 37 MF)
- KEGG enrichment returns annotated pathways with gene counts (paper: 9 pathways, 70 genes annotated)
- Enriched terms are consistent with the tissue/disease context (paper: cancer-related biological significance on GSE81861)

## Limitations

- Biological interpretability depends on the quality of the upstream clustering labels; ambiguous cell labels ('abandoned') should be removed beforehand
- The paper demonstrates this on the GSE81861 dataset; pathway gene annotation coverage (e.g., 70 genes across 9 KEGG pathways) limits interpretability for small gene lists

## Evidence

- [results] From the top twenty differentially expressed genes of the nine clusters, GO enrichment analysis yielded 533 enriched GOs (465 BP, 31 CC, 37 MF) and KEGG analysis matched 9 pathways with 70 genes annotated: [evidence span withheld — anchor status: not_found]
