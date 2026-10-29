---
name: label-transfer-accuracy-evaluation
description: Use when benchmarking single-cell diagonal integration methods (scConfluence,
  Seurat, rliger, MultiMAP, Uniport, scGLUE) on unpaired modalities and you need a
  label-based accuracy score for cell-type agreement in the joint embedding, especially
  under unbalanced population scenarios.
when_to_use_negative:
- Data are paired multimodal (same cells in both modalities) — diagonal integration
  and label transfer across cells are not the right evaluation
- Cell-type annotations are absent or not shared between the two modalities, so labels
  cannot be transferred or scored
- Modality pairs lack cross-modality feature connections (e.g. electrophysiology vs
  gene expression), which scConfluence-based latent spaces require
tools:
- name: scConfluence
  role: Produces the latent embeddings whose label-transfer accuracy is scored; instantiated
    via per-modality scconfluence AutoEncoders and model.get_latent()
  repo: https://github.com/cantinilab/scconfluence
- name: Seurat v4.3.0
  role: Baseline whose anchor-based integration embeddings are scored by label-transfer
    accuracy
- name: rliger v1.0.0
  role: Baseline (iNMF integration) scored by label-transfer accuracy
- name: MultiMAP
  role: Baseline integration method scored by label-transfer accuracy
- name: Uniport
  role: Baseline OT-based integration method scored by label-transfer accuracy
- name: scGLUE
  role: Baseline graph-coupled integration method scored by label-transfer accuracy
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:1d78e4ed5691e2e4e6b30e090f7621dfaaa4781d9dbc4761688d7a04789f0e72
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.85
n_claims: 20
---

# label-transfer-accuracy-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies diagonal-integration quality by training a label-transfer classifier on one modality's cell-type labels in the shared latent space and scoring correctness on the other modality. It is one of the four metrics (with cell-type purity, connectivity, and FOSCTTM) used in the scConfluence benchmarks on unbalanced scCATseq HeLa/HCT/K562 cell-line data.

## When to use

Use when benchmarking single-cell diagonal integration methods (scConfluence, Seurat, rliger, MultiMAP, Uniport, scGLUE) on unpaired modalities and you need a label-based accuracy score for cell-type agreement in the joint embedding, especially under unbalanced population scenarios.

## When NOT to use

- Data are paired multimodal (same cells in both modalities) — diagonal integration and label transfer across cells are not the right evaluation
- Cell-type annotations are absent or not shared between the two modalities, so labels cannot be transferred or scored
- Modality pairs lack cross-modality feature connections (e.g. electrophysiology vs gene expression), which scConfluence-based latent spaces require

## Inputs

- MuData object with scRNA-seq and scATAC-seq counts (scCATseq HeLa/HCT/K562 cell lines) and cell-type annotations
- Latent embeddings per method stored in mdata.obsm['latent']
- Four unbalanced scenario splits (balanced, K562-50%, K562-100%, K562+HCT removed)

## Outputs

- Label-transfer accuracy value per method per scenario (fraction in [0,1])
- Rows in benchmark_metrics.csv with columns method, scenario, label-transfer accuracy (plus purity, connectivity, FOSCTTM)

## How to apply

Run each integration method (scConfluence via model.get_latent() into mdata.obsm['latent']; Seurat v4.3.0 anchors, rliger v1.0.0 iNMF, MultiMAP, Uniport, scGLUE per their default pipelines) on the same four scCATseq splits: full/balanced, 50% K562 removed, 100% K562 removed, and K562+HCT removed. For each method and scenario, transfer cell-type labels from the scRNA modality to the scATAC modality using the shared latent embedding, then compute the fraction of correctly transferred labels. Rationale: because scConfluence uses unbalanced OT relaxation to handle populations absent in a modality, label-transfer accuracy specifically tests whether cell identities are preserved when population proportions differ across modalities. Assemble per-method, per-scenario values into benchmark_metrics.csv alongside cell-type purity, connectivity, and FOSCTTM, then check that the ranking reproduces Fig. 2b where scConfluence outperforms all five baselines in every scenario.

## Related tools

- **scConfluence** (Produces the latent embeddings whose label-transfer accuracy is scored; instantiated via per-modality scconfluence AutoEncoders and model.get_latent()) — https://github.com/cantinilab/scconfluence
- **Seurat v4.3.0** (Baseline whose anchor-based integration embeddings are scored by label-transfer accuracy)
- **rliger v1.0.0** (Baseline (iNMF integration) scored by label-transfer accuracy)
- **MultiMAP** (Baseline integration method scored by label-transfer accuracy)
- **Uniport** (Baseline OT-based integration method scored by label-transfer accuracy)
- **scGLUE** (Baseline graph-coupled integration method scored by label-transfer accuracy)

## Examples

```
from sklearn.neighbors import KNeighborsClassifier; knn = KNeighborsClassifier().fit(latent_rna, labels_rna); acc = (knn.predict(latent_atac) == labels_atac).mean()
```

## Evaluation signals

- benchmark_metrics.csv contains exactly 6 methods x 4 scenarios = 24 rows with finite values for label-transfer accuracy and the other three metrics
- Label-transfer accuracy lies in [0, 1] for every method-scenario pair
- Qualitative ranking reproduces Fig. 2b: scConfluence beats all five baselines in every unbalancing scenario
- Accuracy degrades for all methods as scenarios become less balanced (full -> K562 50% -> K562 100% -> K562+HCT removed), but scConfluence degrades least

## Limitations

- Rare cell populations are hard to classify and under-represented, especially since mini-batch gradient descent rarely co-samples rare populations from each modality
- All methods degrade as scenarios become less balanced, so absolute accuracy values depend strongly on the level of population unbalancing

## Evidence

- [methods] four evaluation metrics: [evidence span withheld — anchor status: not_found]
- [results] validation of metric table: [evidence span withheld — anchor status: not_found]
