---
name: highly-variable-gene-selection
description: Use when preprocessing raw UMI count matrices (e.g., GSE115746 macaque
  retina bipolar cells) for DESC clustering, or whenever a compact set of informative
  genes is needed before deep-embedding or low-dimensional representation.
when_to_use_negative:
- Input is already a feature-reduced matrix or a precomputed low-dimensional embedding,
  so no gene filtering is needed
- Analysis requires all genes (e.g., differential expression testing or marker discovery),
  where discarding low-dispersion genes would bias results
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Scanpy
  role: Provides filter_genes_dispersion (sc.pp.filter_genes_dispersion) for highly
    variable gene selection, plus sc.pp.log1p and sc.pp.scale preprocessing
  repo: https://github.com/theislab/scanpy
- name: DESC
  role: Downstream consumer of the filtered, normalized gene matrix; trains the deep
    embedding for clustering and batch-effect removal
  repo: https://github.com/eleozzr/desc
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:51db5e6358a90d014335ab3747f5cc657fbc27a5d804e0b297aaa682203004b9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/theislab/scanpy
  tools:
  - DESC
  - Keras
  - Scanpy
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# highly-variable-gene-selection

## Summary

Selects highly variable genes from scRNA-seq count data using Scanpy's filter_genes_dispersion (e.g., n_top_genes=1000) as a preprocessing step before DESC deep-embedding clustering, reducing noise and dimensionality.

## When to use

Use when preprocessing raw UMI count matrices (e.g., GSE115746 macaque retina bipolar cells) for DESC clustering, or whenever a compact set of informative genes is needed before deep-embedding or low-dimensional representation.

## When NOT to use

- Input is already a feature-reduced matrix or a precomputed low-dimensional embedding, so no gene filtering is needed
- Analysis requires all genes (e.g., differential expression testing or marker discovery), where discarding low-dispersion genes would bias results

## Inputs

- Raw scRNA-seq UMI count matrix (e.g., AnnData object, such as the GSE115746 macaque retina matrix with 21,017 foveal + 9,285 peripheral bipolar cells)
- Cell metadata (macaque id, region id, sample id) for batch-aware analyses

## Outputs

- AnnData/expression matrix restricted to the top highly variable genes (n_top_genes=1000 in the paper)
- Log1p-normalized and scaled (max_value=6) matrix ready for desc.train input

## How to apply

Start from a count matrix and normalize at the cell level by dividing UMI counts per gene by total UMIs per cell, multiplying by 10,000, and log-transforming with sc.pp.log1p. Then select highly variable genes with Scanpy's sc.pp.filter_genes_dispersion (the paper uses n_top_genes=1000), which picks genes with high dispersion across cells. Standardize the retained genes at the gene level with sc.pp.scale(max_value=6) so each gene has zero mean and unit variance before feeding the matrix to desc.train. The rationale is that DESC learns a low-dimensional representation via a stacked autoencoder, so restricting input to variable genes focuses the embedding on biologically informative variation and supports batch-effect removal as long as technical differences stay smaller than biological variation.

## Related tools

- **Scanpy** (Provides filter_genes_dispersion (sc.pp.filter_genes_dispersion) for highly variable gene selection, plus sc.pp.log1p and sc.pp.scale preprocessing) — https://github.com/theislab/scanpy
- **DESC** (Downstream consumer of the filtered, normalized gene matrix; trains the deep embedding for clustering and batch-effect removal) — https://github.com/eleozzr/desc

## Examples

```
sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata, n_top_genes=1000); sc.pp.scale(adata, max_value=6)
```

## Evaluation signals

- The AnnData passed to desc.train contains exactly the selected number of highly variable genes (n_top_genes=1000 in the paper's methods)
- Gene values after preprocessing are log1p-normalized (10,000 counts scaling) and gene-standardized with values clipped at max_value=6
- Downstream DESC clustering achieves high ARI against reference labels (ARI ~0.920 without batch info; 0.919-0.970 with batch definitions on macaque retina data)
- Only genes passing filter_genes_dispersion dispersion ranking are retained; no cell-level filtering loss is expected in this step

## Limitations

- Filtering to a fixed number of top variable genes (e.g., 1000) discards low-dispersion genes, which may drop informative markers for rare cell types
- The article provides no software version numbers for Scanpy; older versions use filter_genes_dispersion while newer Scanpy versions have replaced it, risking non-reproducibility
- For reproducing the paper's DESC results exactly, TensorFlow must be below 2.0 with desc 2.0.3, and preprocessing must match the paper's normalization pipeline

## Evidence

- [discussion] Highly variable gene selection using filter_genes_dispersion from Scanpy: "Highly variable genes are selected using the filter_genes_dispersion function from the Scanpy package25"
- [discussion] Gene-level standardization threshold: "the cell level normalized values for each gene are standardized by subtracting the mean across all cells and divided by the standard deviation across all cells for the given gene."
- [results] Downstream clustering accuracy validating the preprocessing: [evidence span withheld — anchor status: not_found]
- [readme] README on installing desc for reproducing results: "Because desc depend on `tensorflow`, you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper."
