---
name: spatial-adjacency-hit-number-evaluation
description: Use when evaluating whether a method's inferred single-cell spatial coordinates
  or SC-ST mapping reproduce the true spatial neighborhood structure of cells, using
  semi-simulated SC/ST data with known ground-truth positions and adjacency.
when_to_use_negative:
- Real ST data where no ground-truth single-cell coordinates or adjacency exist —
  MAE and hit-number@K cannot be computed, so cell-type PCC alone applies.
- Evaluating spot-level deconvolution methods that output cell-type proportions per
  spot rather than per-cell spatial coordinates, since the single-cell adjacency metric
  presumes per-cell position predictions.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: STEM
  role: Primary spatial mapping method whose embeddings (SOmodel, netE) are evaluated
    by the hit-number@K and MAE metrics
  repo: https://github.com/WhirlFirst/STEM
- name: CellTrek
  role: Baseline method whose predicted coordinates and adjacency are scored with
    the same metrics
- name: scSpace
  role: Baseline method scoring predicted coordinates against ground truth
- name: Seurat
  role: Baseline integration method whose SC-ST integrated graph outputs are converted
    to predicted coordinates for scoring
- name: Spaotsc
  role: Baseline optimal-transport mapping method evaluated with the same metrics
- name: Tangram
  role: Baseline mapping-matrix method whose SC-ST matrix is converted to coordinates
    for scoring
- name: scanpy
  role: Used to normalize (normalize_total, log1p) inputs and compute qc metrics such
    as n_genes_by_counts used for dropout rate
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:59cf4c4113e96ae4dc02c0c587c9b1c278f526f5d5742a930acc588c5ffae1ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - CellTrek
  - STEM
  - Seurat
  - Spaotsc
  - Tangram
  - scSpace
  - scanpy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# spatial-adjacency-hit-number-evaluation

## Summary

Benchmark metric that scores predicted single-cell spatial adjacency by counting correct single-cell (SC) neighbors among K nearest cells (K=200, hit-number@K) against ground-truth adjacency, alongside MAE of predicted vs. ground-truth coordinates, to compare spatial mapping methods such as STEM against CellTrek, scSpace, Seurat, Spaotsc, and Tangram.

## When to use

Use when evaluating whether a method's inferred single-cell spatial coordinates or SC-ST mapping reproduce the true spatial neighborhood structure of cells, using semi-simulated SC/ST data with known ground-truth positions and adjacency.

## When NOT to use

- Real ST data where no ground-truth single-cell coordinates or adjacency exist — MAE and hit-number@K cannot be computed, so cell-type PCC alone applies.
- Evaluating spot-level deconvolution methods that output cell-type proportions per spot rather than per-cell spatial coordinates, since the single-cell adjacency metric presumes per-cell position predictions.

## Inputs

- Predicted spatial coordinates per single cell from each compared method (unified coordinate format)
- Ground-truth single-cell spatial coordinates from the semi-simulation (simu_st_metadata.csv spatial metadata for E1z2)
- Ground-truth SC-SC spatial adjacency derived from true coordinates
- Ground-truth ST spot cell-type labels (simu_st_celltype.csv)
- Predicted SC-ST mapping matrices (e.g. netsc2st / netsc2st softmax matrices)

## Outputs

- Per-method MAE between predicted and ground-truth spatial coordinates
- Per-method hit-number@K (K=200 correct SC neighbors count)
- Per-cell-type PCC of cell-type proportions for every method
- Aggregated metrics table written as benchmark_metrics.csv

## How to apply

After unifying each method's output into predicted coordinates, an SC-SC adjacency structure, and an SC-ST mapping matrix, compute the MAE between predicted and ground-truth spatial coordinates for every cell. Then compute hit-number@K: for each cell, take its K=200 nearest neighbors under the predicted embedding/adjacency and count how many are correct SC neighbors in the ground-truth adjacency derived from true coordinates. Also compute per-cell-type Pearson correlation (PCC) of cell-type proportions between predicted spatial assignments and ground-truth ST cell types. Assemble per-method MAE, hit-number@K, and per-cell-type PCC into a single metrics table (benchmark_metrics.csv); on the E1z2 semi-simulation the correct method should show roughly two-fold higher hit number than the second-best method and the lowest MAE.

## Related tools

- **STEM** (Primary spatial mapping method whose embeddings (SOmodel, netE) are evaluated by the hit-number@K and MAE metrics) — https://github.com/WhirlFirst/STEM
- **CellTrek** (Baseline method whose predicted coordinates and adjacency are scored with the same metrics)
- **scSpace** (Baseline method scoring predicted coordinates against ground truth)
- **Seurat** (Baseline integration method whose SC-ST integrated graph outputs are converted to predicted coordinates for scoring)
- **Spaotsc** (Baseline optimal-transport mapping method evaluated with the same metrics)
- **Tangram** (Baseline mapping-matrix method whose SC-ST matrix is converted to coordinates for scoring)
- **scanpy** (Used to normalize (normalize_total, log1p) inputs and compute qc metrics such as n_genes_by_counts used for dropout rate)

## Examples

```
hits = sum(1 for n in knn_indices(pred_coor, k=200).ravel() if n in gt_neighbors[cell]); mae = np.abs(pred_coor - gt_coor).mean()
```

## Evaluation signals

- On E1z2 semi-simulation, the reference method (STEM) achieves roughly two-fold the hit number of the second-best method among CellTrek, scSpace, Seurat, Spaotsc, Tangram.
- STEM consistently attains the lowest MAE between predicted and ground-truth spatial coordinates among all compared methods.
- Per-cell-type PCC of cell-type proportions is computed for every method and assembled in the same metrics table — verify all three metrics present per method.
- Hit-number@K uses K=200: verify the neighbor count parameter equals 200 and correctness is judged against ground-truth adjacency, not predicted adjacency.
- benchmark_metrics.csv contains one row/entry per method with MAE, hit-number@K, and per-cell-type PCC values.

## Limitations

- The metric requires semi-simulated data with known ground-truth single-cell spatial coordinates; it cannot be computed on real datasets without a validated spatial ground truth.
- Performance conclusions are specific to the E1z2 mouse embryo SeqFISH semi-simulation and may not generalize to ST and SC data from different donors in the same disease state, which the authors left as untested future work.
- Dropout-rate-dependent preprocessing (dp = 1 - median SC n_genes_by_counts / median ST n_genes_by_counts, ~0.584 for E1z2) affects the upstream mapping quality that these metrics score.

## Evidence

- [methods] hit-number@K counting correct SC neighbors among K=200 using ground-truth adjacency: [evidence span withheld — anchor status: not_found]
- [results] STEM two-fold hit number and lowest MAE: [evidence span withheld — anchor status: not_found]
- [methods] unify baseline outputs: [evidence span withheld — anchor status: not_found]
- [readme] mapping matrices by softmax of dot product: "netst2sc = F.softmax(stembedding.mm(scembedding.t()),dim=1).detach().cpu().numpy()"
- [methods] predicted coordinates from all_coord: "adata.obsm['spatialDA'] = all_coord(pd.DataFrame(netsc2st,index=adata.obs_names,columns=stadata.obs_names),spcoor)"
