---
name: scrnaseq-log2-transformation-preprocessing
description: Use when raw scRNA-seq count data (rows = cells, columns = genes) must
  be pre-processed and log2-transformed before dropout imputation or before measuring
  K-means cell-clustering accuracy with ARI.
when_to_use_negative:
- The input is already a pre-processed log2-transformed matrix — rerunning pre-process.R
  would double-transform the counts
- The data are bulk RNA-seq or non-expression matrices; the pipeline is specific to
  sparse single-cell count matrices with dropout events
- Only differential expression or trajectory analysis (not clustering accuracy or
  dropout imputation) is the goal
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: AutoImpute
  role: Autoencoder-based imputation model applied to the pre-processed log2-transformed
    matrix to recover dropouts before K-means clustering
  repo: github.com/divyanshu-talwar/AutoImpute
- name: scikit-learn
  role: Provides the K-means implementation run per dataset with K equal to the number
    of unique cell-types, and the ARI metric against original annotations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:8494426be1151c1de5cdfde4bb68f2bf53aa2c6cd0777ce6e12fa803ea24a913
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# scrnaseq-log2-transformation-preprocessing

## Summary

Preprocesses sparse single-cell RNA-seq count matrices into log2-transformed, cell-by-gene matrices suitable for imputation (e.g. AutoImpute) and downstream K-means clustering evaluated by Adjusted Rand Index against original cell-type annotations.

## When to use

Use when raw scRNA-seq count data (rows = cells, columns = genes) must be pre-processed and log2-transformed before dropout imputation or before measuring K-means cell-clustering accuracy with ARI.

## When NOT to use

- The input is already a pre-processed log2-transformed matrix — rerunning pre-process.R would double-transform the counts
- The data are bulk RNA-seq or non-expression matrices; the pipeline is specific to sparse single-cell count matrices with dropout events
- Only differential expression or trajectory analysis (not clustering accuracy or dropout imputation) is the goal

## Inputs

- Raw single-cell RNA-seq count matrix in .csv format (cells as rows, genes as columns), placed in AutoImpute\ Model/data/raw/
- Pre-processed log2-transformed expression matrices in <dataset_name>.{mat, csv} format
- Original per-cell type annotations for each dataset (Blakeley, Jurkat-293T, Kolodziejczyk, PBMC, Preimplantation, Quake, Usoskin, Zeisel)

## Outputs

- Pre-processed count matrix <dataset_name>.{mat, csv} produced by pre-process.R
- Imputed count matrix of the same dimensions (AutoImpute.py --imputed_save output)
- ari_table.csv: ARI values with rows for the eight datasets and columns for no-imputation, scImpute, drImpute, MAGIC, and AutoImpute

## How to apply

Place the raw count data in .csv format into AutoImpute\ Model/data/raw/ and run the R pre-processing script 'Rscript pre-process.R <input-file-name> <dataset-name>' to produce a pre-processed <dataset_name>.{mat, csv} matrix with cells as rows and genes as columns. Feed this pre-processed count matrix to AutoImpute.py, which learns the inherent distribution of the input scRNA-seq data via an overcomplete autoencoder (default hidden_units 2000, iterations 7000, learning rate 0.0001, convergence threshold 0.0001) and imputes dropouts with minimal modification to biologically silent genes. For clustering benchmarks, run scikit-learn K-means with K set to the number of unique cell-types per dataset on both the unimputed log2-transformed matrix and each imputed matrix, then compute the Adjusted Rand Index against the original cell annotations. For very large datasets (e.g. PBMC, ~68,000 cells) limit evaluation to AutoImpute because scImpute, drImpute, and MAGIC exhaust system memory.

## Related tools

- **AutoImpute** (Autoencoder-based imputation model applied to the pre-processed log2-transformed matrix to recover dropouts before K-means clustering) — github.com/divyanshu-talwar/AutoImpute
- **scikit-learn** (Provides the K-means implementation run per dataset with K equal to the number of unique cell-types, and the ARI metric against original annotations)

## Examples

```
Rscript pre-process.R Blakeley_raw_data.csv blakeley && python AutoImpute.py --data blakeley.csv --hidden_units 2000 --iterations 7000
```

## Evaluation signals

- All ARI values in ari_table.csv fall within [0, 1]
- The ari_table.csv values reproduce the ordering of Table 1 across the eight datasets and five imputation conditions
- AutoImpute attains the best ARI on five of the eight datasets and its ARI always exceeds the no-imputation baseline
- The pre-processed matrix has the same dimensions (cells × genes) as the input count matrix, per the README's stated output format
- For PBMC (~68,000 cells), only the AutoImpute column is populated because the other imputation methods exhaust memory

## Limitations

- For PBMC (~68,000 cells), scImpute, drImpute, and MAGIC exhaust system memory, so comparisons are restricted to AutoImpute
- AutoImpute.py defaults to Python 2.7 with tensorflow, and its documented example dataset is blakeley.csv; other datasets require prior R pre-processing
- The paper notes no discussion-section content was available in the source document, so caveats beyond the benchmark scope are not documented

## Evidence

- [abstract] abstract — competitive performance: "AutoImpute performed competitively wrt., the existing single-cell imputation methods, on the grounds of expression recovery from subsampled data, cell-clustering accuracy, variance stabilization and"
- [intro] intro — overcomplete autoencoder proposal: "In this work, we propose AutoImpute (Fig. 1), a novel method for sparse gene expression matrix imputation using overcomplete autoencoders."
- [readme] readme — pre-processing command: "Rscript pre-process.R <input-file-name> <dataset-name>"
- [readme] readme — input/output format: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells. It's output is an imputed count matrix with the same dimension."
