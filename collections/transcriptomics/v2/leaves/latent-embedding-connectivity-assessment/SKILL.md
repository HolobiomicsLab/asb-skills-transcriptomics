---
name: latent-embedding-connectivity-assessment
description: Use when you have a shared low-dimensional embedding from unpaired multiomic
  integration (e.g. scConfluence latents in mdata.obsm['latent']) and must quantify
  alignment quality or hyperparameter sensitivity (e.g. unbalanced-OT m) using purity,
  FOSCTTM, and graph connectivity.
when_to_use_negative:
- The embedding was produced by paired (jointly profiled) multimodal integration rather
  than diagonal integration of unpaired data
- No cross-modality feature connections exist for the modality pair (e.g. electrophysiology
  vs gene expression), so scConfluence latents cannot be produced and this scoring
  scheme does not apply
- Cell-type labels are unavailable, making purity/kNN label agreement and label-dependent
  FOSCTTM evaluation infeasible
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scConfluence
  role: Produces the shared latent embeddings being assessed; hyperparameter m of
    the unbalanced OT (default 0.5 via ot.partial.partial_wasserstein) is swept during
    the sensitivity analysis
  repo: https://github.com/cantinilab/scconfluence
- name: Scanpy
  role: Computes the neighbor graph and UMAP on the latent representation (sc.pp.neighbors,
    sc.tl.umap) used for connectivity and purity scoring
- name: Muon
  role: Supplies the preprocessed MuData object and per-modality normalizations feeding
    the embedding
- name: POT
  role: Computes the unbalanced OT plan via ot.partial.partial_wasserstein at each
    swept m value
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-024-51382-x
    title: 'scConfluence: single-cell diagonal integration with regularized Inverse
      Optimal Transport on weakly connected features'
  source_content_hash: sha256:36fc43f0e31cc813592e108997628220ad4a3c011825e2981a7de13c57aee0ea
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# latent-embedding-connectivity-assessment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Scores a shared single-cell latent embedding (e.g. scConfluence's model.get_latent() output in mdata.obsm['latent']) using cell-type purity (kNN label agreement), FOSCTTM cross-modality consistency, and graph connectivity computed with Scanpy, and optionally sweeps sensitivity to hyperparameters such as the unbalanced-OT transported mass m.

## When to use

Use when you have a shared low-dimensional embedding from unpaired multiomic integration (e.g. scConfluence latents in mdata.obsm['latent']) and must quantify alignment quality or hyperparameter sensitivity (e.g. unbalanced-OT m) using purity, FOSCTTM, and graph connectivity.

## When NOT to use

- The embedding was produced by paired (jointly profiled) multimodal integration rather than diagonal integration of unpaired data
- No cross-modality feature connections exist for the modality pair (e.g. electrophysiology vs gene expression), so scConfluence latents cannot be produced and this scoring scheme does not apply
- Cell-type labels are unavailable, making purity/kNN label agreement and label-dependent FOSCTTM evaluation infeasible

## Inputs

- MuData object with cell-type labels and preprocessed modalities (scRNA log-normalized with Scanpy normalize_total/log1p, scATAC TF-IDF via Muon, protein CLR via Muon)
- Cross-modality cost matrix in mdata.uns['cross_modality1+modality2'] with mdata.uns['cross_keys']
- Fitted scConfluence model (scconfluence.model.ScConfluence with unimodal AutoEncoder instances)
- Latent embeddings in mdata.obsm['latent'] obtained via model.get_latent()

## Outputs

- Per-m (or per-configuration) table of cell-type purity, FOSCTTM, and graph connectivity values (e.g. sensitivity_curves.csv)
- Metric-vs-m sensitivity curve figure (e.g. sensitivity_fig13.png)
- Neighbor graph and UMAP computed on the 'latent' representation

## How to apply

After fitting the multimodal model (scConfluence with uncoupled AutoEncoders trained on preprocessed MuData with a cross-modality cost matrix in mdata.uns['cross_modality1+modality2']), extract embeddings with model.get_latent() into mdata.obsm['latent']. Build a neighbor graph with Scanpy (sc.pp.neighbors on the 'latent' representation), compute a UMAP (sc.tl.umap), then score: cell-type purity as kNN label agreement, FOSCTTM as cross-modality nearest-neighbor consistency, and graph connectivity. To assess sensitivity to the unbalanced-OT transported mass, sweep m over a grid (e.g. 0.3, 0.4, 0.5, 0.6, 0.75), refitting at each value — the default m=0.5, computed via ot.partial.partial_wasserstein, is expected to sit in a robust flat region of the metric curves. Write the metric-vs-m table to sensitivity_curves.csv and plot the curves (e.g. sensitivity_fig13.png) for comparison against the paper's Supplementary Fig. 13.

## Related tools

- **scConfluence** (Produces the shared latent embeddings being assessed; hyperparameter m of the unbalanced OT (default 0.5 via ot.partial.partial_wasserstein) is swept during the sensitivity analysis) — https://github.com/cantinilab/scconfluence
- **Scanpy** (Computes the neighbor graph and UMAP on the latent representation (sc.pp.neighbors, sc.tl.umap) used for connectivity and purity scoring)
- **Muon** (Supplies the preprocessed MuData object and per-modality normalizations feeding the embedding)
- **POT** (Computes the unbalanced OT plan via ot.partial.partial_wasserstein at each swept m value)

## Examples

```
import scconfluence, scanpy as sc; model = scconfluence.model.ScConfluence(mdata, unimodal_aes=aes); model.fit(save_path='results'); mdata.obsm['latent'] = model.get_latent(); sc.pp.neighbors(mdata, use_rep='latent'); sc.tl.umap(mdata)
```

## Evaluation signals

- The metric-vs-m curves reproduce the qualitative pattern of the paper's Supplementary Fig. 13
- The default m=0.5 configuration yields robust (near-optimal, flat-region) purity, FOSCTTM, and connectivity values
- Latent embeddings are present in mdata.obsm['latent'] for every m value in the swept grid
- Graph connectivity is computed on a Scanpy neighbor graph built from the 'latent' representation, not the raw counts
- In progressively unbalanced scCATseq scenarios (half K562 removed, all K562 removed, K562 and HCT removed), scores remain consistent with scConfluence's reported robustness advantage over state-of-the-art methods

## Limitations

- The sensitivity analysis compares against Supplementary Fig. 13, which is referenced but not included in the provided document, so exact target values are unavailable
- Rare cell populations are hard to handle: they are under-represented in mini-batch gradient descent, which can distort embedding quality metrics
- No statistical method was used to predetermine sample size, so metric differences should be interpreted cautiously

## Evidence

- [discussion] default m=0.5: [evidence span withheld — anchor status: not_found]
- [full_text] workflow m-sweep: [evidence span withheld — anchor status: not_found]
- [full_text] workflow scoring metrics: [evidence span withheld — anchor status: not_found]
