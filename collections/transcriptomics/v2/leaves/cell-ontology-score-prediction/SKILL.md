---
name: cell-ontology-score-prediction
description: Use when single-cell RNA-seq cells from one or many datasets must be
  scored against the full Cell Ontology with a pretrained OnClass model, so the score
  matrix can serve as an embedding for annotation, clustering, or integration benchmarks
  against expression-based methods like Scanorama.
when_to_use_negative:
- The cell types of interest are outside the Cell Ontology or are topologically identical
  siblings with the same parents in the Cell Ontology graph, which OnClass cannot
  distinguish
- The goal is batch correction of expression data in the presence of many unseen cell
  types — no such methods are developed here, and Scanorama on expression is the baseline
  being compared against
- Input is raw reads or gene expression already organized as an integrated embedding,
  not scRNA-seq expression matrices to be scored against the ontology
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: OnClass
  role: Produces the cell-by-Cell-Ontology-type score matrix (pred_Y_all) via OnClassModel.Predict
    with use_normalize=True, using pretrained models and Cell Ontology files
  repo: https://github.com/wangshenguiuc/OnClass
- name: Scanorama
  role: Builds the expression-based comparison embedding by integrating the 26 datasets
    (run_scanorama_multiply_datasets, scan_dim = 10) for silhouette coefficient benchmarking
  repo: https://github.com/brianhie/scanorama
- name: tensorflow 2.0
  role: Neural network backend required to install and run OnClass (pip install tensorflow==2.0)
- name: Python 3.6
  role: Runtime environment specified by the OnClass installation instructions
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:85c5baf759bce9ad57e716aeb8e07ca417d1ed25e9ab706d4a40ce7097c62d9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - Python 3.6
  - Scanorama
  - tensorflow 2.0
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# cell-ontology-score-prediction

## Summary

Runs OnClass pretrained models over scRNA-seq datasets to produce a cell-by-all-Cell-Ontology-type score matrix (pred_Y_all) whose vectors can be reduced by PCA and used as a cell embedding, enabling clustering and cross-dataset integration by cell type rather than platform.

## When to use

Use when single-cell RNA-seq cells from one or many datasets must be scored against the full Cell Ontology with a pretrained OnClass model, so the score matrix can serve as an embedding for annotation, clustering, or integration benchmarks against expression-based methods like Scanorama.

## When NOT to use

- The cell types of interest are outside the Cell Ontology or are topologically identical siblings with the same parents in the Cell Ontology graph, which OnClass cannot distinguish
- The goal is batch correction of expression data in the presence of many unseen cell types — no such methods are developed here, and Scanorama on expression is the baseline being compared against
- Input is raw reads or gene expression already organized as an integrated embedding, not scRNA-seq expression matrices to be scored against the ontology

## Inputs

- Annotated scRNA-seq datasets downloaded from figshare (expression matrices with cell type labels)
- Pretrained OnClass model files
- Cell Ontology files: cl.ontology, cell type network, and cl.ontology.nlp.emb (NLP embeddings)

## Outputs

- pred_Y_all.released.npy — per-dataset cell-by-all-Cell-Ontology-type score matrix from OnClassModel.Predict
- PCA-reduced low-dimensional cell embedding of the OnClass score vectors
- silhouette_scores.csv — per-dataset silhouette coefficients for OnClass vs Scanorama embeddings plus the significance P value

## How to apply

Download the pretrained OnClass models and the annotated scRNA-seq datasets from figshare, and load the Cell Ontology files (cl.ontology, the cell type network, and cl.ontology.nlp.emb) with read_ontology_file. For each dataset, call OnClassModel.Predict with use_normalize=True to obtain pred_Y_all, the cell-by-all-Cell-Ontology-type score matrix, saving each result as pred_Y_all.released.npy. Reduce the pred_Y_all score vectors to a low-dimensional cell embedding via PCA; the ontology graph is what lets OnClass extend scores to unseen cell types with no annotated cells in training. For a comparison embedding, integrate the same datasets on expression with Scanorama (e.g., run_scanorama_multiply_datasets with scan_dim=10). Judge correctness by computing the silhouette coefficient of the OnClass-score PCA embedding versus the Scanorama embedding per dataset — OnClass-based integration should significantly improve over expression-based integration (reported P-value < 7e-293, two-sided test), with cells clustered by cell type rather than platform artifacts.

## Related tools

- **OnClass** (Produces the cell-by-Cell-Ontology-type score matrix (pred_Y_all) via OnClassModel.Predict with use_normalize=True, using pretrained models and Cell Ontology files) — https://github.com/wangshenguiuc/OnClass
- **Scanorama** (Builds the expression-based comparison embedding by integrating the 26 datasets (run_scanorama_multiply_datasets, scan_dim = 10) for silhouette coefficient benchmarking) — https://github.com/brianhie/scanorama
- **tensorflow 2.0** (Neural network backend required to install and run OnClass (pip install tensorflow==2.0))
- **Python 3.6** (Runtime environment specified by the OnClass installation instructions)

## Examples

```
pred_Y_all = model.Predict(test_feature, test_genes, use_normalize=True); np.save('pred_Y_all.released.npy', pred_Y_all)
```

## Evaluation signals

- Each dataset yields a saved score matrix named pred_Y_all.released.npy, produced with use_normalize=True
- OnClass run consumes the three ontology inputs (cl.ontology, cell type network, cl.ontology.nlp.emb) loaded via read_ontology_file
- Silhouette coefficient of the OnClass score-PCA embedding significantly exceeds the Scanorama expression embedding (reported P < 7e-293, two-sided test), recorded per dataset in silhouette_scores.csv
- Embedded cells cluster by cell type labels rather than by platform/dataset artifacts

## Limitations

- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states
- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- Methods to correct batch effects in the presence of many unseen cell types are not yet developed

## Evidence

- [discussion] OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph.: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [discussion] as part of the package we provided a pre-trained model that can output cell type annotations for millions of cells in a few minutes: "as part of the package we provided a pre-trained model that can output cell type annotations for millions of cells in a few minutes on a modern server."
- [methods] we use the hierarchical ontology structure from the Cell Ontology: "we use the hierarchical ontology structure from the `Cell Ontology <http://www.obofoundry.org/ontology/cl.html>`__"
- [methods] Batch correction using Scanorama: "Batch correction using Scanorama::"
- [readme] Scanorama enables batch-correction and integration of heterogeneous scRNA-seq datasets: [evidence span withheld — anchor status: external_doc]
