---
name: winning-fraction-cluster-pruning
description: Use when scTriangulate has produced per-cell winning annotations and
  raw winning clusters need filtering by winning fraction (win_fraction_cutoff) or
  minimum size (abs_thresh) before downstream analysis.
when_to_use_negative:
- When the analysis goal is a consensus/ensemble clustering rather than decision-level
  integration — ensemble approaches like ClusterEnsembles select the consensus instead
  of pruning by stability.
- When fewer than two conflicting annotation columns exist in adata.obs, since pruning
  operates on reconciled multi-annotation winning clusters.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scTriangulate
  role: Performs pruning of raw winning clusters via win_fraction_cutoff and abs_thresh
    and reclassifies pruned cells using a nearest-neighbor strategy within the lazy_run/postprocessing
    workflow
  repo: https://github.com/frankligy/scTriangulate
- name: scanpy
  role: Provides the AnnData framework and neighborhood infrastructure supporting
    the nearest-neighbor reclassification of cells from pruned clusters
  repo: https://github.com/frankligy/scTriangulate
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-36016-y
    title: Decision level integration of unimodal and multimodal single cell data
      with scTriangulate
  source_content_hash: sha256:230069d40fd7c3841bac7529012acc8d2d1fb92d349022b3c66475849623a9d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/frankligy/scTriangulate
  tools:
  - scTriangulate
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# winning-fraction-cluster-pruning

## Summary

Prunes unstable or undersized winning clusters produced by scTriangulate's Shapley-based cell assignment, then reclassifies their cells to a nearest-neighbor cluster label, yielding a cleaned final clustering solution.

## When to use

Use when scTriangulate has produced per-cell winning annotations and raw winning clusters need filtering by winning fraction (win_fraction_cutoff) or minimum size (abs_thresh) before downstream analysis.

## When NOT to use

- When the analysis goal is a consensus/ensemble clustering rather than decision-level integration — ensemble approaches like ClusterEnsembles select the consensus instead of pruning by stability.
- When fewer than two conflicting annotation columns exist in adata.obs, since pruning operates on reconciled multi-annotation winning clusters.

## Inputs

- Per-cell winning cluster assignments from scTriangulate Shapley selection (e.g. exported from sctri.lazy_run())
- Per-cluster stability metrics (reassign score, TFIDF5, TFIDF10, SCCAF score)
- AnnData object with conflicting annotation columns in adata.obs and adata.obsm['X_umap']

## Outputs

- Final per-cell pruned cluster labels (e.g. sctriangulate_pbmc10k_labels.csv)
- Per-cell winning-fraction confidence values
- Per-cluster stability metrics table with pruned clusters flagged

## How to apply

After the Shapley-value step assigns each cell to the annotation cluster with the highest Phi value, compute for each raw winning cluster the fraction of cells whose final assignment matches the original cluster-level assignment. Mark clusters as unstable when this winning fraction is below the user-defined win_fraction_cutoff (default 0.25) and flag clusters smaller than abs_thresh (default 10 cells) as unreliable, following the paper's finding that the default threshold of 10 produced consistent reliable results. Re-classify cells from pruned clusters using a nearest-neighbor strategy so no cell is dropped. Finally, export the per-cell pruned labels, per-cell winning-fraction confidence values, and per-cluster stability metrics (reassign, TFIDF5, TFIDF10, SCCAF) for validation against reference annotations such as Azimuth L2.

## Related tools

- **scTriangulate** (Performs pruning of raw winning clusters via win_fraction_cutoff and abs_thresh and reclassifies pruned cells using a nearest-neighbor strategy within the lazy_run/postprocessing workflow) — https://github.com/frankligy/scTriangulate
- **scanpy** (Provides the AnnData framework and neighborhood infrastructure supporting the nearest-neighbor reclassification of cells from pruned clusters) — https://github.com/frankligy/scTriangulate

## Examples

```
sctri = ScTriangulate(dir='./output', adata=adata, query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3']); sctri.lazy_run(win_fraction_cutoff=0.25, abs_thresh=10)
```

## Evaluation signals

- No pruned cluster retains a winning fraction below the win_fraction_cutoff of 0.25 in the final labels.
- No retained cluster contains fewer than abs_thresh (default 10) cells after pruning and reassignment.
- Every input cell receives exactly one final label (binary assignment matrix X of shape I * N — each cell assigned one annotation).
- Final labels agree with independent reference annotations (e.g. Azimuth L2) at a level comparable to the paper's PBMC 10k benchmark.
- Per-cluster stability metrics (reassign, TFIDF5, TFIDF10, SCCAF) are exported for every retained cluster.

## Limitations

- The default thresholds (win_fraction_cutoff 0.25, abs_thresh 10) are programmatic defaults that may not be optimal for all study designs, as the discussion notes.
- Integrating cluster predictions from dozens of annotation sources significantly increases runtime of the upstream Shapley step feeding the pruning stage.
- Naming of the pruned cell populations remains largely non-standardized and must be curated manually.

## Evidence

- [methods] win_fraction_cutoff default 0.25: "By default, wfc < 0.25 will be labeled as unstable clusters."
- [methods] abs_thresh default 10: "the default threshold of 10 produced consistent reliable results for different evaluated datasets (Supplementary Figure 7a)."
- [methods] pruning and nearest-neighbor reclassification: "the final annotation assignments are pruned according to the fraction of final versus original cell-cluster assignment (user-defined) and re-classified using a nearest-neighbor strategy."
- [discussion] one annotation per cell: "define a binary assignment matrix X of the shape I * N, where each cell can only be definitively assigned to one annotation"
- [readme] README lazy_run invocation: "sctri = ScTriangulate(dir='./output',adata=adata,query=['sctri_rna_leiden_1','sctri_rna_leiden_2','sctri_rna_leiden_3'])
sctri.lazy_run()"
