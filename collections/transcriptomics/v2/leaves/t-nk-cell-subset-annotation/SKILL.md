---
name: t-nk-cell-subset-annotation
description: Use when comparing batch-corrected vs. raw single-cell datasets cell-type-by-cell-type
  and a defined subset (e.g., T/NK cells from liver cancer samples HCC03N and HCC04N)
  must be consistently identified across each corrected .h5ad before running per-subset
  statistics.
when_to_use_negative:
- When the goal is whole-dataset integration quality assessment (e.g., over-correction
  scores on UMAP embeddings) rather than subset-specific statistics
- When reference and query datasets originate from different tissues or lack similar
  cell types, since scCobra label transfer requires tissue/cell-type compatibility
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scanpy
  role: Preprocessing, .h5ad I/O, neighbor computation, and Leiden clustering used
    to resolve and extract the T/NK cell subset from each matrix
- name: scCobra
  role: Online label transfer — a pre-trained KNN classifier annotates the query dataset
    so cell identities (including T/NK cells) can be assigned across batch-affected
    matrices without retraining
  repo: https://github.com/mcgilldinglab/scCobra
- name: scIB
  role: Runs the Leiden clustering resolution sweep (0.1–2.0 in increments of 0.1)
    used to identify cell populations in the integrated data
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:34b9b4aac7dc43b20eb96f37c1317d6ebae6fb572265cc153a43437f2e832a17
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - Harmony
  - Scanorama
  - Seurat
  - scCobra
  - scDML
  - scDREAMER
  - scIB
  - scVI
  - scanpy
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# t-nk-cell-subset-annotation

## Summary

Annotates and extracts the T/NK cell population from raw and batch-corrected single-cell matrices (.h5ad) so that downstream comparisons (e.g., differential expression between two healthy liver cancer samples) are performed on a matched cell subset. In the scCobra workflow this subset annotation is the prerequisite for the 470→30 DEG reduction benchmark.

## When to use

Use when comparing batch-corrected vs. raw single-cell datasets cell-type-by-cell-type and a defined subset (e.g., T/NK cells from liver cancer samples HCC03N and HCC04N) must be consistently identified across each corrected .h5ad before running per-subset statistics.

## When NOT to use

- When the goal is whole-dataset integration quality assessment (e.g., over-correction scores on UMAP embeddings) rather than subset-specific statistics
- When reference and query datasets originate from different tissues or lack similar cell types, since scCobra label transfer requires tissue/cell-type compatibility

## Inputs

- Raw preprocessed AnnData (.h5ad) from the liver cancer dataset (e.g., GSE149614 samples HCC03N and HCC04N)
- Batch-corrected expression matrices (.h5ad) from scCobra, Harmony, Scanorama, and/or scVI
- Reference labels or reference dataset of matching tissue/cell types for KNN label transfer (optional)

## Outputs

- Annotated T/NK cell subset extracted from each raw and corrected matrix
- Per-matrix subset annotations usable as input to differential expression testing
- DEG counts per condition (raw, scCobra, Harmony, Scanorama, scVI) for the T/NK subset, e.g., deg_counts_S3.csv

## How to apply

After preprocessing (filter cells <200 genes, genes <3 cells, mitochondrial fraction >5%, log1p normalization) and batch correction, load each corrected expression matrix as .h5ad and identify the T/NK cell population consistently in the raw data and in every corrected output. Cluster cells with Leiden clustering swept across resolutions 0.1–2.0 (increments of 0.1) as done in the scIB pipeline to pick a resolution that resolves the T/NK compartment, and optionally confirm identities via the scCobra online label-transfer route, where a pre-trained KNN classifier trained on a reference annotates the query dataset. Extract the annotated T/NK subset from each raw and corrected matrix so that the subsequent differential expression run between HCC03N and HCC04N (log2FC > 0.6, adjusted P < 0.05) is computed on equivalent cells. Rationale: DEG counts are only comparable across correction methods if the same biological subset is compared, since batch noise and over-correction can distort cell-type composition.

## Related tools

- **scanpy** (Preprocessing, .h5ad I/O, neighbor computation, and Leiden clustering used to resolve and extract the T/NK cell subset from each matrix)
- **scCobra** (Online label transfer — a pre-trained KNN classifier annotates the query dataset so cell identities (including T/NK cells) can be assigned across batch-affected matrices without retraining) — https://github.com/mcgilldinglab/scCobra
- **scIB** (Runs the Leiden clustering resolution sweep (0.1–2.0 in increments of 0.1) used to identify cell populations in the integrated data)

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git && jupyter nbconvert --to notebook --execute run_integration.ipynb
```

## Evaluation signals

- The T/NK population is identified in both raw and every batch-corrected .h5ad matrix with consistent composition, so DEG counts between HCC03N and HCC04N are subset-comparable
- Leiden clustering over the 0.1–2.0 resolution sweep yields a stable T/NK partition rather than a resolution artifact
- Downstream DEG calls (log2FC > 0.6, adjusted P < 0.05) reproduce the reported pattern: raw = 470 DEGs, scCobra = 30, Harmony = 40, scVI = 66
- Annotated subset sizes are plausible relative to the source dataset (e.g., immune/liver cancer data with ~33k cells across 10 batches)

## Limitations

- scCobra label transfer requires reference and query datasets to come from the same tissue type or contain similar cell types; cross-tissue transfer is out of scope
- Batch correction methods run with default parameters may blur cell-type boundaries, affecting subset extraction fidelity
- Mis-annotating or inconsistently extracting the T/NK subset invalidates the downstream DEG comparison between correction methods

## Evidence

- [discussion] The scIB pipeline applied Leiden clustering across a range of resolutions from 0.1 to 2.0: "The scIB pipeline applied Leiden clustering across a range of resolutions from 0.1 to 2.0 (in increments of 0.1)"
- [discussion] KNN classifier annotates query: [evidence span withheld — anchor status: not_found]
- [discussion] Label transfer tissue constraint: "the label transfer performed by scCobra requires that the reference and query datasets originate from the same tissue type or contain similar cell types"
- [readme] README integration workflow entry point: "## Run

Please refer run_integration.ipynb"
