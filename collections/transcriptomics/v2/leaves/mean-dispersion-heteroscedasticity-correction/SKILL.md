---
name: mean-dispersion-heteroscedasticity-correction
description: Use when preparing a QC-filtered 10X scRNA-seq count matrix for PCA/clustering
  and you need highly variable genes selected with mean-dependence of dispersion corrected
  via expression-level binning.
when_to_use_negative:
- The count matrix has not been quality-filtered (low-quality cells and genes not
  yet removed via read.data), since dispersion estimates would be dominated by noise
- The goal is downstream differential expression or marker calling (get.robust.markers)
  rather than feature selection for PCA/clustering
- Confounder correction is the actual need — cell cycle, ribosomal, mitochondrial,
  and exhaustion scores should be handled by add.confounder.variables instead
edam_operation: http://edamontology.org/operation_3233
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: robustSingleCell
  role: Provides the get.variable.genes function that bins genes by mean expression,
    computes dispersion-scaled statistics within bins, and retains genes exceeding
    one standard deviation from the bin-average dispersion
  repo: github.com/asmagen/robustSingleCell
- name: R
  role: Language in which the pipeline and get.variable.genes step are implemented
    and run
- name: GEOquery
  role: Used upstream to download the raw 10X LCMV data (matrix.mtx, gene.tsv, barcode.tsv)
    that feed the variable-gene selection
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.celrep.2019.10.131
    title: Single-Cell Profiling Defines Transcriptomic Signatures Specific to Tumor-Reactive
      versus Virus-Responsive CD4+ T Cells
  source_content_hash: sha256:a7a07bd7e550fc91724bc7a7b4892238df281189210d655af5126f7df3479dfb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GEOquery
  - R
  - robustSingleCell
  edam_operation: http://edamontology.org/operation_3233
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# mean-dispersion-heteroscedasticity-correction

## Summary

The robustSingleCell get.variable.genes step identifies highly variable genes by binning genes by mean expression and computing dispersion-scaled statistics within each bin, correcting for the mean-dispersion relationship (heteroscedasticity) before selecting genes for downstream PCA and clustering.

## When to use

Use when preparing a QC-filtered 10X scRNA-seq count matrix for PCA/clustering and you need highly variable genes selected with mean-dependence of dispersion corrected via expression-level binning.

## When NOT to use

- The count matrix has not been quality-filtered (low-quality cells and genes not yet removed via read.data), since dispersion estimates would be dominated by noise
- The goal is downstream differential expression or marker calling (get.robust.markers) rather than feature selection for PCA/clustering
- Confounder correction is the actual need — cell cycle, ribosomal, mitochondrial, and exhaustion scores should be handled by add.confounder.variables instead

## Inputs

- QC-filtered count matrix (10X genomics format: matrix.mtx, gene.tsv, barcode.tsv) loaded via read.data
- robustSingleCell project environment initialized with initialize.project (e.g. dataset LCMV1, origin CD44+ cells, experiment Rep1)

## Outputs

- Set of highly variable gene identifiers (variable_genes.csv) restricted to genes passing the binned dispersion threshold
- Variable-gene feature space used as input to the PCA function and clustering analysis

## How to apply

Run get.variable.genes on the QC-filtered expression matrix produced by read.data (e.g. the LCMV1 replicate subsampled to 500 cells). Genes are binned by their mean expression level; within each bin, dispersion is scaled and compared to the bin average, accounting for heteroscedasticity so that high-dispersion high-mean genes are not spuriously favored. Retain genes whose dispersion exceeds one standard deviation from the average dispersion for their expression bin. The resulting gene set (written to variable_genes.csv) is then used as the input feature space for the subsequent PCA step, which itself uses shuffled-data simulations to pick the number of PCs.

## Related tools

- **robustSingleCell** (Provides the get.variable.genes function that bins genes by mean expression, computes dispersion-scaled statistics within bins, and retains genes exceeding one standard deviation from the bin-average dispersion) — github.com/asmagen/robustSingleCell
- **R** (Language in which the pipeline and get.variable.genes step are implemented and run)
- **GEOquery** (Used upstream to download the raw 10X LCMV data (matrix.mtx, gene.tsv, barcode.tsv) that feed the variable-gene selection)

## Examples

```
LCMV1 <- read.data(LCMV1, subsample = 500); LCMV1 <- get.variable.genes(LCMV1)
```

## Evaluation signals

- The output gene set is restricted to genes whose dispersion exceeds one standard deviation from the average dispersion within their mean-expression bin
- variable_genes.csv contains a non-empty set of gene identifiers suitable as input for PCA and clustering
- The selection was computed after read.data QC filtering (subsample = 500), so low-quality cells and genes are absent from the input
- The downstream PCA runs successfully on the selected genes and the PCA function's shuffled-data simulations determine an appropriate number of PCs
- Applying get.variable.genes to both replicates (LCMV1 and LCMV2) yields per-dataset variable gene sets, consistent with clustering each dataset separately

## Limitations

- The dispersion-vs-mean binning corrects heteroscedasticity within one dataset; datasets should be clustered separately to account for dataset-specific technical and biological differences
- The robustSingleCell package is not currently maintained, per the README
- The tutorial workflow downsamples to 500 cells (subsample = 500) and requires at least 8G of memory on slurm, so full-scale runs may need adjusted resources

## Evidence

- [methods] Binned dispersion HVG selection prepares PCA/clustering: "Next, we identify highly variable genes for the following PCA and clustering analyses."
- [readme] read.data QC filtering precedes variable gene selection: "`read.data` function reads the data in 10X genomics format and performs
quality filtering as described in *Magen et al 2019*."
- [readme] get.variable.genes invocation in the tutorial workflow: "LCMV1 <- get.variable.genes(LCMV1) 
exhaustion_markers <- c('Pdcd1', 'Cd244', 'Havcr2', 'Ctla4', 'Cd160', 'Lag3', 'Tigit', 'Cd96')"
- [readme] Datasets clustered separately before integration: "We cluster each dataset separately to account for dataset-specific
technical and biological differences."
- [methods] PCA uses shuffled simulations for PC number choice: "The `PCA` function performs multiple simulation analyses of shuffled data to determine the appropriate number of PCs."
