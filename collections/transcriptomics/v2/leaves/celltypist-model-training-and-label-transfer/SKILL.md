---
name: celltypist-model-training-and-label-transfer
description: Use when cell-type labels must be transferred from a large labeled reference
  scRNA-seq dataset (.h5ad AnnData with a cell_type key and X_pca representation)
  to a query dataset, and annotation fidelity should be benchmarked against a full-reference
  transfer.
when_to_use_negative:
- The reference has no reliable cell_type labels or the reference and query cell types
  have no overlap (e.g. unrelated tissues), since label transfer requires an established,
  well-annotated reference and validating predictions against original annotations
- The goal is de novo cell-type discovery or clustering of unlabeled data rather than
  transferring labels from an existing atlas
- Input is already a confidence-scored, annotated query — the relevant step then is
  filtering cells below the user-defined confidence threshold, not re-training a model
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CellTypist
  role: Trains the reference model and transfers labels with confidence scores to
    the query dataset; agreement with the full-reference transfer is the evaluation
    metric
- name: Scanpy
  role: Loads .h5ad AnnData, selects highly variable genes, computes PCA into adata.obsm['X_pca'],
    and implements uniform subsampling of the reference
- name: scValue
  role: Produces the value-based reference sketch (use_rep='X_pca', cell_type_key='cell_type',
    type_weighting='inv_cv', strategy='FB', n_trees=100, seed=42) on which a CellTypist
    model is trained
  repo: https://github.com/LHBCB/scvalue
- name: scikit-learn
  role: Underpins the random-forest OOB data-value estimation that ranks cells before
    sketch-based model training (paper used scikit-learn 1.5.2)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbaf279
    title: 'scValue: value-based subsampling of large-scale single-cell transcriptomic
      data for machine and deep learning tasks'
  source_content_hash: sha256:b88cc1767219808dccf51ee5c05c87dff0ff1a722503b7af07564dc8d7ee0b49
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/LHBCB/scvalue
  repo_url: https://github.com/LHBCB/scvalue
  tools:
  - CellTypist
  - GeoSketch
  - Hopper
  - Scanpy
  - Seurat
  - Sphetcher
  - scSampler
  - scValue
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# celltypist-model-training-and-label-transfer

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains CellTypist models on labeled scRNA-seq references (e.g. the 428k-cell human gut atlas stored as .h5ad AnnData with a cell_type obs column) and transfers labels to query datasets, quantifying per-population agreement against a full-reference transfer. In the scValue paper this is the annotation task used to judge whether a sketch (scValue, Uniform, GeoSketch, Sphetcher, Hopper, scSampler) preserves label-transfer fidelity.

## When to use

Use when cell-type labels must be transferred from a large labeled reference scRNA-seq dataset (.h5ad AnnData with a cell_type key and X_pca representation) to a query dataset, and annotation fidelity should be benchmarked against a full-reference transfer.

## When NOT to use

- The reference has no reliable cell_type labels or the reference and query cell types have no overlap (e.g. unrelated tissues), since label transfer requires an established, well-annotated reference and validating predictions against original annotations
- The goal is de novo cell-type discovery or clustering of unlabeled data rather than transferring labels from an existing atlas
- Input is already a confidence-scored, annotated query — the relevant step then is filtering cells below the user-defined confidence threshold, not re-training a model

## Inputs

- Reference scRNA-seq dataset as .h5ad AnnData (log-normalized, with adata.obs cell-type labels and adata.obsm['X_pca'], e.g. the 428k-cell human gut atlas)
- Optional: subsampled AnnData sketches (e.g. 10% via SCValue with strategy='FB', type_weighting='inv_cv', n_trees=100, seed=42)
- Query scRNA-seq dataset with unlabeled populations (e.g. immune T-cell populations)

## Outputs

- Trained CellTypist models (one per sketch and one for the full 428k-cell reference)
- Transferred cell-type labels with per-cell confidence scores for the query dataset
- Per-population agreement scores versus the full-reference transfer (nine T-cell populations in the Gut-to-immune case)
- Dot plot figure4_label_transfer_agreement.png reproducing Figure 4

## How to apply

First load the reference (e.g. the 428k-cell human gut scRNA-seq atlas) with Scanpy, select highly variable genes, and compute a PCA stored in adata.obsm['X_pca'] on the log-normalized matrix (paper used top 3,000 HVGs). If benchmarking subsampling, build a balanced sketch with SCValue (sketch_size as an integer cell count or float fraction, use_rep='X_pca', cell_type_key='cell_type', type_weighting='inv_cv', strategy='FB', n_trees=100, seed=42) and train one CellTypist model per sketch plus one on the full reference as the gold-standard baseline. Transfer labels from each model to the query (immune T-cell populations), each cell receiving a confidence score from the established CellTypist model. Validate predicted cell types against the original annotations and compute per-population agreement with the full-reference transfer — for the Gut-to-immune case study this is done per each of the nine T-cell populations. Render agreement patterns as a dot plot (Figure 4, figure4_label_transfer_agreement.png), where scValue and Uniform should sit closest to the full-reference transfer.

## Related tools

- **CellTypist** (Trains the reference model and transfers labels with confidence scores to the query dataset; agreement with the full-reference transfer is the evaluation metric)
- **Scanpy** (Loads .h5ad AnnData, selects highly variable genes, computes PCA into adata.obsm['X_pca'], and implements uniform subsampling of the reference)
- **scValue** (Produces the value-based reference sketch (use_rep='X_pca', cell_type_key='cell_type', type_weighting='inv_cv', strategy='FB', n_trees=100, seed=42) on which a CellTypist model is trained) — https://github.com/LHBCB/scvalue
- **scikit-learn** (Underpins the random-forest OOB data-value estimation that ranks cells before sketch-based model training (paper used scikit-learn 1.5.2))

## Examples

```
import scanpy as sc; from scvalue import SCValue; sc.pp.highly_variable_genes(adata, n_top_genes=3000); sc.pp.pca(adata, n_comps=50); scv = SCValue(adata=adata, sketch_size=0.1, use_rep='X_pca', cell_type_key='cell_type', n_trees=100, strategy='FB', seed=42); adata_sub = scv.value()
```

## Evaluation signals

- The dot plot must show scValue (together with Uniform) yielding agreement closest to the full-reference transfer across the nine T-cell populations, consistent with the paper's Figure 4
- A CellTypist model trained on the full 428k-cell Gut reference is generated first and serves as the gold-standard transfer for all agreement comparisons
- Per-population agreement is computed for each of the nine T-cell populations individually, not only as a global accuracy
- Every transferred cell carries a confidence score from the established CellTypist model, and predicted types are validated against the original annotation
- SCValue run parameters are checkable: DV stored in adata.obs['dv'], sketch returned as scv.adata_sub, and dv_values.csv/dv_summary.csv if write_dv=True

## Limitations

- Tool versions for benchmarked software such as CellTypist are not specified in the study, complicating exact reproduction (README lists Scanpy 1.10.2 and AnnData 0.10.8 for the paper's experiments)
- The scValue confidence-score threshold for filtering is user-defined with no default value given
- With sparse matrices and use_rep='X', scValue may densify internally (memory cost); scValue expects features from adata.obsm['X_pca'] by default
- Uniform subsampling, though the simplest baseline, may not capture the full transcriptional diversity of a dataset — agreement close to the full reference does not guarantee rare-cell-type capture (Gini coefficient should also be assessed)

## Evidence
