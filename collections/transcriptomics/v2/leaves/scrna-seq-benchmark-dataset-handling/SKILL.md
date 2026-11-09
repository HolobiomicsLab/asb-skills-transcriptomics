---
name: scrna-seq-benchmark-dataset-handling
description: Use when you have a ranked DEG list from singleCellHaystack's `haystack`
  function (sorted by log.p.vals via show_result_haystack) and want to validate that
  top-ranked genes are enriched for known, high-confidence cell-type marker genes.
when_to_use_negative:
- The expression data is continuous normalized counts rather than binary detection
  data — the current singleCellHaystack implementation requires binary detection data
- The goal is clustering-based DEG calling or per-cluster marker testing rather than
  assessing enrichment of known markers among a clustering-independent ranked DEG
  list
- No ranked gene list exists (e.g. input is an unranked feature table or a gene set
  without p-values), so rank bins of 50 cannot be constructed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: singleCellHaystack
  role: Produces the ranked DEG list (via haystack and show_result_haystack) whose
    top ranks are interrogated for marker enrichment; the R package includes functions
    for clustering and visualization of genes
  repo: github.com/alexisvdb/singleCellHaystack
- name: R
  role: Implementation environment for running singleCellHaystack and tabulating per-bin
    marker frequencies into marker_enrichment_by_rank.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:6310daff1fe5df8635feb1f2b5fc146bdc73b7c9cb2daeb7a14587ecc3140bd0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# scrna-seq-benchmark-dataset-handling

## Summary

A benchmarking/validation skill for singleCellHaystack DEG rankings: known cell-type marker databases (CellMarker) are used to quantify how enriched biologically credible genes are among the top-ranked differentially expressed genes produced across real scRNA-seq datasets.

## When to use

Use when you have a ranked DEG list from singleCellHaystack's `haystack` function (sorted by log.p.vals via show_result_haystack) and want to validate that top-ranked genes are enriched for known, high-confidence cell-type marker genes.

## When NOT to use

- The expression data is continuous normalized counts rather than binary detection data — the current singleCellHaystack implementation requires binary detection data
- The goal is clustering-based DEG calling or per-cluster marker testing rather than assessing enrichment of known markers among a clustering-independent ranked DEG list
- No ranked gene list exists (e.g. input is an unranked feature table or a gene set without p-values), so rank bins of 50 cannot be constructed

## Inputs

- Ranked DEG list from singleCellHaystack's haystack function (sorted by log.p.vals), e.g. from Tabula Muris-style scRNA-seq datasets or the example figshare datasets
- CellMarker mouse marker gene database with per-marker publication counts
- Binary detection expression matrix (gene detected/not-detected per cell) and input coordinates (t-SNE, PCA first 20 PCs, or spatial 2D coordinates)

## Outputs

- marker_enrichment_by_rank.csv: per-rank-bin frequencies of high-confidence markers, with marker-class annotation and rank-bin index
- Validation summary showing top-50 bin marker frequency (~32.4% expected) versus declining frequency in tail bins

## How to apply

Download the CellMarker mouse marker gene database, which records for each marker the number of publications reporting it, and split markers into a high-confidence class (reported in >=5 publications) and a low-confidence class (1-4 publications). Obtain the ranked DEG list from singleCellHaystack by running `haystack` on the input coordinates (e.g. t-SNE or the first 20 principal components) with the binary detection matrix, and retrieve results with `show_result_haystack` sorted by log.p.vals. Partition the ranked list into consecutive bins of 50 ranks and count the frequency of high-confidence markers per bin, annotating each bin with marker class and rank-bin index. Write the per-bin frequency table to marker_enrichment_by_rank.csv. The rationale is that high-confidence markers are only 2.2% of all genes, so a top-50 bin dominated by markers indicates the Kullback-Leibler divergence-based ranking is biologically meaningful.

## Related tools

- **singleCellHaystack** (Produces the ranked DEG list (via haystack and show_result_haystack) whose top ranks are interrogated for marker enrichment; the R package includes functions for clustering and visualization of genes) — github.com/alexisvdb/singleCellHaystack
- **R** (Implementation environment for running singleCellHaystack and tabulating per-bin marker frequencies into marker_enrichment_by_rank.csv)

## Examples

```
res <- haystack(dat.tsne, dat.expression)
```

## Evaluation signals

- The first bin (top 50 ranks) must show a high-confidence marker frequency consistent with the reported ~32.4% average enrichment observed across the 136 analyzed transcriptome datasets
- High-confidence marker frequency should monotonically (or near-monotonically) decrease toward the tail of the ranked DEG list
- marker_enrichment_by_rank.csv contains one row per rank bin with a marker-class annotation column and the rank-bin index
- The high-confidence marker class is defined strictly as markers reported in >=5 publications; the low-confidence class covers 1-4 publications
- Sanity-check that high-confidence markers constitute only ~2.2% of all genes in the background set before binning

## Limitations

- singleCellHaystack requires binary detection data as input (a gene is either detected or not in each cell); continuous expression measures are not supported, and a hard detection threshold is a weak point — genes with different expression levels but the same detection pattern get identical DKL and p-value
- The detection threshold definition is left to the user, introducing potential arbitrariness
- P-values can be inflated because input coordinates (PCs, t-SNE, UMAP) reuse the gene expression data; this issue was deferred to future updates
- Many zero counts may reflect dropout events from lack of sensitivity; imputation was not applied

## Evidence

- [full_text] High-confidence CellMarker genes (reported in >=5 publications), though only 2.2% of all genes, made up on average 32.4% of the top 50 ranked DEGs across the 136 datasets (Fig. 5): [evidence span withheld — anchor status: not_found]
- [abstract] We illustrate the usage of singleCellHaystack through applications on 136 real transcriptome datasets and a spatial transcriptomics dataset: "We illustrate the usage of singleCellHaystack through applications on 136 real transcriptome datasets and a spatial transcriptomics dataset"
- [methods] res <- haystack(dat.tsne, dat.expression): "res <- haystack(dat.tsne, dat.expression)"
- [readme] It does so without relying on clustering of samples into arbitrary clusters.: [evidence span withheld — anchor status: external_doc]
- [discussion] The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell).: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
