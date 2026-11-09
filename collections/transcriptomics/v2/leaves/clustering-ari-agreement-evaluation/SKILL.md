---
name: clustering-ari-agreement-evaluation
description: Use when an ICAT run (control Louvain → NCFS upweighting → semi-supervised
  Louvain) has produced `sslouvain` assignments and you must quantify recovery against
  simulated ground-truth `Population` labels via adjusted Rand index.
when_to_use_negative:
- Real experimental scRNA-seq data where no ground-truth cell-state labels exist —
  ARI cannot be computed without a reference partition, so this skill is restricted
  to simulated benchmarks where `Population` labels are known.
- Evaluating the control-arm clustering alone (first-step Louvain on controls) before
  NCFS upweighting and semi-supervised clustering have been run — there are no `sslouvain`
  assignments to score yet.
- Comparing clusterings across methods on data where treatment status leaks into the
  labels (e.g. scoring with treatment rather than population identity), which would
  not measure control-guided state recovery.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: icat-sc
  role: Produces the `sslouvain` cluster assignments (via `models.icat` with NCFS
    and semi-supervised Louvain) that are scored against true labels; supplies `simulate.SingleCellDataset`
    and `simulate.perturb` to generate labeled ground truth.
  repo: https://github.com/BradhamLab/icat
- name: scanpy
  role: Holds the data as AnnData, performs `sc.pp.log1p` normalization and `sc.tl.umap`
    visualization of the upweighted space, and carries the `sslouvain` and `Population`
    columns used in the ARI comparison.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad278
    title: 'ICAT: a novel algorithm to robustly identify cell states following perturbations
      in single-cell transcriptomes'
  source_content_hash: sha256:e04f860b3733623bb43f9b95ba283f8d3b90955fe4c25f956b24be5b5020c44a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BradhamLab/icat
  tools:
  - icat-sc
  - numpy
  - scanpy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# clustering-ari-agreement-evaluation

## Summary

Validates the ICAT two-step control-guided clustering output (the `sslouvain` labels in the returned AnnData) by computing the adjusted Rand index (ARI) against known ground-truth population labels, e.g. the `Population` column of simulated single-cell datasets. Because ICAT is unsupervised on real data, agreement scoring against simulated ground truth is the paper's acceptance criterion for correct reconstruction of the pipeline.

## When to use

Use when an ICAT run (control Louvain → NCFS upweighting → semi-supervised Louvain) has produced `sslouvain` assignments and you must quantify recovery against simulated ground-truth `Population` labels via adjusted Rand index.

## When NOT to use

- Real experimental scRNA-seq data where no ground-truth cell-state labels exist — ARI cannot be computed without a reference partition, so this skill is restricted to simulated benchmarks where `Population` labels are known.
- Evaluating the control-arm clustering alone (first-step Louvain on controls) before NCFS upweighting and semi-supervised clustering have been run — there are no `sslouvain` assignments to score yet.
- Comparing clusterings across methods on data where treatment status leaks into the labels (e.g. scoring with treatment rather than population identity), which would not measure control-guided state recovery.

## Inputs

- AnnData object (`adata`) containing concatenated control and perturbed cells with ICAT output labels in `adata.obs['sslouvain']`
- Ground-truth per-cell labels (`adata.obs['Population']`) from `simulate.SingleCellDataset`
- Optional hyperparameter set used for the run (NCFS `reg`, `sigma`; Louvain/sslouvain `n_neighbors`, `resolution`)

## Outputs

- Scalar adjusted Rand index (ARI) between `sslouvain` clusters and true population labels
- Per-cell label table plus ARI written to `icat_two_step_results.csv`
- Optional `sc.tl.umap` projection of `adata.obsm['X_icat']` colored by `sslouvain` and `Population` for visual confirmation

## How to apply

Run the ICAT workflow on a simulated dataset built with `simulate.SingleCellDataset` (populations=2, genes=1000, dispersion drawn from [1, 2, 3]), perturb the controls with `simulate.perturb`, concatenate both arms, and normalize with scanpy `sc.pp.log1p`. After Louvain-clustering the control arm, NCFS feature weighting (`ncfs_kws {'reg': 1, 'sigma': 3}`), and semi-supervised Louvain (`neighbor_kws {'n_neighbors': 15}`, `cluster_kws {'resolution': 0.75}`) with control labels immutable and perturbed cells initialized as singletons, collect the final assignments from the `sslouvain` column of the returned AnnData. Compute the adjusted Rand index between `sslouvain` and the true simulated population labels (the `Population` column carried through from simulation). Write the per-cell labels plus the ARI to `icat_two_step_results.csv`. The reconstruction is accepted when ARI is approximately 1 on the equal-population simulated dataset; intermediate ARI values indicate that hyperparameters (`sigma`, `reg`, `n_neighbors`, `resolution`) should be re-tuned via the `optimize` grid-search functions.

## Related tools

- **icat-sc** (Produces the `sslouvain` cluster assignments (via `models.icat` with NCFS and semi-supervised Louvain) that are scored against true labels; supplies `simulate.SingleCellDataset` and `simulate.perturb` to generate labeled ground truth.) — https://github.com/BradhamLab/icat
- **scanpy** (Holds the data as AnnData, performs `sc.pp.log1p` normalization and `sc.tl.umap` visualization of the upweighted space, and carries the `sslouvain` and `Population` columns used in the ARI comparison.)

## Examples

```
out = model.cluster(adata, adata.obs['treatment']); from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(out.obs['sslouvain'], out.obs['Population'])
```

## Evaluation signals

- Adjusted Rand index between `sslouvain` assignments and true `Population` labels is approximately 1 on the equal-population simulated dataset (populations=2, genes=1000, dispersion from [1, 2, 3]) — the stated acceptance threshold.
- Output file `icat_two_step_results.csv` exists and contains per-cell labels plus the ARI value.
- Control-cell labels are immutable: every control cell's `sslouvain` label matches its pre-computed control Louvain label (`control_clusters`).
- The returned AnnData contains an `sslouvain` column in `obs` and the upweighted matrix in `obsm['X_icat']` with neighbors defined in that space, allowing `sc.tl.umap` to run directly.
- UMAP of `obsm['X_icat']` colored by `sslouvain` and `Population` shows matching cluster assignments across control and perturbed arms.

## Limitations

- ARI requires ground-truth labels, so this evaluation is only meaningful on simulated data; on real perturbation experiments ICAT's clustering is unsupervised and cannot be scored this way.
- Results depend on hyperparameters (`sigma`, `reg`, `n_neighbors`, `resolution`); the README recommends grid-searching Louvain and NCFS parameters prior to the full workflow, so a low ARI may reflect suboptimal parameters rather than a broken implementation.
- ICAT does not automatically compute UMAP, t-SNE, or other reduced-dimension visualizations during clustering — the visual sanity check must be constructed explicitly from `adata.obsm['X_icat']`.

## Evidence

- [readme] final clustering is stored in the `sslouvain` column of the returned `AnnData` object: "The final clustering is stored in the `sslouvain` column of the returned `AnnData` object."
- [intro] semi-supervised clustering preserves control labels while perturbed cells cluster freely: "perform semi-supervised clustering such that the originally identified control labels remain constant, but cells from experimental conditions are free to cluster with any other cells"
- [other] ICAT label initialization: control labels retained, treated cells as singletons: [evidence span withheld — anchor status: not_found]
- [other] ARI computed between sslouvain assignments and true simulated population labels: [evidence span withheld — anchor status: not_found]
- [readme] NCFS hyperparameters chosen by MCC over k-fold cross validation: "find the "best" `sigma` and `reg` NCFS values by measuring the MCC in a weighted KNN over k-fold cross validation"
