---
name: cross-dataset-single-cell-embedding-comparison
description: Use when the goal is to test whether predicted cell-type annotation scores
  (cell-by-Cell-Ontology-type matrix from OnClass Predict) integrate multiple heterogeneous
  scRNA-seq datasets better than expression-space integration (Scanorama), quantified
  by silhouette coefficient.
when_to_use_negative:
- Input is a single dataset or already an integrated/embedded feature table — the
  comparison requires multiple raw expression datasets to run Scanorama integration
  against the OnClass score embedding
- The question is annotation accuracy of assigned labels rather than embedding/integration
  quality — use OnClass cross-dataset prediction benchmarking (silhouette of embeddings
  is not a label-accuracy metric)
- The Cell Ontology is known to miss the cell types/states of interest — the paper
  notes the ontology 'was not developed specifically for scRNA-seq and, therefore,
  likely misses cell types and cell states'
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: OnClass
  role: Predicts cell-type scores against the full Cell Ontology (OnClassModel.Predict,
    use_normalize=True); the pred_Y_all score matrix is PCA-embedded to form the ontology-based
    integration embedding
  repo: https://github.com/wangshenguiuc/OnClass
- name: Scanorama
  role: Provides the expression-based integration baseline; datasets are integrated
    with scan_dim = 10 via run_scanorama_multiply_datasets to produce the comparison
    embedding
  repo: https://github.com/brianhie/scanorama
- name: tensorflow 2.0
  role: Deep-learning backend for running the pretrained OnClass prediction models
    (pip install tensorflow==2.0, tensorflow-gpu==2.0, Python 3.6)
- name: Python
  role: Implementation environment for the OnClass pipeline, PCA reduction, silhouette
    computation, and significance testing
  repo: https://github.com/wangshenguiuc/OnClass
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:eff99dad914b3d7ec438364395c243cc0cabfccefe6474e850473e390b4644ed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wangshenguiuc/OnClass
  tools:
  - OnClass
  - Python
  - Scanorama
  - tensorflow 2.0
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-dataset-single-cell-embedding-comparison

## Summary

Compares ontology-based single-cell integration against expression-based batch correction by embedding each of 26 annotated scRNA-seq datasets in two ways (PCA of OnClass Cell-Ontology score vectors vs. Scanorama integration) and scoring both with per-dataset silhouette coefficients over shared cell-type labels.

## When to use

Use when the goal is to test whether predicted cell-type annotation scores (cell-by-Cell-Ontology-type matrix from OnClass Predict) integrate multiple heterogeneous scRNA-seq datasets better than expression-space integration (Scanorama), quantified by silhouette coefficient.

## When NOT to use

- Input is a single dataset or already an integrated/embedded feature table — the comparison requires multiple raw expression datasets to run Scanorama integration against the OnClass score embedding
- The question is annotation accuracy of assigned labels rather than embedding/integration quality — use OnClass cross-dataset prediction benchmarking (silhouette of embeddings is not a label-accuracy metric)
- The Cell Ontology is known to miss the cell types/states of interest — the paper notes the ontology 'was not developed specifically for scRNA-seq and, therefore, likely misses cell types and cell states'

## Inputs

- 26 annotated scRNA-seq datasets (figshare download used in the OnClass paper)
- Pretrained OnClass models and Cell Ontology files: cl.ontology, cell type network, cl.ontology.nlp.emb
- Cell-type labels per dataset for silhouette computation
- Raw expression matrices (cells-by-genes) for the Scanorama integration arm

## Outputs

- pred_Y_all.released.npy — per-dataset cell-by-all-Cell-Ontology-type score matrices from OnClassModel.Predict
- PCA-embedded OnClass score representation per dataset
- Scanorama-integrated expression embedding (scan_dim = 10) per dataset
- silhouette_scores.csv — per-dataset silhouette coefficients for both embeddings and the significance P value (expected P < 7e-293)

## How to apply

Download the pretrained OnClass models, the 26 annotated scRNA-seq datasets from figshare, and the Cell Ontology files (cl.ontology, cell type network, cl.ontology.nlp.emb) loaded via read_ontology_file. For each dataset run OnClassModel.Predict with use_normalize=True to obtain pred_Y_all, the cell-by-all-Cell-Ontology-type score matrix, saved as pred_Y_all.released.npy. Reduce each pred_Y_all score matrix to a low-dimensional cell embedding with PCA. Build the expression-based comparison embedding by integrating the datasets with Scanorama (scan_dim = 10, e.g. via run_scanorama_multiply_datasets). Compute the silhouette coefficient of each embedding per dataset using cell-type labels, then test the significance of the OnClass-over-Scanorama silhouette improvement (two-sided test) and write per-dataset values plus the P value to silhouette_scores.csv.

## Related tools

- **OnClass** (Predicts cell-type scores against the full Cell Ontology (OnClassModel.Predict, use_normalize=True); the pred_Y_all score matrix is PCA-embedded to form the ontology-based integration embedding) — https://github.com/wangshenguiuc/OnClass
- **Scanorama** (Provides the expression-based integration baseline; datasets are integrated with scan_dim = 10 via run_scanorama_multiply_datasets to produce the comparison embedding) — https://github.com/brianhie/scanorama
- **tensorflow 2.0** (Deep-learning backend for running the pretrained OnClass prediction models (pip install tensorflow==2.0, tensorflow-gpu==2.0, Python 3.6))
- **Python** (Implementation environment for the OnClass pipeline, PCA reduction, silhouette computation, and significance testing) — https://github.com/wangshenguiuc/OnClass

## Examples

```
integrated, corrected, genes = scanorama.correct(datasets, genes_list, return_dimred=True)
```

## Evaluation signals

- Per-dataset silhouette coefficient of the OnClass score-PCA embedding significantly exceeds that of the Scanorama expression embedding (two-sided test, P < 7e-293 reported)
- Each of the 26 datasets yields a saved pred_Y_all.released.npy score matrix from OnClassModel.Predict with use_normalize=True
- silhouette_scores.csv contains per-dataset silhouette values for both embeddings plus the P value
- In the OnClass embedding, cells cluster by cell type rather than by platform/dataset artifact (qualitative check in the embedding space)
- Scanorama integration uses the paper's parameter scan_dim = 10 for the baseline embedding

## Limitations

- The Cell Ontology was not developed specifically for scRNA-seq and likely misses cell types and cell states
- OnClass cannot distinguish topologically identical unseen cell types, such as siblings with the same parents in the Cell Ontology graph
- Methods to correct batch effects in the presence of many unseen cell types are not yet developed

## Evidence

- [other] Quantifying integration performance with the silhouette coefficient, OnClass-based integration of the 26 datasets significantly improved over expression-based integration using Scanorama: [evidence span withheld — anchor status: not_found]
- [other] Reduce pred_Y_all to PCA embedding and Scanorama baseline: [evidence span withheld — anchor status: not_found]
- [discussion] OnClass predicts scores extended to unseen Cell Ontology types: "OnClass will then perform the third step to extend these scores to unseen cell types based on the Cell Ontology graph."
- [readme] Scanorama API usage for integration/batch correction: [evidence span withheld — anchor status: external_doc]
- [discussion] Pretrained model availability: "as part of the package we provided a pre-trained model that can output cell type annotations for millions of cells in a few minutes on a modern server."
