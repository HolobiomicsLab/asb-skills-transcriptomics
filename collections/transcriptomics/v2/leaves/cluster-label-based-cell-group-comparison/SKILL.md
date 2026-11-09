---
name: cluster-label-based-cell-group-comparison
description: Use when you have final per-cell cluster labels from an scRNA-seq consensus
  clustering run and need to characterize each cluster biologically via differential
  expression plus GO/KEGG enrichment.
when_to_use_negative:
- Cluster labels are not yet finalized — this skill presupposes the wMetaC ensemble
  output, not intermediate base-clustering labels
- Input is raw counts without log2 normalization and low-expression filtering (genes
  detected in <2% of cells must be removed first)
- The goal is clustering performance evaluation (ARI/NMI) rather than biological interpretation
  of cluster identity
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scEFSC
  role: Produces the final cluster labels (via multiple unsupervised feature selections
    — Low Variance, Laplacian Score, SPEC, MCFS — feeding clustering algorithms combined
    by weighted-based meta-clustering) that define the cell groups compared for differential
    expression
  repo: https://github.com/Conan-Bian/scEFSC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.04.023
    title: 'scEFSC: Accurate single-cell RNA-seq data analysis via ensemble consensus
      clustering based on multiple feature selections'
  source_content_hash: sha256:cdb60b088cd90bdbcf427ea519900b87ed0e021423602294a568a32ee8c74831
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cluster-label-based-cell-group-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Uses cluster labels produced by scEFSC's weighted-based meta-clustering (wMetaC) step to run pairwise differential gene expression testing (e.g., Wilcoxon rank-sum) on a log2-normalized scRNA-seq matrix, then submits significant gene lists to GO and KEGG enrichment to establish biological interpretability of the clustering, as demonstrated on GSE81861/GSE83139 human pancreas data.

## When to use

Use when you have final per-cell cluster labels from an scRNA-seq consensus clustering run and need to characterize each cluster biologically via differential expression plus GO/KEGG enrichment.

## When NOT to use

- Cluster labels are not yet finalized — this skill presupposes the wMetaC ensemble output, not intermediate base-clustering labels
- Input is raw counts without log2 normalization and low-expression filtering (genes detected in <2% of cells must be removed first)
- The goal is clustering performance evaluation (ARI/NMI) rather than biological interpretation of cluster identity

## Inputs

- log2-normalized scRNA-seq expression matrix (genes x cells), e.g. GSE83139 human pancreas SMARTer data
- final per-cell cluster labels from the scEFSC wMetaC (weighted-based meta-clustering) step
- gene annotation mapping for GO/KEGG term lookup

## Outputs

- dge_results.csv — per-cluster-comparison differentially expressed genes with p-values and adjusted p-values
- go_kegg_enrichment.csv — GO terms and KEGG pathways with term ID, gene count, p-value, adjusted p-value

## How to apply

Load the log2-normalized expression matrix (for the paper this is the log2-transformed GSE83139 human pancreas SMARTer scRNA-seq matrix, after removing genes detected in fewer than 2% of cells) together with the final cluster labels from the scEFSC wMetaC ensemble step. For each pair of clusters, run differential gene expression testing ranking genes by a statistical test such as Wilcoxon rank-sum between the two cell groups, then apply multiple-testing correction to per-gene p-values and retain significant DE genes per comparison in dge_results.csv. Submit the significant gene lists to gene ontology enrichment across biological process, molecular function, and cellular component categories, and run KEGG pathway enrichment on the same lists to identify over-represented signaling and metabolic pathways. Write GO and KEGG statistics (term, gene count, p-value, adjusted p-value) to go_kegg_enrichment.csv; the paper's benchmark found 533 enriched GOs (465 BP, 31 CC, 37 MF) and 9 KEGG pathways with 70 genes annotated from the top twenty DE genes of nine clusters, showing cancer-related biological significance.

## Related tools

- **scEFSC** (Produces the final cluster labels (via multiple unsupervised feature selections — Low Variance, Laplacian Score, SPEC, MCFS — feeding clustering algorithms combined by weighted-based meta-clustering) that define the cell groups compared for differential expression) — https://github.com/Conan-Bian/scEFSC

## Examples

```
p <- apply(norm_expr[, cells_a], 1, function(x) wilcox.test(x, norm_expr[, cells_b])$p.value); padj <- p.adjust(p, method='BH')
```

## Evaluation signals

- dge_results.csv exists and contains per-cluster-comparison DE genes with adjusted p-values (multiple-testing correction applied)
- go_kegg_enrichment.csv contains both GO terms and KEGG pathways with enrichment statistics (term, gene count, p-value, adjusted p-value)
- On the paper's setup (top twenty DE genes of nine clusters), GO analysis should surface enrichment across all three ontologies — in the paper 533 GOs (465 BP, 31 CC, 37 MF)
- KEGG analysis on the same gene lists should identify over-represented pathways (9 pathways with 70 genes annotated in the paper) showing cancer-related biological significance
- DE tests are run pairwise between clusters, so the output should cover cluster-pair comparisons, not one-vs-rest only

## Limitations

- Biological interpretability is only demonstrated on GSE81861; the DGE/GO/KEGG layer does not itself improve clustering accuracy
- Interpretation depends on prior preprocessing: log2 transformation, removal of genes detected in <2% of cells, and the non-negative kernel autoencoder pre-selection of 5000 genes
- For GSE81861, cells labeled ambiguously as 'abandoned' were removed before analysis

## Evidence

- [abstract] we established the biological interpretability of scEFSC by carrying out differential gene expression analysis, gene ontology enrichment and KEGG analysis: "we established the biological interpretability of scEFSC by carrying out differential gene expression analysis, gene ontology enrichment and KEGG analysis"
- [intro] we realized differential gene expression analysis, gene ontology enrichment and KEGG analysis, demonstrating the biological interpretability of scEFSC: "we realized differential gene expression analysis, gene ontology enrichment and KEGG analysis, demonstrating the biological interpretability of scEFSC"
- [readme] To begin data pre-processing was performed using a log2 transformation to normalize the data and then genes detected in the normalized data in less than 2% of the cells were removed: "To begin data pre-processing was performed using a log2 transformation to normalize the data and then genes detected in the normalized data in less than 2% of the cells were removed"
- [abstract] the clustering results are combined using weighted-based meta-clustering: "the clustering results are combined using weighted-based meta-clustering"
