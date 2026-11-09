---
name: cell-type-unbalancing-simulation
description: Use when evaluating whether a single-cell diagonal integration method
  (scRNA with scATAC or surface protein) remains robust as cell populations become
  unbalanced or absent from one modality.
when_to_use_negative:
- When the integration task is paired/multiomic (same cells measured in both modalities)
  rather than unpaired diagonal integration
- When no cross-modality feature connections are available (e.g., integrating electrophysiology
  with gene expression), since scConfluence relies on weakly connected features for
  its cost matrix
- When the goal is detecting rare cell populations rather than stress-testing robustness
  to population imbalance — the paper notes rare populations are under-represented
  in mini-batch parameter estimation
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scConfluence
  role: Diagonal integration method with unbalanced (partial) Optimal Transport relaxation,
    evaluated across unbalanced scenarios
  repo: https://github.com/cantinilab/scconfluence
- name: Seurat v4.3.0
  role: Baseline anchor-based integration run on each unbalanced split
- name: rliger v1.0.0
  role: Baseline iNMF integration run on each unbalanced split
- name: Muon
  role: Preprocessing of modalities (tfidf for scATAC, filter_obs/filter_var QC) before
    subsampling
- name: Scanpy
  role: Log-normalization of scRNA counts (normalize_total to 10,000 then log1p)
- name: SciPy
  role: cdist computation of the cross-modality cost matrix from connected features
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:5ac903c6ba72ab2ecc62b21c623fd2c9242f0395256b81e14302aade525a6034
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cantinilab/scconfluence
  repo_url: https://github.com/cantinilab/scconfluence
  tools:
  - Liger
  - MultiMAP
  - Muon
  - Scanpy
  - SciPy
  - Seurat v4.3.0
  - Uniport
  - rliger v1.0.0
  - scConfluence
  - scGLUE
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# Cell-type unbalancing simulation for diagonal integration benchmarks

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking procedure that subsamples cell populations (e.g., K562 and HCT cell lines in scCATseq scRNA-scATAC data) to create graded unbalanced diagonal-integration scenarios, testing robustness of integration methods when proportions differ across modalities.

## When to use

Use when evaluating whether a single-cell diagonal integration method (scRNA with scATAC or surface protein) remains robust as cell populations become unbalanced or absent from one modality.

## When NOT to use

- When the integration task is paired/multiomic (same cells measured in both modalities) rather than unpaired diagonal integration
- When no cross-modality feature connections are available (e.g., integrating electrophysiology with gene expression), since scConfluence relies on weakly connected features for its cost matrix
- When the goal is detecting rare cell populations rather than stress-testing robustness to population imbalance — the paper notes rare populations are under-represented in mini-batch parameter estimation

## Inputs

- Preprocessed MuData object with scRNA-seq and scATAC-seq counts from HeLa, HCT, and K562 cell lines (scCATseq)
- Cross-modality cost matrix computed from connected features (scipy.spatial.distance.cdist) stored under mdata.uns
- Cell-type annotations for K562 and HCT populations used for targeted subsampling

## Outputs

- Four unbalanced scenario splits (balanced, 50% K562 removed, 100% K562 removed, K562 and HCT removed)
- Latent embeddings per method per scenario (e.g., mdata.obsm['latent'] from model.get_latent())
- benchmark_metrics.csv: tidy table with columns method, scenario, cell-type purity, label-transfer accuracy, connectivity, FOSCTTM (6 methods x 4 scenarios rows)

## How to apply

Starting from the preprocessed MuData object (scRNA log-normalized to 10,000 counts with log1p; scATAC TF-IDF normalized via Muon), generate four scenarios by subsampling cell populations: the full/balanced set, 50% of K562 cells removed, 100% of K562 removed, and both K562 and HCT removed. Run each integration method (scConfluence, Seurat v4.3.0 anchors, rliger v1.0.0 iNMF, MultiMAP, Uniport, scGLUE) on identical splits so degradation is attributable to unbalancing alone. For scConfluence, train uncoupled autoencoders with a cost matrix from connected features stored in mdata.uns (with mdata.uns['cross_keys']) and tune sinkhorn_loss_weight and reach; the unbalanced relaxation of Optimal Transport (ot.partial.partial_wasserstein, default transported mass m=0.5) lets the method handle modality-absent populations. Compute cell-type purity, label-transfer accuracy, connectivity, and FOSCTTM per method per scenario, and assemble a tidy table (method, scenario, metrics) as benchmark_metrics.csv. The rationale is that methods relying on balanced populations should degrade as scenarios become less balanced, while unbalanced OT-based methods should maintain latent mixing and structure preservation.

## Related tools

- **scConfluence** (Diagonal integration method with unbalanced (partial) Optimal Transport relaxation, evaluated across unbalanced scenarios) — https://github.com/cantinilab/scconfluence
- **Seurat v4.3.0** (Baseline anchor-based integration run on each unbalanced split)
- **rliger v1.0.0** (Baseline iNMF integration run on each unbalanced split)
- **Muon** (Preprocessing of modalities (tfidf for scATAC, filter_obs/filter_var QC) before subsampling)
- **Scanpy** (Log-normalization of scRNA counts (normalize_total to 10,000 then log1p))
- **SciPy** (cdist computation of the cross-modality cost matrix from connected features)

## Examples

```
import scconfluence, mudata as md; mdata = md.read_h5mu('cell_lines.h5mu'); model = scconfluence.ScConfluence(ae_dict={...}, cross_keys=mdata.uns['cross_keys']); model.fit(save_path='results'); mdata.obsm['latent'] = model.get_latent(mdata)
```

## Evaluation signals

- benchmark_metrics.csv contains all 6 methods x 4 scenarios rows with finite values for cell-type purity, label-transfer accuracy, connectivity, and FOSCTTM
- Qualitative ranking reproduces Fig. 2b: scConfluence outperforms all five baselines in every unbalancing scenario
- All methods should show metric degradation as scenarios become less balanced, confirming the unbalancing was effective
- scConfluence latent embeddings maintain mixing and structure preservation with tuned sinkhorn_loss_weight and reach (default partial-OT mass m=0.5)

## Limitations

- Rare cell populations are hard to detect and under-represented in parameter estimation, especially with mini-batch gradient descent, which rarely co-samples rare populations from each modality
- Only population-level subsampling is simulated; real biological imbalance may differ from artificial removal of K562/HCT cells
- Hyperparameter sensitivity and cell-type proportions per scenario are documented only in Supplementary Figs 13-18, which may not be available

## Evidence

- [results] Robustness to unbalanced populations: [evidence span withheld — anchor status: not_found]
- [intro] Unbalanced OT rationale: "thanks to the unbalanced relaxation of Optimal Transport33, scConfluence can also deal with cell types absent in a modality"
- [methods] Partial OT default: [evidence span withheld — anchor status: not_found]
