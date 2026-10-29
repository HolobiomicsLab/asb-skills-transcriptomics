---
name: nearest-neighbor-cell-querying
description: Use when query scRNA-seq cells (h5ad) must be matched to annotated reference
  cells for label transfer or cell-type querying, and when similarity should account
  for posterior uncertainty rather than plain embedding distance.
when_to_use_negative:
- Reference datasets with fewer than ~1000 cells, where the neural network model may
  suffer under-training issues
- Queries against references lacking unified, comparable annotations — first curate
  the reference database
- Data still in legacy h5 format without conversion to h5ad (use the h5_to_h5ad migration
  step first)
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Cell BLAST
  role: 'Primary cell-querying tool: builds the neural network-based generative embedding
    and computes the customized cell-to-cell similarity metric'
  repo: https://github.com/gao-lab/Cell_BLAST
- name: DIRECTi
  role: Deep generative model trained on reference data (Cell_BLAST.directi.fit_DIRECTi)
    to produce the unbiased embedding used for nearest-neighbor search
  repo: https://github.com/gao-lab/Cell_BLAST
- name: scVI
  role: Canonical Gaussian-diagonal variational autoencoder baseline trained on the
    same reference data for ROC/AUC comparison of similarity metrics
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17281-7
    title: Searching large-scale scRNA-seq databases via unbiased cell embedding with
      Cell BLAST
  source_content_hash: sha256:97760b84461048c3c9b41ff28b86c1c99d733fdaecc35f75bfbb692be31b3961
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gao-lab/Cell_BLAST
  tools:
  - Cell BLAST
  - DIRECTi
  - scVI
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# nearest-neighbor-cell-querying

## Summary

Retrieves nearest reference cells for each query cell in a low-dimensional unbiased embedding (DIRECTi/scVI) and transfers reference annotations via a posterior-based cell-to-cell similarity metric (NPD), benchmarked against Euclidean distance using ROC/AUC.

## When to use

Use when query scRNA-seq cells (h5ad) must be matched to annotated reference cells for label transfer or cell-type querying, and when similarity should account for posterior uncertainty rather than plain embedding distance.

## When NOT to use

- Reference datasets with fewer than ~1000 cells, where the neural network model may suffer under-training issues
- Queries against references lacking unified, comparable annotations — first curate the reference database
- Data still in legacy h5 format without conversion to h5ad (use the h5_to_h5ad migration step first)

## Inputs

- Query scRNA-seq dataset in h5ad format
- Reference scRNA-seq dataset in h5ad format with annotations (e.g., cell type)
- Trained DIRECTi (or scVI) low-dimensional embedding of the reference data

## Outputs

- Nearest-neighbor query-reference cell pairs under the embedding
- Posterior-based NPD and Euclidean distances for each pair
- ROC curves and AUC values for cell-type querying accuracy
- Transferred annotations (e.g., Cell Ontology terms) for query cells

## How to apply

Load query and reference scRNA-seq datasets in h5ad format with Cell_BLAST.data, then train a DIRECTi model on the reference via Cell_BLAST.directi.fit_DIRECTi (RMSpropOptimizer per the migration guide) to obtain an unbiased low-dimensional embedding that handles batch effect. Retrieve nearest-neighbor pairs between query cells and reference cells under the embedding; for each pair compute both the posterior-based normalized projection distance (NPD) between posterior distributions and the Euclidean distance between embedding vectors. When benchmarking similarity metrics (e.g., against a scVI Gaussian-diagonal variational posterior baseline in a ten-dimensional embedding), build ROC curves and compute AUC for whether nearest reference cells share the query cell's type. Validate correctness by confirming the NPD-based ROC curve dominates (higher AUC) the Euclidean-distance ROC, consistent with the reported NPD superiority with the adversarial variational posterior (Supplementary Figs. 6g-h, 8c).

## Related tools

- **Cell BLAST** (Primary cell-querying tool: builds the neural network-based generative embedding and computes the customized cell-to-cell similarity metric) — https://github.com/gao-lab/Cell_BLAST
- **DIRECTi** (Deep generative model trained on reference data (Cell_BLAST.directi.fit_DIRECTi) to produce the unbiased embedding used for nearest-neighbor search) — https://github.com/gao-lab/Cell_BLAST
- **scVI** (Canonical Gaussian-diagonal variational autoencoder baseline trained on the same reference data for ROC/AUC comparison of similarity metrics)

## Examples

```
import Cell_BLAST as cb; cb.data.h5_to_h5ad(inputfilename, outputfilename); model = cb.directi.fit_DIRECTi(reference_data)
```

## Evaluation signals

- The NPD-based ROC curve dominates (has higher AUC than) the Euclidean-distance ROC curve for same-type nearest-neighbor discrimination
- Package imports successfully (import Cell_BLAST as cb; print(cb.__version__))
- Query and reference inputs load successfully via Cell_BLAST.data in h5ad format
- DIRECTi model trains with default RMSpropOptimizer on the reference data without error
- AUC improvement of NPD over Euclidean is confirmed with independent query data (e.g., ten-dimensional embedding)

## Limitations

- Integrating datasets at mega-scale may cause over-correcting of batches with highly skewed cell-type composition
- The Cell BLAST model is over-parameterized and requires a reasonable number of samples to train properly
- A cell number of 3000 is generally sufficient, but datasets with fewer than 1000 cells may suffer from under-training issues

## Evidence

- [abstract] Cell BLAST accuracy with variational posterior: "Herein, we present Cell BLAST, an accurate and robust cell-querying method built on a neural network-based generative model and a customized cell-to-cell similarity metric."
- [discussion] scVI posterior baseline: "the canonical variational autoencoder model used by scVI and several other tools11,25–27 enforces Gaussian distribution (with diagonal-covariance matrices) for the variational posterior."
- [discussion] Batch effect modeling rationale: "By explicitly modeling multilevel batch effect as well as uncertainty in cell-to-cell similarity estimation, Cell BLAST is an accurate and robust querying algorithm"
- [readme] Install via pip: "Install Cell BLAST from PyPI by running:

   `pip install Cell-BLAST`"
- [readme] Reproducibility branch: [evidence span withheld — anchor status: external_doc]
- [discussion] Under-training limitation: "a cell number of 3000 is generally sufficient for good performance, while small-sized datasets with fewer than 1000 cells may suffer from under-training issues."
