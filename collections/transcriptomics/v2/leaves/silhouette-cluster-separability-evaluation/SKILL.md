---
name: silhouette-cluster-separability-evaluation
description: Use when you have annotated single-cell expression matrices (e.g. Jurkat-293T
  with CD3D/XIST marker annotations, Zeisel with nine major classes) in raw vs. imputed
  versions and need to assess whether imputation improves cell-type subpopulation
  separability.
when_to_use_negative:
- Cell-type labels are unavailable or unreliable — silhouette scoring requires pre-defined
  clusters per cell and cannot be computed from unannotated data
- The goal is expression-value recovery accuracy (masked matrix recovery) rather than
  visual/quantitative separability; use the paper's subsampling recovery evaluation
  instead
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: AutoImpute
  role: Produces the imputed count matrices that are compared against the unimputed
    data and competing methods (scImpute, drImpute, MAGIC) in the silhouette/PCA separability
    analysis
  repo: https://github.com/divyanshu-talwar/AutoImpute
- name: scikit-learn
  role: Supplies PCA for 2D projection and silhouette_score for the average silhouette
    index over all cells
- name: matplotlib
  role: Renders the 2D PCA scatter plots of cell transcriptomes colored by cell-type
    annotation
- name: numpy
  role: Array handling for expression matrices in the PCA and silhouette computations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-018-34688-x
    title: 'AutoImpute: Autoencoder based imputation of single-cell RNA-seq data'
  source_content_hash: sha256:ff3e66a2b17cdab3bb9f9357f1eef143a36e8c940d3e55791e3d26202e637abc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/divyanshu-talwar/AutoImpute
  tools:
  - AutoImpute
  - matplotlib
  - numpy
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# silhouette-cluster-separability-evaluation

## Summary

Quantify cell-type separability in scRNA-seq data by projecting log-transformed expression matrices into 2D PCA space and computing the average silhouette index per imputation condition, as used in the AutoImpute paper to show that AutoImpute groups same cell-types together with improved separability (Fig. 5, Supplementary Table S4).

## When to use

Use when you have annotated single-cell expression matrices (e.g. Jurkat-293T with CD3D/XIST marker annotations, Zeisel with nine major classes) in raw vs. imputed versions and need to assess whether imputation improves cell-type subpopulation separability.

## When NOT to use

- Cell-type labels are unavailable or unreliable — silhouette scoring requires pre-defined clusters per cell and cannot be computed from unannotated data
- The goal is expression-value recovery accuracy (masked matrix recovery) rather than visual/quantitative separability; use the paper's subsampling recovery evaluation instead

## Inputs

- Log-transformed scRNA-seq count matrix (CSV) with rows = cells, columns = genes, e.g. Jurkat-293T (3,388 cells) or Zeisel (3,005 cells)
- Imputed count matrices of the same dimensions from each method (AutoImpute, scImpute, drImpute, MAGIC)
- Per-cell cell-type annotations (e.g. CD3D/XIST marker calls for Jurkat vs. 293T; nine major classes for Zeisel)

## Outputs

- 2D PCA scatter plots colored by cell-type, e.g. figure5a_jurkat_pca.png and figure5b_zeisel_pca.png
- Per-dataset, per-method average silhouette values written to silhouette_values.csv (one row per dataset and imputation condition)

## How to apply

Load the log-transformed expression matrices (e.g. Jurkat-293T, 3,388 cells annotated by CD3D/XIST markers; Zeisel, 3,005 cells across nine major classes) together with the imputed matrices from each method (AutoImpute, scImpute, drImpute, MAGIC). For each dataset and imputation condition, apply scikit-learn PCA to reduce the gene expression matrix to two dimensions and scatter-plot cells colored by cell-type annotation, matching Fig. 5 panels (a) Jurkat-293T and (b) Zeisel. Compute the average silhouette index over all cells with scikit-learn's silhouette_score, setting k equal to the number of unique cell-types in each dataset. The rationale is that a higher average silhouette indicates that cells of the same type cluster more tightly after imputation, so a method improving separability (as AutoImpute does) should raise the silhouette value relative to the unimputed data.

## Related tools

- **AutoImpute** (Produces the imputed count matrices that are compared against the unimputed data and competing methods (scImpute, drImpute, MAGIC) in the silhouette/PCA separability analysis) — https://github.com/divyanshu-talwar/AutoImpute
- **scikit-learn** (Supplies PCA for 2D projection and silhouette_score for the average silhouette index over all cells)
- **matplotlib** (Renders the 2D PCA scatter plots of cell transcriptomes colored by cell-type annotation)
- **numpy** (Array handling for expression matrices in the PCA and silhouette computations)

## Examples

```
python AutoImpute.py --data jurkat_293T.csv --hidden_units 2000 --lambda_val 1 --initial_learning_rate 0.0001
```

## Evaluation signals

- silhouette_values.csv contains one row per dataset and per imputation condition (raw, AutoImpute, scImpute, drImpute, MAGIC)
- AutoImpute's average silhouette index is higher than the unimputed data's, consistent with Supplementary Table S4
- figure5a_jurkat_pca.png and figure5b_zeisel_pca.png render cells colored by annotation in 2D PCA space, with same cell-types visibly grouped after AutoImpute
- k for silhouette_score equals the number of unique cell-types (2 for Jurkat-293T via CD3D/XIST, 9 for Zeisel classes)
- Imputed matrices have the same dimensions as the input log-transformed matrices (rows = cells, columns = genes)

## Limitations

- Silhouette values depend on the provided cell-type annotations; misannotated cells (e.g. ambiguous CD3D/XIST marker calls) will depress scores independent of imputation quality
- 2D PCA may not capture all separability structure; the analysis is a visualization-plus-metric complement to the paper's other evaluations (expression recovery, clustering accuracy, variance stabilization)

## Evidence

- [results] AutoImpute groups the same cell-types together showing improved separability: [evidence span withheld — anchor status: not_found]
- [abstract] AutoImpute performed competitively wrt., the existing single-cell imputation methods: "AutoImpute performed competitively wrt., the existing single-cell imputation methods, on the grounds of expression recovery from subsampled data, cell-clustering accuracy, variance stabilization and"
- [readme] The input to AutoImpute.py is a pre-processed count single cell matrix: "The input to `AutoImpute.py` is a pre-processed count single cell matrix, with columns representing genes and rows representing cells. It's output is an imputed count matrix with the same dimension."
