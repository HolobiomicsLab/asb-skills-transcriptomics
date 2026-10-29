---
name: hyperparameter-sensitivity-analysis
description: Use when a model's default hyperparameter (e.g. the unbalanced OT transported
  mass m=0.5 in scConfluence) lacks justification for your dataset and you need to
  demonstrate that embeddings are insensitive to its choice across progressively unbalanced
  modality scenarios.
when_to_use_negative:
- Prior information about the level of unbalanceness is available and a specific m
  is justified — a sweep adds little beyond confirming the known value
- The method being tuned does not expose a transported-mass parameter or relies on
  cross-modality feature connections that are unavailable (e.g. electrophysiology
  vs gene expression)
- The goal is benchmarking against other integration methods rather than characterizing
  robustness to one model's hyperparameter
edam_operation: http://edamontology.org/operation_3562
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scConfluence
  role: Multimodal model whose unbalanced OT transported-mass parameter m is swept
    and refit at each grid value; latent embeddings extracted with model.get_latent()
  repo: https://github.com/cantinilab/scconfluence
- name: POT
  role: Provides ot.partial.partial_wasserstein used to compute the unbalanced OT
    plan at each m value
- name: Scanpy
  role: Computes sc.pp.neighbors and sc.tl.umap on the 'latent' representation to
    derive graph connectivity and support metric scoring
- name: Muon
  role: Holds the preprocessed MuData object with cross-modality cost matrices in
    .uns
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:cdebdbda28c70e906754dd346d63493d97193ab5641c466a8b12f104ea47fcd7
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
  - Muon
  - POT
  - Scanpy
  - scConfluence
  edam_operation: http://edamontology.org/operation_3562
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# hyperparameter-sensitivity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Sweeps the unbalanced-Optimal-Transport transported-mass parameter m of scConfluence over a grid (e.g. 0.3–0.75), refitting the model at each value and scoring the shared latent space to verify the default setting sits in a robust performance plateau, as in the paper's Supplementary Fig. 13 analysis on the scCATseq cell-line benchmark.

## When to use

Use when a model's default hyperparameter (e.g. the unbalanced OT transported mass m=0.5 in scConfluence) lacks justification for your dataset and you need to demonstrate that embeddings are insensitive to its choice across progressively unbalanced modality scenarios.

## When NOT to use

- Prior information about the level of unbalanceness is available and a specific m is justified — a sweep adds little beyond confirming the known value
- The method being tuned does not expose a transported-mass parameter or relies on cross-modality feature connections that are unavailable (e.g. electrophysiology vs gene expression)
- The goal is benchmarking against other integration methods rather than characterizing robustness to one model's hyperparameter

## Inputs

- Preprocessed unpaired scCATseq (or similar) MuData object with QC-filtered, normalized modalities
- Cross-modality cost matrix from connected features in mdata.uns['cross_modality1+modality2'] with mdata.uns['cross_keys']
- Cell-type labels for kNN purity scoring
- Grid of transported-mass values m (e.g. 0.3, 0.4, 0.5, 0.6, 0.75)

## Outputs

- Latent embeddings per m value in mdata.obsm['latent']
- Metric-vs-m curves (cell-type purity, FOSCTTM, graph connectivity) written to sensitivity_curves.csv
- Sensitivity figure (e.g. sensitivity_fig13.png) comparable to Supplementary Fig. 13

## How to apply

Train the scConfluence model (AutoEncoder per modality assembled with scconfluence.model.ScConfluence) at its default unbalanced OT setting m=0.5, computed via ot.partial.partial_wasserstein on the cross-modality cost matrix stored in mdata.uns['cross_modality1+modality2']. Then sweep m over a grid such as 0.3, 0.4, 0.5, 0.6, 0.75, refitting the model at each value and extracting embeddings with model.get_latent() into mdata.obsm['latent']. At each m value, score the shared latent space with cell-type purity (kNN label agreement), FOSCTTM (cross-modality nearest-neighbor consistency), and graph connectivity computed with Scanpy (sc.pp.neighbors, sc.tl.umap on the 'latent' representation). The rationale is that the paper sets m=0.5 by default because it 'produces robust performances when prior information about the level of unbalanceness of the data is not available'; the sweep tests whether that holds for your data. Plot each metric against m, compare the curve shapes against Supplementary Fig. 13, and record results in a table (e.g. sensitivity_curves.csv) and figure (e.g. sensitivity_fig13.png). Acceptance: curves show a near-optimal flat region around the default, and robustness extends to increasingly unbalanced scenarios (e.g. half K562 removed, all K562 removed, K562 and HCT removed in the scCATseq benchmark).

## Related tools

- **scConfluence** (Multimodal model whose unbalanced OT transported-mass parameter m is swept and refit at each grid value; latent embeddings extracted with model.get_latent()) — https://github.com/cantinilab/scconfluence
- **POT** (Provides ot.partial.partial_wasserstein used to compute the unbalanced OT plan at each m value)
- **Scanpy** (Computes sc.pp.neighbors and sc.tl.umap on the 'latent' representation to derive graph connectivity and support metric scoring)
- **Muon** (Holds the preprocessed MuData object with cross-modality cost matrices in .uns)

## Examples

```
model = scconfluence.model.ScConfluence(mdata, unimodal_aes=autoencoders); model.fit(save_path='results'); mdata.obsm['latent'] = model.get_latent()
```

## Evaluation signals

- Metric-vs-m curves (purity, FOSCTTM, connectivity) reproduce the qualitative pattern of Supplementary Fig. 13
- The default m=0.5 configuration falls in a near-optimal flat region of all three metrics
- Performance remains robust across progressively unbalanced scCATseq scenarios (half K562 removed, all K562 removed, K562 and HCT removed)
- Latent embeddings are correctly written to mdata.obsm['latent'] via model.get_latent() at every grid value
- sensitivity_curves.csv contains one scored row per m value with all three metrics

## Limitations

- The paper notes a challenging aspect for scConfluence and state-of-the-art methods is properly dealing with rare cell populations, which are hard to detect and under-represented in mini-batch gradient descent — sensitivity conclusions may not extend to very rare populations
- scConfluence relies on cross-modality feature connections, so the sweep is inapplicable for modality pairs lacking such connections (e.g. electrophysiology vs gene expression)
- Supplementary Figs 13–18 (hyperparameter sensitivity) are referenced but not included in the article text, so exact curve values must be regenerated rather than compared numerically

## Evidence

- [discussion] Default m and robustness rationale: [evidence span withheld — anchor status: not_found]
- [discussion] Unbalanced OT plan computation: [evidence span withheld — anchor status: not_found]
- [results] Robustness finding on unbalanced scenarios: [evidence span withheld — anchor status: not_found]
- [discussion] Sensitivity figures referenced but missing: [evidence span withheld — anchor status: not_found]
