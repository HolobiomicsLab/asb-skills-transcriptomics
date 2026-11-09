---
name: gene-ontology-enrichment-analysis
description: Use when you have cluster labels from single-cell RNA-seq clustering
  (e.g., scEFSC wMetaC output) plus a normalized expression matrix, and need to interpret
  clusters biologically via GO/KEGG enrichment of top DE genes.
when_to_use_negative:
- Input clustering lacks significant differentially expressed genes after multiple-testing
  correction, leaving no gene lists to submit for enrichment
- The analysis goal is clustering accuracy evaluation (ARI/NMI) rather than biological
  interpretation of cluster labels
- Data are not scRNA-seq gene expression clusters (e.g., proteomics or variant-level
  data where GO gene-set enrichment is not the interpretive frame)
edam_operation: http://edamontology.org/operation_3672
edam_topics:
- http://edamontology.org/topic_0089
- http://edamontology.org/topic_3308
tools:
- name: scEFSC
  role: Produces the cluster labels and the differential gene expression / GO / KEGG
    interpretability workflow demonstrated in the paper
  repo: https://github.com/Conan-Bian/scEFSC
- name: Seurat
  role: Referenced single-cell clustering method whose labels could similarly feed
    downstream GO/KEGG enrichment (intro-level comparison tool)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2022.04.023
    title: 'scEFSC: Accurate single-cell RNA-seq data analysis via ensemble consensus
      clustering based on multiple feature selections'
  source_content_hash: sha256:bfb873b8969545120250f46dfdba8f44963a68ec7983d91fbe9a0d8d29192859
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
  - Seurat
  - scEFSC
  edam_operation: http://edamontology.org/operation_3672
  edam_topics:
  - http://edamontology.org/topic_0089
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# gene-ontology-enrichment-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Establishes biological interpretability of single-cell clustering labels by submitting significant differentially expressed gene lists per cluster to Gene Ontology enrichment (BP, CC, MF) and KEGG pathway analysis, as demonstrated on scEFSC clusters of GSE81861/GSE83139 scRNA-seq data.

## When to use

Use when you have cluster labels from single-cell RNA-seq clustering (e.g., scEFSC wMetaC output) plus a normalized expression matrix, and need to interpret clusters biologically via GO/KEGG enrichment of top DE genes.

## When NOT to use

- Input clustering lacks significant differentially expressed genes after multiple-testing correction, leaving no gene lists to submit for enrichment
- The analysis goal is clustering accuracy evaluation (ARI/NMI) rather than biological interpretation of cluster labels
- Data are not scRNA-seq gene expression clusters (e.g., proteomics or variant-level data where GO gene-set enrichment is not the interpretive frame)

## Inputs

- log2-normalized scRNA-seq expression matrix (genes x cells, e.g., GSE83139 human pancreas or GSE81861 human tissues)
- final cluster labels per cell from the scEFSC wMetaC ensemble clustering step
- significant DE gene lists per cluster comparison with adjusted p-values

## Outputs

- dge_results.csv: per-cluster-comparison differentially expressed genes with adjusted p-values
- go_kegg_enrichment.csv: GO terms and KEGG pathways with enrichment statistics (term, gene count, p-value, adjusted p-value)

## How to apply

Load the log2-normalized scRNA-seq expression matrix together with the final cluster labels from the scEFSC weighting-based meta-clustering step. For each pair of clusters, rank genes with a per-gene statistical test such as the Wilcoxon rank-sum test between the two cell groups, then apply multiple-testing correction and retain significantly differentially expressed genes. From the top twenty differentially expressed genes of each cluster, perform GO enrichment over biological process (BP), molecular function (MF), and cellular component (CC) categories, and run KEGG pathway enrichment on the same gene lists. Record term, gene count, p-value, and adjusted p-value in go_kegg_enrichment.csv, with DE results in dge_results.csv. In the paper's run this yielded 533 enriched GO terms (465 BP, 31 CC, 37 MF) and 9 KEGG pathways with 70 annotated genes, showing cancer-related biological significance.

## Related tools

- **scEFSC** (Produces the cluster labels and the differential gene expression / GO / KEGG interpretability workflow demonstrated in the paper) — https://github.com/Conan-Bian/scEFSC
- **Seurat** (Referenced single-cell clustering method whose labels could similarly feed downstream GO/KEGG enrichment (intro-level comparison tool))

## Evaluation signals

- Both output tables exist: dge_results.csv contains per-cluster-comparison DE genes with adjusted p-values, and go_kegg_enrichment.csv contains GO terms and KEGG pathways with enrichment statistics
- GO enrichment covers all three categories, with counts reported per category (expected scale in the paper: 533 GOs = 465 BP + 31 CC + 37 MF)
- KEGG results report matched pathways and the number of annotated genes (paper: 9 pathways, 70 genes annotated)
- All enrichment statistics include term, gene count, p-value, and adjusted p-value columns
- The enrichment output is consistent with the paper's reported biological-interpretability analysis (differential gene expression, GO enrichment, and KEGG analysis)

## Limitations

- The analysis relies on clustering labels being biologically meaningful; poor clustering produces uninterpretable DE gene lists
- Only the top twenty differentially expressed genes per cluster were submitted for enrichment in the paper, limiting sensitivity of pathway detection
- The interpretation is dataset- and tissue-dependent (cancer-related significance was observed on GSE81861 human tissues; a pancreas/diabetes dataset like GSE83139 may yield different pathways)

## Evidence

- [results] GO and KEGG results on nine clusters: [evidence span withheld — anchor status: not_found]
