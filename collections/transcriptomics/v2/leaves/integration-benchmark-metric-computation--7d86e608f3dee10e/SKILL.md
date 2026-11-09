---
name: integration-benchmark-metric-computation
description: Use when comparing single-cell diagonal integration methods (e.g. scRNA-scATAC
  or scRNA-surface protein) on unpaired data and you need quantitative, per-method
  performance scores to produce a ranked benchmark table.
when_to_use_negative:
- When integrating paired (multiome-like) data where cells are matched across modalities
  — the paper's benchmark targets unpaired diagonal integration settings
- When cross-modality feature connections are unavailable (e.g. integrating electrophysiology
  with gene expression), since the embedding pipeline these metrics evaluate depends
  on such features
- When the goal is single-modality clustering or batch correction rather than cross-modality
  integration benchmarking
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scConfluence
  role: Method under benchmarking; provides joint latent embeddings via uncoupled
    autoencoders and regularized Inverse Optimal Transport that are scored by the
    metrics
  repo: https://github.com/cantinilab/scconfluence
- name: Seurat v4.3.0
  role: Baseline integration method (anchor-based) whose outputs are scored with the
    same metrics
- name: rliger v1.0.0
  role: Baseline iNMF integration method whose outputs are scored with the same metrics
- name: MultiMAP
  role: Baseline integration method scored with the same metrics
- name: Uniport
  role: Baseline integration method scored with the same metrics
- name: scGLUE
  role: Baseline integration method scored with the same metrics
- name: Muon
  role: Provides the MuData object holding each method's embedding in obsm for metric
    computation
- name: Scanpy
  role: Neighbors + UMAP computation for visualizing the joint embeddings alongside
    metric evaluation
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:26b6d6e2f20afc2862beba29378b4b25b55d04322da670ab74669e16dfcc49d4
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
  - Seurat
  - Seurat v4.3.0
  - Uniport
  - rliger v1.0.0
  - scConfluence
  - scGLUE
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 16
---

# integration-benchmark-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking procedure from the scConfluence paper (Nat Commun 2024) that scores diagonal integration methods (scConfluence, Seurat, Liger, MultiMAP, Uniport, scGLUE) on unpaired single-cell multimodal tasks using four quantitative metrics — cell-type purity, label-transfer accuracy, connectivity of the latent embedding, and FOSCTTM — across balanced and unbalanced population scenarios.

## When to use

Use when comparing single-cell diagonal integration methods (e.g. scRNA-scATAC or scRNA-surface protein) on unpaired data and you need quantitative, per-method performance scores to produce a ranked benchmark table.

## When NOT to use

- When integrating paired (multiome-like) data where cells are matched across modalities — the paper's benchmark targets unpaired diagonal integration settings
- When cross-modality feature connections are unavailable (e.g. integrating electrophysiology with gene expression), since the embedding pipeline these metrics evaluate depends on such features
- When the goal is single-modality clustering or batch correction rather than cross-modality integration benchmarking

## Inputs

- Joint latent embeddings from each integration method (e.g. mdata.obsm['latent'] from scConfluence, plus embeddings from Seurat, Liger, MultiMAP, Uniport, scGLUE)
- Cell type labels for each modality (e.g. HeLa/HCT/K562 line identities for the scCATseq cell-line data)
- Original (pre-integration) cross-modality distance matrices for FOSCTTM computation
- Scenario/dataset identifiers (four unbalancing scenarios; four gold-standard datasets: Multiome PBMC, OP Multiome, BMCITE, OP Cite)

## Outputs

- benchmark_metrics.csv: tidy table with columns method, dataset/scenario, and metric values (purity, transfer accuracy, connectivity, FOSCTTM, cell type FOSCTTM)
- Validation summary confirming all methods x scenarios rows are present with finite metric values
- Method ranking comparable to Fig. 2b and Fig. 3b-e of the paper

## How to apply

After running each integration method on the same data splits, extract each method's joint embedding (e.g. from mdata.obsm['latent'] for scConfluence) and compute the four evaluation metrics: cell-type purity, label-transfer accuracy, connectivity of the latent embedding, and FOSCTTM (fraction of samples closer in the original data than in the embedding). For unbalanced scenarios, evaluate all metrics per scenario — e.g. the scCATseq HeLa/HCT/K562 cell-line benchmark with four subsampling levels: full/balanced, 50% of K562 removed, 100% of K562 removed, and both K562 and HCT removed. On the gold-standard benchmarks (Multiome PBMC 10X, Open problems Multiome, BMCITE, OP Cite) also compute cell type FOSCTTM. Assemble all values into a tidy table with columns for method, scenario/dataset, and each metric, written to benchmark_metrics.csv, then verify the qualitative ranking reproduces the paper's Fig. 2b / Fig. 3b-e finding that scConfluence outperforms the state-of-the-art.

## Related tools

- **scConfluence** (Method under benchmarking; provides joint latent embeddings via uncoupled autoencoders and regularized Inverse Optimal Transport that are scored by the metrics) — https://github.com/cantinilab/scconfluence
- **Seurat v4.3.0** (Baseline integration method (anchor-based) whose outputs are scored with the same metrics)
- **rliger v1.0.0** (Baseline iNMF integration method whose outputs are scored with the same metrics)
- **MultiMAP** (Baseline integration method scored with the same metrics)
- **Uniport** (Baseline integration method scored with the same metrics)
- **scGLUE** (Baseline integration method scored with the same metrics)
- **Muon** (Provides the MuData object holding each method's embedding in obsm for metric computation)
- **Scanpy** (Neighbors + UMAP computation for visualizing the joint embeddings alongside metric evaluation)

## Examples

```
import pandas as pd; metrics = compute_purity(latent, labels), compute_transfer_acc(latent, labels), compute_connectivity(latent, labels), compute_foscttm(latent, orig_dists); pd.DataFrame([{**m, 'method': method, 'scenario': sc} for ...]).to_csv('benchmark_metrics.csv', index=False)
```

## Evaluation signals

- benchmark_metrics.csv contains all methods x scenarios rows (e.g. 6 methods x 4 scenarios for the cell-line benchmark) with finite values for all four metrics
- Qualitative ranking reproduces Fig. 2b: scConfluence beats all five baselines in every unbalancing scenario
- On the CITE-seq benchmarks, scConfluence outperforms the five baselines on all four metrics (purity, transfer accuracy, connectivity, FOSCTTM) on both datasets
- All methods show performance degradation as scenarios become less balanced (full → 50% K562 removed → 100% K562 removed → K562 and HCT removed), with scConfluence degrading least
- UMAP visualizations of joint embeddings per method and dataset reproduce the layout of Fig. 3b-e

## Limitations

- Rare cell populations are hard to detect and under-represented in parameter estimation, especially with mini-batch gradient descent, affecting metric stability for rare types
- Metric comparability assumes all methods ran on identical data splits and preprocessing; pipelines differ (R Seurat/Liger vs Python methods)
- The benchmark cannot resolve behavior on modality-specific populations absent in one modality for methods lacking unbalanced OT handling; scConfluence handles this via the unbalanced relaxation of Optimal Transport

## Evidence

- [methods] Benchmark scoring per scenario with four metrics: [evidence span withheld — anchor status: not_found]
- [results] scConfluence robustness finding: [evidence span withheld — anchor status: not_found]
- [results] CITE-seq benchmark result: [evidence span withheld — anchor status: not_found]
