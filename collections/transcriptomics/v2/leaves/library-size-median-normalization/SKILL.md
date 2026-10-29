---
name: library-size-median-normalization
description: Use when a raw scRNA-seq count matrix (.csv, cells as rows, genes as
  columns) must be library-size normalized with median-of-totals scaling to make counts
  comparable across cells prior to imputation or downstream analysis.
when_to_use_negative:
- The matrix is already pre-processed and stored in the <dataset_name>.{mat, csv}
  format consumed by AutoImpute.py — re-normalizing would double-scale the counts.
- The analysis requires per-gene rather than per-cell normalization, or a normalization
  scheme incompatible with feeding AutoImpute (e.g. SCTransform-style variance stabilization
  on raw counts).
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: AutoImpute
  role: Pre-processing R scripts (pre-process.R) implement gene filtering, median
    normalization, top-1000 dispersion gene selection, and log2 transform before the
    AutoImpute autoencoder imputation model runs
  repo: github.com/divyanshu-talwar/AutoImpute
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:558c8097e98924114d25b653c8068e752b05c833df394be22e5f23fef3d64b47
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Library-size median normalization

## Summary

Normalizes a raw scRNA-seq count matrix by dividing each read count by its cell's total counts and scaling by the median of total read counts across cells, as implemented in the AutoImpute pre-processing pipeline before autoencoder imputation.

## When to use

Use when a raw scRNA-seq count matrix (.csv, cells as rows, genes as columns) must be library-size normalized with median-of-totals scaling to make counts comparable across cells prior to imputation or downstream analysis.

## When NOT to use

- The matrix is already pre-processed and stored in the <dataset_name>.{mat, csv} format consumed by AutoImpute.py — re-normalizing would double-scale the counts.
- The analysis requires per-gene rather than per-cell normalization, or a normalization scheme incompatible with feeding AutoImpute (e.g. SCTransform-style variance stabilization on raw counts).

## Inputs

- Raw scRNA-seq count matrix as .csv (cells as rows, genes as columns), placed in AutoImpute\ Model/data/raw/
- Input file name and dataset name arguments passed to pre-process.R

## Outputs

- Median-normalized, gene-filtered, log2-transformed matrix stored as <dataset_name>.{mat, csv} in AutoImpute\ Model/data/

## How to apply

Load the raw count matrix from AutoImpute\ Model/data/raw/ as a .csv file, then first filter genes, retaining only those detected with >=3 reads in at least three cells. Perform median normalization by dividing each read count by the total counts in its cell, then multiplying by the median of the total read counts across cells; this removes cell-level capture-depth differences. Continue the AutoImpute pre-processing by selecting the top 1000 genes ranked by dispersion (coefficient of variance) and log2-transforming after adding 1 as a pseudo-count. The normalized matrix is produced by the R script pre-process.R and saved in the <dataset_name>.{mat, csv} format that AutoImpute.py consumes.

## Related tools

- **AutoImpute** (Pre-processing R scripts (pre-process.R) implement gene filtering, median normalization, top-1000 dispersion gene selection, and log2 transform before the AutoImpute autoencoder imputation model runs) — github.com/divyanshu-talwar/AutoImpute

## Examples

```
Rscript pre-process.R Blakeley_raw_data.csv blakeley
```

## Evaluation signals

- Output matrix contains exactly 1000 gene columns corresponding to the highest-dispersion genes
- Output is saved in the pre-processed <dataset_name>.{mat, csv} format consumed by AutoImpute.py
- Each cell's normalized profile is scaled so its total equals the median of total read counts across cells
- Un-expressed genes (fewer than >=3 reads in at least three cells) are absent from the output matrix
- Downstream `python AutoImpute.py --data <dataset_name>.{mat, csv}` runs without dimension or format errors

## Limitations

- The pre-processing pipeline is implemented in R (Python 2.7 is used only for the AutoImpute model itself), so R with the listed dependencies (R.matlab, Matrix, dplyr, data.table, etc.) is required
- Selection of the top 1000 genes by dispersion discards all other genes, which may not suit analyses requiring the full transcriptome
- No discussion section content was available in the source document, so caveats about the normalization choice itself are not elaborated in the article

## Evidence

- [methods] Perform median normalization: divide each read count by the total counts in its cell, then multiply by the median of the total read counts across cells.: [evidence span withheld — anchor status: not_found]
- [readme] The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells.: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells."
- [methods] Apply gene filtering: retain only genes detected with >=3 reads in at least three cells, discarding un-expressed genes.: [evidence span withheld — anchor status: not_found]
