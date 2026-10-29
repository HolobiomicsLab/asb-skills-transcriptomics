---
name: scanorama-expression-integration
description: Use when a multi-dataset scRNA-seq collection must be integrated purely
  from expression profiles as a batch-correction/integration baseline — e.g., to test
  whether annotation-score embeddings cluster cells by cell type better than expression-level
  integration.
when_to_use_negative:
- The goal is annotation-based integration rather than expression-based integration
  — the OnClass paper shows ontology-score PCA embeddings cluster cells by type better
  than Scanorama (P < 7e-293), so Scanorama is the wrong primary method when label-quality
  clustering is the objective
- Data has not undergone noise reduction — Scanorama is designed to run downstream
  of imputation/highly-variable gene filtering, not as a first processing step
- Only a single dataset is present — Scanorama's mutual-NN matching requires multiple
  datasets/batches to integrate
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: Scanorama
  role: Performs the expression-based batch correction and integration of the 26 heterogeneous
    datasets (run_scanorama_multiply_datasets, scan_dim = 10) to produce the baseline
    embedding whose silhouette coefficient is compared to the OnClass score embedding
  repo: https://github.com/brianhie/scanorama
- name: OnClass
  role: Produces the comparator cell-type-score embedding (pred_Y_all PCA) whose silhouette
    significantly exceeds the Scanorama expression embedding, defining the significance
    test this integration baseline feeds into
  repo: https://github.com/wangshenguiuc/OnClass
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-25725-x
    title: Leveraging the Cell Ontology to classify unseen cell types
  source_content_hash: sha256:ce5751cb55e594722d16fa498ff8a7022c18e061da779a6c1e0eab2f66c64a33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/brianhie/scanorama
  tools:
  - OnClass
  - Python
  - Scanorama
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# scanorama-expression-integration

## Summary

Integrates heterogeneous scRNA-seq datasets based on expression profiles using Scanorama, producing a low-dimensional integrated embedding (here with scan_dim = 10) that serves as the expression-based baseline against which Cell Ontology annotation-based integration (OnClass score embeddings) is compared via silhouette coefficients.

## When to use

Use when a multi-dataset scRNA-seq collection must be integrated purely from expression profiles as a batch-correction/integration baseline — e.g., to test whether annotation-score embeddings cluster cells by cell type better than expression-level integration.

## When NOT to use

- The goal is annotation-based integration rather than expression-based integration — the OnClass paper shows ontology-score PCA embeddings cluster cells by type better than Scanorama (P < 7e-293), so Scanorama is the wrong primary method when label-quality clustering is the objective
- Data has not undergone noise reduction — Scanorama is designed to run downstream of imputation/highly-variable gene filtering, not as a first processing step
- Only a single dataset is present — Scanorama's mutual-NN matching requires multiple datasets/batches to integrate

## Inputs

- List of scRNA-seq expression matrices (cells-by-genes; scipy.sparse.csr_matrix or numpy.ndarray), one per dataset
- List of per-dataset gene name lists (genes_list) aligned to each matrix
- Cell type label annotations per cell (for silhouette-based evaluation)
- Optionally scanpy AnnData objects for the scanpy wrapper API

## Outputs

- Integrated low-dimensional cell embedding (scan_dim = 10 in the 26-dataset comparison), one embedding per dataset
- For correct()/correct_scanpy(): batch-corrected cells-by-gene matrices plus integrated embedding (return_dimred=True)
- Per-dataset silhouette coefficient values of the Scanorama embedding with respect to cell type labels (e.g., written to silhouette_scores.csv alongside the comparator embedding)

## How to apply

Install Scanorama (pip install scanorama; under anaconda first install python-annoy) and load each dataset as a cells-by-genes matrix with its gene list. Integrate all datasets jointly — in the OnClass 26-dataset comparison this is done via run_scanorama_multiply_datasets with scan_dim = 10 — using scanorama.integrate/correct (with return_dimred=True for a joint embedding), or the Scanpy wrappers integrate_scanpy/correct_scanpy which store the embedding in adata.obsm['X_scanorama']. Scanorama performs mutual-NN matching between datasets and returns a low-dimensional embedding intended to align shared cell populations across platforms. To evaluate the integration, compute the silhouette coefficient of the Scanorama embedding using cell type labels for each dataset, and compare against the alternative (OnClass pred_Y_all PCA) embedding; in the source study the OnClass embedding significantly outperformed the Scanorama expression embedding (P < 7e-293, two-sided test), so Scanorama serves as the expression-based control. Reference data (~720 MB, 26 heterogeneous datasets) is available at http://scanorama.csail.mit.edu/data_light.tar.gz.

## Related tools

- **Scanorama** (Performs the expression-based batch correction and integration of the 26 heterogeneous datasets (run_scanorama_multiply_datasets, scan_dim = 10) to produce the baseline embedding whose silhouette coefficient is compared to the OnClass score embedding) — https://github.com/brianhie/scanorama
- **OnClass** (Produces the comparator cell-type-score embedding (pred_Y_all PCA) whose silhouette significantly exceeds the Scanorama expression embedding, defining the significance test this integration baseline feeds into) — https://github.com/wangshenguiuc/OnClass

## Examples

```
import scanorama; integrated, genes = scanorama.integrate(datasets, genes_list)
```

## Evaluation signals

- Integration completes over all datasets and returns embeddings of the requested dimensionality (scan_dim = 10) with one embedding per input dataset
- Silhouette coefficients computed on the Scanorama embedding with cell type labels are quantified per dataset and recorded (e.g., in silhouette_scores.csv) alongside the comparator embedding's values
- In the source reproduction, the Scanorama expression embedding's silhouette is significantly lower than the OnClass score-PCA embedding's silhouette (OnClass improvement P < 7e-293, two-sided test), i.e., Scanorama cells cluster by platform/dataset artifacts rather than cell type
- The same datasets, gene lists, and cell label set are used for both embeddings so the silhouette comparison is paired and a two-sided significance test is valid

## Limitations

- Scanorama results are intended as input to downstream clustering/visualization and should follow noise-reduction steps (imputation, highly-variable gene filtering); feeding raw counts degrades matching
- In the OnClass paper, methods to correct batch effects in the presence of many unseen cell types are not yet developed, and expression-based integration can fail to remove platform artifacts relative to ontology-score-based clustering
- From R via reticulate, sparse matrix returns are problematic — set return_dense=TRUE, which increases memory usage

## Evidence

- [readme] Scanorama enables batch-correction and integration of heterogeneous scRNA-seq datasets: [evidence span withheld — anchor status: external_doc]
- [readme] integrate API usage: "# Integration.
integrated, genes = scanorama.integrate(datasets, genes_list)"
- [readme] Scanorama designed for downstream of noise reduction: [evidence span withheld — anchor status: external_doc]
- [readme] 26-dataset light data download: [evidence span withheld — anchor status: external_doc]
- [methods] Scanorama in OnClass methods: "Batch correction using Scanorama::"
- [results] silhouette significance over Scanorama: [evidence span withheld — anchor status: not_found]
