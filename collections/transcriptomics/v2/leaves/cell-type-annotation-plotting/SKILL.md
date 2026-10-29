---
name: cell-type-annotation-plotting
description: Use when comparing imputed vs. unimputed scRNA-seq matrices (e.g., Jurkat-293T
  or Zeisel datasets) to assess whether imputation improves clustering of annotated
  cell-type subpopulations, as in Fig. 5 of the AutoImpute paper.
when_to_use_negative:
- The expression matrix has no cell-type annotations, so silhouette scoring against
  a defined k is impossible.
- The goal is dropout detection or expression recovery from subsampled data rather
  than post-imputation cell-type separability assessment.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutoImpute
  role: Produces the imputed expression matrices upstream that are evaluated for cell-type
    separability; run with --data <dataset_name>.csv, --hidden_units 2000, --lambda_val
    1, --initial_learning_rate 0.0001
  repo: github.com/divyanshu-talwar/AutoImpute
- name: scikit-learn
  role: Provides PCA for 2D dimensionality reduction and silhouette_score for the
    quantitative separability metric
- name: matplotlib
  role: Renders the 2D PCA scatter plots colored by cell-type annotation
- name: numpy
  role: Array handling for the expression and imputed matrices prior to PCA and silhouette
    computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:1bd5f770be75c96b84b276810b8f122de6497ff5ded417744e0874bb9d46a506
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AutoImpute
  - matplotlib
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cell-type-annotation-plotting

## Summary

Quantify and visualize cell-type separability in scRNA-seq data by projecting log-transformed expression matrices (before and after imputation) into 2D PCA space colored by cell-type annotations, and computing average silhouette scores to compare imputation methods such as AutoImpute, scImpute, drImpute, and MAGIC.

## When to use

Use when comparing imputed vs. unimputed scRNA-seq matrices (e.g., Jurkat-293T or Zeisel datasets) to assess whether imputation improves clustering of annotated cell-type subpopulations, as in Fig. 5 of the AutoImpute paper.

## When NOT to use

- The expression matrix has no cell-type annotations, so silhouette scoring against a defined k is impossible.
- The goal is dropout detection or expression recovery from subsampled data rather than post-imputation cell-type separability assessment.

## Inputs

- log-transformed scRNA-seq expression matrix (cells x genes, .csv) e.g. Jurkat-293T (3,388 cells) or Zeisel (3,005 cells)
- imputed expression matrices from each method (AutoImpute, scImpute, drImpute, MAGIC) in the same cells x genes layout
- cell-type annotation vector per cell (e.g., CD3D/XIST marker labels for Jurkat-293T; nine major classes for Zeisel)

## Outputs

- silhouette_values.csv with one row per dataset and imputation condition containing the average silhouette index
- figure5a_jurkat_pca.png — 2D PCA scatter of Jurkat-293T cells colored by cell-type
- figure5b_zeisel_pca.png — 2D PCA scatter of Zeisel cells colored by cell-type

## How to apply

Load the log-transformed expression matrices and their imputed counterparts (produced upstream with AutoImpute.py using --hidden_units 2000, --lambda_val 1, --initial_learning_rate 0.0001). For each dataset-condition pair, apply scikit-learn PCA to reduce the gene-by-cell matrix to two dimensions, then scatter-plot cells colored by annotation (CD3D/XIST markers for Jurkat-293T; nine major classes for Zeisel), reproducing Fig. 5 panels (a) and (b). Compute the average silhouette index with sklearn.metrics.silhouette_score, with k equal to the number of unique cell-types, for every condition. Write per-dataset, per-method silhouette values to silhouette_values.csv and save the 2D plots as figure5a_jurkat_pca.png and figure5b_zeisel_pca.png. The rationale is that a good imputation method (AutoImpute) groups same cell-types together, improving quantitative separability over the unimputed baseline.

## Related tools

- **AutoImpute** (Produces the imputed expression matrices upstream that are evaluated for cell-type separability; run with --data <dataset_name>.csv, --hidden_units 2000, --lambda_val 1, --initial_learning_rate 0.0001) — github.com/divyanshu-talwar/AutoImpute
- **scikit-learn** (Provides PCA for 2D dimensionality reduction and silhouette_score for the quantitative separability metric)
- **matplotlib** (Renders the 2D PCA scatter plots colored by cell-type annotation)
- **numpy** (Array handling for the expression and imputed matrices prior to PCA and silhouette computation)

## Examples

```
python AutoImpute.py --data jurkat.csv --hidden_units 2000 --lambda_val 1 --initial_learning_rate 0.0001; python plot_pca_silhouette.py --dataset jurkat --methods autoimpute scimpute drimpute magic --out figure5a_jurkat_pca.png --silhouette silhouette_values.csv
```

## Evaluation signals

- silhouette_values.csv contains exactly one row per dataset and imputation condition (unimputed plus each of AutoImpute, scImpute, drImpute, MAGIC).
- AutoImpute's average silhouette index exceeds the unimputed data's value for both Jurkat-293T and Zeisel, consistent with Supplementary Table S4.
- figure5a_jurkat_pca.png and figure5b_zeisel_pca.png render cells colored by the correct annotation scheme (CD3D/XIST markers; nine major classes respectively).
- Imputed matrices have the same cells x genes dimensions as the input log-transformed matrices.

## Limitations

- Evaluation depends on pre-existing, trusted cell-type annotations; mislabeled cells will distort silhouette values and PCA coloring.
- Silhouette index and 2D PCA are coarse global summaries; local subpopulation structure may not be captured.
- Imputed matrices must be generated with the documented AutoImpute parameters; deviations change the separability comparison.

## Evidence

- [results] In the 2D PCA visualization and average silhouette values for the Jurkat-293T and Zeisel datasets before and after imputation, AutoImpute groups the same cell-types together showing improved separability.: [evidence span withheld — anchor status: not_found]
- [readme] AutoImpute.py CLI parameters for upstream imputation: "--hidden_units 2000	# Size of hidden layer or latent space dimensions."
- [results] silhouette computed with scikit-learn: [evidence span withheld — anchor status: not_found]
- [abstract] AutoImpute competitive performance on cell-type separability: "AutoImpute performed competitively wrt., the existing single-cell imputation methods, on the grounds of expression recovery from subsampled data, cell-clustering accuracy, variance stabilization and"
