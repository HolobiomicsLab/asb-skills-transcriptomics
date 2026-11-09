---
name: adjusted-rand-index-clustering-evaluation
description: Use when multiple cell-type annotation labelings of the same cells (e.g.,
  scanpy Ingest transfer vs Leiden variants) must be scored against expert annotations
  to rank methods on a single dataset.
when_to_use_negative:
- No human-expert ground-truth annotations exist for the query cells — ARI cannot
  be computed and external validation metrics or marker-based review is needed instead
- Labelings cover different subsets of cells (e.g., after removing unannotated cells),
  so pairings are not one-to-one and ARI would be invalid
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scanpy Ingest
  role: Generates the label-transfer labeling that is one of the three candidates
    scored by ARI
- name: scanpy
  role: Provides Leiden clustering (unconstrained at resolution 1) used as the baseline
    labeling
- name: Cellar
  role: Interactive tool implementing semi-supervised Leiden with frozen clusters
    whose refined labeling is evaluated by ARI
  repo: github.com/euxhenh/cellar
- name: AnnData (Annotated Data object)
  role: In-memory format holding query cells, transferred labels, and expert ground-truth
    annotations for ARI computation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-29744-0
    title: Interactive single-cell data analysis using Cellar
  source_content_hash: sha256:bc779b7c94216be88ef9473e58df1fbd6453c7eb70e64ef6633d8421c0985108
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - AnnData
  - AnnData (Annotated Data object)
  - Cellar
  - scanpy
  - scanpy Ingest
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# adjusted-rand-index-clustering-evaluation

## Summary

Compute the adjusted Rand index (ARI) to quantitatively compare cell-type labelings (label transfer, unconstrained clustering, semi-supervised clustering) against human-expert ground truth, as done in Cellar's HuBMAP spleen evaluation where frozen-cluster Leiden reached ARI 0.66 vs Ingest 0.39 and unconstrained Leiden 0.27.

## When to use

Use when multiple cell-type annotation labelings of the same cells (e.g., scanpy Ingest transfer vs Leiden variants) must be scored against expert annotations to rank methods on a single dataset.

## When NOT to use

- No human-expert ground-truth annotations exist for the query cells — ARI cannot be computed and external validation metrics or marker-based review is needed instead
- Labelings cover different subsets of cells (e.g., after removing unannotated cells), so pairings are not one-to-one and ARI would be invalid

## Inputs

- AnnData object of the reference dataset (HBMP3-spleen-CC3) with expert cell-type labels
- AnnData object of the query dataset (HBMP2-spleen-2) with ground-truth human-expert annotations
- Candidate label vectors per cell: Ingest-transferred labels, unconstrained Leiden labels (resolution 1), semi-supervised frozen-cluster Leiden labels (clusters 0, 4, 9, 10 frozen)

## Outputs

- Per-labeling ARI scores against ground truth (Ingest 0.39, unconstrained Leiden 0.27, frozen-cluster Leiden 0.66)
- Comparison table saved as ari_comparison.csv

## How to apply

Load the reference (HBMP3-spleen-CC3) and query (HBMP2-spleen-2) datasets as AnnData objects, keeping only genes overlapping between them. Generate each candidate labeling: scanpy Ingest label transfer in a 40-component PCA latent space fit on the reference, unconstrained Leiden clustering at resolution 1, and semi-supervised Leiden at resolution 1 with the least-noisy clusters (0, 4, 9, 10) frozen during iterations. Compute ARI of each labeling against the ground-truth cell-type annotations assigned by a human expert, then assemble the scores into a comparison table (ari_comparison.csv). Rationale: ARI corrects for chance agreement, so it fairly ranks labelings that partition the same cells into different numbers of clusters; the expected ordering is semi-supervised Leiden (0.66) > Ingest (0.39) > unconstrained Leiden (0.27), showing freezing confident clusters refines label transfer.

## Related tools

- **scanpy Ingest** (Generates the label-transfer labeling that is one of the three candidates scored by ARI)
- **scanpy** (Provides Leiden clustering (unconstrained at resolution 1) used as the baseline labeling)
- **Cellar** (Interactive tool implementing semi-supervised Leiden with frozen clusters whose refined labeling is evaluated by ARI) — github.com/euxhenh/cellar
- **AnnData (Annotated Data object)** (In-memory format holding query cells, transferred labels, and expert ground-truth annotations for ARI computation)

## Examples

```
from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(query.obs['truth_labels'], query.obs['leiden_frozen'])
```

## Evaluation signals

- Reproduced ARI values match the reported ordering and magnitudes of Supplementary Fig. 2: frozen-cluster Leiden 0.66 > Ingest 0.39 > unconstrained Leiden 0.27
- Each labeling assigns exactly one label per query cell and is paired cell-for-cell with the expert ground-truth vector before scoring
- All three labelings are computed on the same cell set restricted to genes overlapping between the two AnnData objects
- Semi-supervised run constraints: resolution 1 with clusters 0, 4, 9, and 10 frozen; deviating frozen-cluster choice should lower the ARI
- Output ari_comparison.csv contains exactly three ARI rows corresponding to the three methods

## Limitations

- ARI depends on quality and consistency of the human-expert ground truth; no accession IDs were given for the HuBMAP annotated reference datasets, complicating exact reproduction
- The frozen-cluster choice (0, 4, 9, 10 as least noisy) is dataset-specific and was selected by inspecting cluster noise, requiring user/domain input
- No software versions stated for Cellar, Python, Dash, or faiss, and reproducibility partially depends on the external web server rather than a locally archived artifact

## Evidence

- [methods] Label transfer between HBMP2-spleen-2 and HBMP3-spleen-CC3 was performed using scanpy's Ingest: [evidence span withheld — anchor status: not_found]
- [intro] Cellar provides methods for semi-supervised clustering: "Cellar provides methods for semi-supervised clustering and projection of expression clusters in spatial single-cell images"
- [discussion] approximate nearest neighbors based on faiss: "approximate nearest neighbors based on faiss25 to speed up neighbors graph construction for Leiden clustering"
- [methods] Preprocessing of the data was done via scanpy: "Preprocessing of the data was done via scanpy"
