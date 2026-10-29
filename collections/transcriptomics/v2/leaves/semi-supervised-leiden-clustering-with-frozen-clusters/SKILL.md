---
name: semi-supervised-leiden-clustering-with-frozen-clusters
description: Use when refining label-transfer annotations of a query scRNA-seq dataset
  (e.g., HBMP2-spleen-2 against an annotated HBMP3-spleen-CC3 reference) where unconstrained
  Leiden clustering alone disagrees with reference labels and a semi-automatic, expert-guided
  correction is acceptable.
when_to_use_negative:
- The query dataset has no annotated reference to transfer labels from, so no 'least
  noisy' constrained clusters can be identified from label-transfer results
- The analysis goal is fully automatic annotation without any expert intervention
  to select or freeze clusters
- The query cells lack ground-truth annotations, making ARI-based validation against
  expert labels impossible
edam_operation: http://edamontology.org/operation_0362
tools:
- name: Cellar
  role: Interactive web server providing the semi-supervised Leiden implementation
    in which least-noisy clusters are frozen as constraints during iterative clustering
  repo: https://github.com/euxhenh/cellar
- name: faiss
  role: Approximate nearest-neighbor graph construction (15 neighbors) to speed up
    neighbors graph building for Leiden clustering
  repo: https://github.com/facebookresearch/faiss
- name: scanpy
  role: Preprocessing of the data and label transfer via scanpy Ingest, whose output
    is refined by the semi-supervised Leiden step
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:56e025611b3184366b4587e17e9dbb1b3a3a738bc4a831bebbe591b7200cf57e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/euxhenh/cellar
  tools:
  - AnnData
  - Cellar
  - faiss
  - scanpy
  edam_operation: http://edamontology.org/operation_0362
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# semi-supervised Leiden clustering with frozen clusters

## Summary

A semi-supervised adaptation of Leiden clustering, available in Cellar, that freezes the least noisy clusters (selected from label-transfer results) as constraints during iterative re-clustering, improving cell-type annotation over unconstrained Leiden or label transfer alone.

## When to use

Use when refining label-transfer annotations of a query scRNA-seq dataset (e.g., HBMP2-spleen-2 against an annotated HBMP3-spleen-CC3 reference) where unconstrained Leiden clustering alone disagrees with reference labels and a semi-automatic, expert-guided correction is acceptable.

## When NOT to use

- The query dataset has no annotated reference to transfer labels from, so no 'least noisy' constrained clusters can be identified from label-transfer results
- The analysis goal is fully automatic annotation without any expert intervention to select or freeze clusters
- The query cells lack ground-truth annotations, making ARI-based validation against expert labels impossible

## Inputs

- Query AnnData object (e.g., HBMP2-spleen-2) with overlapping genes retained relative to the reference
- Annotated reference AnnData object (e.g., HBMP3-spleen-CC3) with human-expert cell-type labels
- Ground-truth expert annotations for the query cells used for ARI evaluation
- Preprocessed expression matrix (cells filtered to 50–3000 expressed genes, genes to 50–3000 cells, CPM 1e5 normalization, log1p transform, unit-variance/zero-mean scaling)

## Outputs

- Frozen-cluster Leiden cluster assignments for every query cell at resolution 1
- Baseline labelings for comparison (scanpy Ingest transferred labels and unconstrained Leiden clusters)
- ari_comparison.csv table of ARI scores for the three labelings (expected: frozen-cluster Leiden 0.66, Ingest 0.39, unconstrained Leiden 0.27)

## How to apply

First project the query AnnData object into a 40-component PCA latent space fit on the reference dataset (HBMP3-spleen-CC3) using scanpy Ingest to obtain transferred labels. Build an approximate neighbors graph with faiss (15 neighbors) to speed up Leiden clustering, then run unconstrained Leiden at resolution 1 as a baseline labeling. Run semi-supervised Leiden at resolution 1 in Cellar, choosing the least noisy clusters as constraints — on HBMP2-spleen-2 these were clusters 0, 4, 9, and 10 — and freezing them during iterative re-clustering so they are never split or reassigned. Finally, compute the adjusted Rand index (ARI) of all three labelings (Ingest, unconstrained Leiden, frozen-cluster Leiden) against human-expert ground-truth annotations and assemble them into a comparison table (ari_comparison.csv).

## Related tools

- **Cellar** (Interactive web server providing the semi-supervised Leiden implementation in which least-noisy clusters are frozen as constraints during iterative clustering) — https://github.com/euxhenh/cellar
- **faiss** (Approximate nearest-neighbor graph construction (15 neighbors) to speed up neighbors graph building for Leiden clustering) — https://github.com/facebookresearch/faiss
- **scanpy** (Preprocessing of the data and label transfer via scanpy Ingest, whose output is refined by the semi-supervised Leiden step)

## Examples

```
import scanpy as sc; ing = sc.tl.Ingest(ref_adata); ing.fit(query_adata); query_adata.obs['celltype'] = ing.map('celltype')
```

## Evaluation signals

- ARI ordering reproduces Supplementary Fig. 2: frozen-cluster Leiden 0.66 > Ingest 0.39 > unconstrained Leiden 0.27
- The frozen clusters (0, 4, 9, 10 on HBMP2-spleen-2) remain completely unchanged across all clustering iterations
- All three labelings are computed on the same query cells over the same 40-component PCA latent space, so ARI values are directly comparable
- The neighbors graph is built with faiss using 15 neighbors before Leiden runs, matching the methods description
- The three ARI scores are assembled into a comparison table saved as ari_comparison.csv

## Limitations

- No software versions stated for Cellar, Python, Dash, or faiss; reproducibility depends on the external web server (https://cellar.cmu.hubmapconsortium.org/app/cellar) rather than a locally installable archived artifact
- Selection of which clusters to freeze ('least noisy') is a manual, expert-driven decision demonstrated only on the HBMP2-spleen-2 / HBMP3-spleen-CC3 pair
- Validation relies on ARI against human-expert annotations, which are themselves subjective cluster labels

## Evidence

- [intro] Cellar provides methods for semi-supervised clustering and projection of expression clusters in spatial single-cell images: "Cellar provides methods for semi-supervised clustering and projection of expression clusters in spatial single-cell images"
- [methods] The embeddings were then used to construct an approximate neighbors graph using faiss with 15 neighbors: [evidence span withheld — anchor status: not_found]
- [methods] Label transfer between HBMP2-spleen-2 and HBMP3-spleen-CC3 was performed using scanpy’s Ingest: "Label transfer between HBMP2-spleen-2 and HBMP3-spleen-CC3 was performed using scanpy’s Ingest"
- [results] a semi-supervised adaptation of Leiden, where the least noisy clusters were chosen as constraints and frozen during iterations, led to a much better ARI score of 0.66: [evidence span withheld — anchor status: not_found]
