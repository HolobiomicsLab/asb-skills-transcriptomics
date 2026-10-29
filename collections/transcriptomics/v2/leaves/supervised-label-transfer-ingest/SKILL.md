---
name: supervised-label-transfer-ingest
description: Use when an unannotated query single-cell dataset has a closely related
  annotated reference (e.g. HuBMAP spleen data) and reference-based label transfer
  is preferred over de novo clustering.
when_to_use_negative:
- No closely related annotated reference dataset exists — unconstrained Leiden clustering
  is the appropriate baseline instead.
- Reference and query share few overlapping genes (e.g. cross-platform or cross-species
  data where the gene-overlap step leaves too little signal).
- The goal is de novo discovery of novel cell states rather than mapping known reference
  annotations.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scanpy Ingest
  role: Performs the reference-based label transfer by projecting the query into the
    reference PCA space and mapping labels
- name: scanpy
  role: Preprocessing and PCA/neighbor machinery supporting the Ingest projection
- name: AnnData
  role: In-memory/mapped data object format holding reference and query matrices and
    label columns
- name: Cellar
  role: Interactive environment in which the label transfer and downstream semi-supervised
    refinement are orchestrated
  repo: https://github.com/euxhenh/cellar
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:9672249fecc6f905d4c944257651d7cf962c08233c10115b6bb281fcf34b299f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/euxhenh/cellar
  tools:
  - AnnData
  - Cellar
  - scanpy
  - scanpy Ingest
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# supervised-label-transfer-ingest

## Summary

Supervised cell-type label transfer in which a query scRNA-seq dataset (e.g. HBMP2-spleen-2) is projected into a reference dataset's PCA latent space (40 components, fit on HBMP3-spleen-CC3) using scanpy Ingest, transferring reference annotations to the query. Serves as the baseline labeling that Cellar's semi-supervised (frozen-cluster) Leiden refinement is designed to improve upon.

## When to use

Use when an unannotated query single-cell dataset has a closely related annotated reference (e.g. HuBMAP spleen data) and reference-based label transfer is preferred over de novo clustering.

## When NOT to use

- No closely related annotated reference dataset exists — unconstrained Leiden clustering is the appropriate baseline instead.
- Reference and query share few overlapping genes (e.g. cross-platform or cross-species data where the gene-overlap step leaves too little signal).
- The goal is de novo discovery of novel cell states rather than mapping known reference annotations.

## Inputs

- Annotated reference AnnData object (HBMP3-spleen-CC3)
- Unannotated query AnnData object (HBMP2-spleen-2)
- Gene-overlap intersection between reference and query datasets
- Reference label column (e.g. leiden/cell-type annotations in adata.obs)

## Outputs

- Per-cell transferred labels for the query dataset (query.obs label column)
- Query embeddings in the reference-fit 40-component PCA space
- ARI score of the transferred labels against expert ground truth (0.39 in the source study)

## How to apply

Load the reference (HBMP3-spleen-CC3) and query (HBMP2-spleen-2) datasets as AnnData objects, keeping only genes overlapping between the two. Fit a 40-component PCA latent space on the reference (scanpy preprocessing with scikit-learn's randomized-SVD PCA under the hood) and use scanpy Ingest to project the query into that space and map reference labels. This transferred labeling is one of three baselines; in the source study it scored ARI 0.39 against expert annotations, beating unconstrained Leiden at resolution 1 (ARI 0.27) but losing to semi-supervised Leiden with least-noisy clusters 0, 4, 9, and 10 frozen (ARI 0.66). The transferred assignments should therefore be treated as a starting point that can be refined by freezing the least-noisy clusters during subsequent semi-supervised Leiden iterations.

## Related tools

- **scanpy Ingest** (Performs the reference-based label transfer by projecting the query into the reference PCA space and mapping labels)
- **scanpy** (Preprocessing and PCA/neighbor machinery supporting the Ingest projection)
- **AnnData** (In-memory/mapped data object format holding reference and query matrices and label columns)
- **Cellar** (Interactive environment in which the label transfer and downstream semi-supervised refinement are orchestrated) — https://github.com/euxhenh/cellar

## Examples

```
ing = sc.tl.Ingest(adata_query); ing.fit(adata_ref); adata_query.obs['labels'] = ing.map('leiden')
```

## Evaluation signals

- ARI of the transferred labels against expert ground-truth annotations should approximate the reported 0.39.
- Expected ordering in the comparison table: frozen-cluster Leiden 0.66 > Ingest 0.39 > unconstrained Leiden 0.27.
- Transferred labels must cover only genes overlapping between reference and query; check the joint gene set after intersection.
- The three ARI values should be assembled and saved to ari_comparison.csv reproducing Supplementary Fig. 2 ordering.
- Query cells should be embedded in the reference-fit 40-component PCA space, not a query-fit space.

## Limitations

- No software versions stated for Cellar, Python, or scanpy, complicating exact reproduction.
- Reproducibility may depend on the external Cellar web server (https://cellar.cmu.hubmapconsortium.org/app/cellar) rather than a locally installable archived artifact.
- No accession IDs are given for the HuBMAP annotated reference datasets, so obtaining the exact reference requires external lookup.
- Label transfer alone underperforms semi-supervised refinement; using it without cluster freezing leaves accuracy on the table (0.39 vs 0.66).

## Evidence

- [methods] Label transfer between HBMP2-spleen-2 and HBMP3-spleen-CC3 was performed using scanpy’s Ingest: "Label transfer between HBMP2-spleen-2 and HBMP3-spleen-CC3 was performed using scanpy’s Ingest"
- [methods] Preprocessing via scanpy and scikit-learn PCA: "We used the PCA implementation of the scikit-learn package with a randomized SVD solver."
- [abstract] Cellar open-source annotated HuBMAP references: "Cellar is open-source and includes several annotated HuBMAP datasets."
- [intro] Semi-supervised clustering capability in Cellar: "Cellar provides methods for semi-supervised clustering and projection of expression clusters in spatial single-cell images"
