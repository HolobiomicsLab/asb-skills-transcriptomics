---
name: pca-dimensionality-reduction
description: Use when you have a cell-by-ontology-type score matrix (e.g. pred_Y_all
  from OnClassModel.Predict) or another high-dimensional per-cell feature table and
  need a low-dimensional embedding for clustering-quality benchmarks such as silhouette
  analysis.
when_to_use_negative:
- The goal is expression-based batch correction/integration itself — use Scanorama
  directly rather than PCA on OnClass scores
- Input is raw count expression data without OnClass predicted annotations — PCA on
  expression alone does not leverage Cell Ontology type scores
- Only two or three datasets are being compared and no cell type labels are available,
  since silhouette computation and the significance test require ground-truth labels
edam_operation: http://edamontology.org/operation_2939
tools:
- name: OnClass
  role: Generates the pred_Y_all cell-by-Cell-Ontology-type score matrix that PCA
    is applied to (OnClassModel.Predict with use_normalize=True)
  repo: https://github.com/wangshenguiuc/OnClass
- name: Scanorama
  role: Produces the expression-based comparison embedding (scan_dim = 10) via run_scanorama_multiply_datasets
    against which the PCA embedding silhouette is benchmarked
  repo: https://github.com/brianhie/scanorama
- name: tensorflow
  role: Deep-learning backend required to run the pretrained OnClass models that produce
    the score matrices
  repo: https://github.com/wangshenguiuc/OnClass
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:0ef659d88db486b5adeca48dd97c14f78a954768025239f2f1a95384c11b0967
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - Scanorama
  - tensorflow
  edam_operation: http://edamontology.org/operation_2939
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pca-dimensionality-reduction

## Summary

Reduces high-dimensional per-cell feature matrices to a low-dimensional embedding via PCA — in this workflow, applied to the OnClass cell-by-Cell-Ontology-type score matrix (pred_Y_all) to produce an embedding whose cluster separation by cell type is scored with the silhouette coefficient and compared against Scanorama expression integration.

## When to use

Use when you have a cell-by-ontology-type score matrix (e.g. pred_Y_all from OnClassModel.Predict) or another high-dimensional per-cell feature table and need a low-dimensional embedding for clustering-quality benchmarks such as silhouette analysis.

## When NOT to use

- The goal is expression-based batch correction/integration itself — use Scanorama directly rather than PCA on OnClass scores
- Input is raw count expression data without OnClass predicted annotations — PCA on expression alone does not leverage Cell Ontology type scores
- Only two or three datasets are being compared and no cell type labels are available, since silhouette computation and the significance test require ground-truth labels

## Inputs

- pred_Y_all cell-by-all-Cell-Ontology-type score matrices per dataset (pred_Y_all.released.npy) from OnClassModel.Predict with use_normalize=True
- Expression matrices (cells-by-genes) for the 26 annotated scRNA-seq datasets
- Cell type labels for each cell (ground-truth annotations used for silhouette computation)
- Cell Ontology files (cl.ontology, cell type network, cl.ontology.nlp.emb) loaded with read_ontology_file

## Outputs

- Low-dimensional PCA embedding of the OnClass score vectors (one embedding per dataset)
- Per-dataset silhouette coefficients for the OnClass PCA embedding and the Scanorama embedding
- silhouette_scores.csv with per-dataset silhouette values and the significance P value of the improvement

## How to apply

First generate the cell-by-all-Cell-Ontology-type score matrix by running OnClassModel.Predict with use_normalize=True on each of the 26 annotated scRNA-seq datasets, saving each as pred_Y_all.released.npy. Apply PCA to reduce these pred_Y_all score vectors to a low-dimensional cell embedding (the OnClass-score embedding). Build the expression-based comparison embedding by integrating the same 26 datasets with Scanorama via run_scanorama_multiply_datasets using scan_dim = 10. Compute the silhouette coefficient of both embeddings per dataset using cell type labels, then test the significance of the silhouette improvement of the OnClass embedding over the Scanorama embedding (two-sided test) and write per-dataset values and the P value to silhouette_scores.csv. The rationale is that OnClass predicted cell type scores should cluster cells by biological identity rather than platform artifacts, so PCA on the score matrix is the feature space in which this is quantified.

## Related tools

- **OnClass** (Generates the pred_Y_all cell-by-Cell-Ontology-type score matrix that PCA is applied to (OnClassModel.Predict with use_normalize=True)) — https://github.com/wangshenguiuc/OnClass
- **Scanorama** (Produces the expression-based comparison embedding (scan_dim = 10) via run_scanorama_multiply_datasets against which the PCA embedding silhouette is benchmarked) — https://github.com/brianhie/scanorama
- **tensorflow** (Deep-learning backend required to run the pretrained OnClass models that produce the score matrices) — https://github.com/wangshenguiuc/OnClass

## Examples

```
from sklearn.decomposition import PCA; emb = PCA(n_components=10).fit_transform(pred_Y_all)
```

## Evaluation signals

- The silhouette coefficient of the OnClass score-PCA embedding significantly exceeds the Scanorama expression embedding, matching the reported improvement (P < 7e-293, two-sided test)
- Per-dataset silhouette values for both embeddings are written to silhouette_scores.csv for all 26 datasets
- PCA embedding is computed from the saved pred_Y_all.released.npy score matrices, not from raw expression
- The comparison Scanorama embedding is built at scan_dim = 10 so the dimensionalities of the two embeddings are comparable
- Silhouette computation uses the same cell type labels across both embeddings per dataset

## Limitations

- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph, so score vectors for such types may not separate under PCA
- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states, capping what the score-space embedding can represent
- The Cell Ontology lacks marker gene associations for most terms, which the NLP embedding (cl.ontology.nlp.emb) partially mitigates but does not eliminate

## Evidence

- [other] Reduce the pred_Y_all score vectors to a low-dimensional cell embedding using PCA.: [evidence span withheld — anchor status: not_found]
- [other] Quantifying integration performance with the silhouette coefficient, OnClass-based integration of the 26 datasets significantly improved over expression-based integration using Scanorama (P-value < 7e-293, two-sided test): [evidence span withheld — anchor status: not_found]
- [readme] integrated, genes = scanorama.integrate(datasets, genes_list): "# Integration.
integrated, genes = scanorama.integrate(datasets, genes_list)"
- [intro] OnClass is able to automatically classify cells to any cell type as long as its corresponding term is captured in the Cell Ontology, even if this cell type does not have annotated cells in the: "OnClass is able to automatically classify cells to any cell type as long as its corresponding term is captured in the Cell Ontology, even if this cell type does not have annotated cells in the"
