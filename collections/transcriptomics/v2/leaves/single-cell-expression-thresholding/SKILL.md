---
name: single-cell-expression-thresholding
description: Use when you need to determine the neighborhood size K for a nearest-neighbor
  graph, or when a binary 'how many geneset genes are expressed past a threshold'
  diagnostic per cell is needed for scRNA-seq AnnData data.
when_to_use_negative:
- You need ranked, continuous per-cell gene set scores (e.g. singscore, ssGSEA, rank_biased_overlap)
  rather than a binary thresholded overlap count — those scoring methods operate on
  ranked data instead of thresholded expression.
- The input is bulk expression data rather than per-cell single-cell data — this diagnostic
  is defined per cell on a scRNA-seq AnnData object.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: gssnng
  role: Provides the geneset_overlap scoring function that counts (or fractions) geneset
    genes expressed past a given threshold per cell
  repo: github.com/IlyaLab/gssnng
- name: Scanpy
  role: Loads the processed AnnData object (sc.datasets.pbmc3k_processed) and supplies
    adata.X expression values for thresholding
- name: AnnData
  role: Container format for the single-cell expression matrix and the resulting per-cell
    score columns
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad150
    title: Single-cell gene set scoring with nearest neighbor graph smoothed data
      (gssnng)
  source_content_hash: sha256:1be36f712e48766cd5ee1d0d51800f00d545ef9965809780a867cbb9e8267c50
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - Python
  - Scanpy
  - gssnng
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# single-cell-expression-thresholding

## Summary

Counts (or computes the fraction of) gene-set genes whose expression exceeds a user-given threshold in each cell, producing a per-cell, per-gene-set overlap score table used to characterize neighborhood size K for nearest-neighbor graph smoothing in gssnng.

## When to use

Use when you need to determine the neighborhood size K for a nearest-neighbor graph, or when a binary 'how many geneset genes are expressed past a threshold' diagnostic per cell is needed for scRNA-seq AnnData data.

## When NOT to use

- You need ranked, continuous per-cell gene set scores (e.g. singscore, ssGSEA, rank_biased_overlap) rather than a binary thresholded overlap count — those scoring methods operate on ranked data instead of thresholded expression.
- The input is bulk expression data rather than per-cell single-cell data — this diagnostic is defined per cell on a scRNA-seq AnnData object.

## Inputs

- Scanpy AnnData object (.h5ad) with expression values in adata.X (e.g. sc.datasets.pbmc3k_processed())
- Gene set file in GMT format (e.g. cibersort_lm22.gmt) with MSigDB nomenclature and default-UP / _UP / _DN direction suffixes
- User-given expression threshold for 'expressed' genes

## Outputs

- geneset_overlap_scores.csv: table with one row per cell and one column per gene set
- Per-cell count and fraction of geneset genes passing the expression threshold (fraction values in [0, 1])

## How to apply

Load the processed single-cell matrix as a Scanpy AnnData object (e.g. sc.datasets.pbmc3k_processed()) and take expression values from adata.X. Parse the gene sets from a .gmt file following the MSigDB nomenclature (e.g. cibersort_lm22.gmt), honoring the default-UP / _UP / _DN direction suffix conventions. For each cell and each gene set, count the signature genes with expression above the user-given threshold and record both the count and the fraction of geneset genes passing; in smoothing applications, sweep K and choose the value where the overlap score plateaus. Assemble one row per cell and one column per gene set, then save the table (e.g. geneset_overlap_scores.csv) for validation.

## Related tools

- **gssnng** (Provides the geneset_overlap scoring function that counts (or fractions) geneset genes expressed past a given threshold per cell) — github.com/IlyaLab/gssnng
- **Scanpy** (Loads the processed AnnData object (sc.datasets.pbmc3k_processed) and supplies adata.X expression values for thresholding)
- **AnnData** (Container format for the single-cell expression matrix and the resulting per-cell score columns)

## Examples

```
import scanpy as sc; q = sc.datasets.pbmc3k_processed(); # then score with gssnng: scores_cells.with_gene_sets(adata=q, gene_set_file='cibersort_lm22.gmt', score_method='geneset_overlap', ...)
```

## Evaluation signals

- geneset_overlap_scores.csv exists and contains one score column per gene set parsed from the .gmt file
- Fraction-mode values fall within [0, 1] for every cell
- Number of rows equals the number of cells in the AnnData object
- Overlap score plateaus with increasing K, and that K is selected as the neighborhood size

## Limitations

- The threshold for calling a gene 'expressed' is user-given, so overlap counts and fractions depend directly on that choice.
- Sparsity and technical noise of single-cell expression measures make raw thresholded signals unreliable, which is why the overlap is used to diagnose smoothing neighborhood size rather than as a final score.
- Gene set direction conventions (default-UP / _UP / _DN) must be parsed correctly from the .gmt names or sets will be misassigned.

## Evidence

- [readme] geneset_overlap: For each geneset, number (or fraction) of genes expressed past a given threshold.: "geneset_overlap: For each geneset, number (or fraction) of genes expressed past a given threshold."
- [intro] The GSSNNG method is based on using the nearest neighbor graph of cells for data smoothing.: "The GSSNNG method is based on using the nearest neighbor graph of cells for data smoothing. This essentially creates mini-pseudobulk expression profiles for each cell"
- [methods] Read in an AnnData object using scanpy: "Read in an AnnData object using scanpy (an h5ad file)."
- [abstract] Sparsity motivates smoothing: "the sparsity and technical noise of single-cell expression measures create difficulties for these methods"
